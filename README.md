<p align="center">
  <img src="https://i.ibb.co/SRpRBmD/cracks-Icon.png" alt="cracks-Icon" width="140px" height="150px"/>
  <h1 align="center">Procedural Terrain Generator</h1>
  <h3 align="center">for <b><i>Minecraft: Java Edition v1.20.1</i></b></h3>
</p>




This Python program procedurally generates a group of floating islands fitted with realistic height mapping, tree and shrub generation, ore generation, and much more. While originally intended for the purpose of on-demand random generation for a minigame, this code is incredibly versatile and can be used in plenty of applications.

## Generation Logic
The generation is divided into several phases: **Map Generation**, **Agent Generation**, **Probabilistic Generation**, and **Voxel Shading**. We will go over each in detail.

### Map Generation
This code generates 4 maps: A **heightmap** generated from fractal noise, a **biome map** generated from larger-scale fractal noise, a **pool map** that protects the generation of lava and water pools, and most importantly, a **crack map** to divide up the circular island into randomly sized and shaped pieces.

- Heightmap
  - For the fractal noise of the heightmap I actually chose the route of writing the noise generation code myself, just so that I would have ultra-fine control over all the aspects of the process. It begins by creating an image of purely random noise, then blurring it a series of times, superimposing a faded version of each step onto the next using the `openCV` library in Python (some level of value scaling is also applied to prevent everything becoming 50% gray). By carefully controlling the relative opacities of each layer, we can fine-tune the scale of the variation in the final heightmap (visualized in `images/mountains5.png`) The scale of variation I selected here works well for this size of island, but feel free to play around with it if you're curious.

- Biome Map
  - The biome map is actually just the heightmap, with 5 more levels of "upscaling" applied so that the features are extremely large -- then we select various ranges of value to assign to different biomes, the result of which can be seen in `images/biomes.png`. Of course, this method of biome mapping means that biomes cannot border other biomes whose corresponding value range does not touch their own. This worked fine for me as I didn't really want snow and desert to be touching anyway, but I might explore a more sophisticated biome mapping method in the future by layering different noise maps together.

- Pool Map
  - There's not really much to say here, the pool map (`images/poolmap.png`) is just a visual representation of the X and Z coordinates of all the water and lava pools on the map. It's not actually used to generate the pools themselves, but rather to delete any surface shrubbery that wants to generate in those blocks.

- Crack Map
  - This is perhaps the most complex of all the maps. The crack map generation uses a modified Voronoi pattern to generate the islands, with some edge-detection code to correctly mark all the boundaries between Voronoi cells. A small bug in this edge-detector led to some Voronoi cells being halfway or even fully cut off, leading to interesting "gaps" or "holes" in the terrain apart from the cracks. I thought this looked really interesting so I decided to keep it.

*Fun fact: the icon for this repository is actually a real island map generated by the algorithm!*

### Agent Generation
In this program, I refer to an **agent** as any procedurally generated object that has an awareness of its location, and the locations of the other agents of its kind. We use agents to generate:
- Patches of different rock types (andesite, diorite, granite)
- Ore patches (coal, copper, iron, lapis lazuli, diamond)
- Lava and water pools
- Trees (oak, spruce)
- Cacti
- Large mushrooms (red, brown)

The method behind agent generation varies based on the type of agent, but all share a basic approach: generate a set amount of agents, use the relative locations of agents to ensure that they are not too close to one another, then modify the blocks around the agent in some way. For rock patches, this could be replacing all the stone in a sphere around the agent, but for ores maybe use a probability function that decays as the blocks get further away from the agent. For trees, cacti and shrooms we use a growth ruleset to ensure they look like Minecraft, and for lava and water pools we use a method of descending discs to ensure that they are "nestled" in the ground and cannot spill into the void.

### Probabilistic Generation
The opposite of an **agent** would be a stateless object that does not have any awareness of itself or neighbors. This is far cheaper to generate, but at the limitation of what we can actually do with it. We use probabilistic generation to spawn:
- Grass (and tall grass)
- Flora (poppies, dandelions)
- Dead bushes
- Small mushrooms (red, brown)
- Sugar cane (only around water pools)

The method behind probabilistic generation is far simpler than that of agents. For each block on the surface of the ground, depending on the biome we have some predetermined probability tables that tell us the % chance that there is a certain object above it. For the specific case of sugar cane, we don't loop over every surface block (because we don't need to), instead when generating the pools themselves there is a section of code that randomly spawns sugar cane on the bordering blocks.

### Voxel Shading
This is the most important aspect of generation. Voxel shading is the idea of "shading" -- a term I use for setting the ground blocks to various block types -- what is originally a terrain made entirely of blue wool (placeholder). The voxel shader runs through the entire 3-dimensional terrain, assigning surface blocks based on the biome, subsurface blocks, and the layers of stone underneath.

Voxel shading also does another important step -- giving the islands their tapered shape. The crack map only generates a 2D top-down projection of the land, but extruding this downward without additional logic would create odd, cylinder-like shapes that abruptly end. To avoid this, we use an erosion algorithm to chip away at the edge blocks as we extrude downward (with some random variation to keep it looking natural). The result is nicely tapering, organic-looking floating islands with exposed ores and rock patches.

## Output
If you want to use what this program generates, I have provided two options: If your goal is to play with the result on your own world and not much more, or if you have a way of reading from the WorldEdit `.schem` schema, then you're all set to use `pvpisland.schem`.

However, if you're looking to automate the generation and you don't have a way of reading the `.schem` file, head to the bottom of `main.py` and modify the code as shown below:

```python
###### MAIN ######
generateIslandShape(180, 180)
#convertToJson() <--- UNCOMMENT THIS LINE!
```
Doing so will add a step to the generation process where all the data of the schematic is converted to a `.json` file that explicitly states the blockData of every block in the island. The structure of this file is quite easy to understand and read from, but due to its uncompressed nature it can get to be very large -- several MB in most cases.

In the context of generation time, this conversion step adds negligible time to the generation, a few hundred milliseconds at most.

##### *Last Modified: 12/26/24 by Ajaya Ramachandran*