# Battle Sister’s BLAST \- Creating a highly efficient render engine for Meta Quest

<img src="/Images/BLAST/Image0-BattleSister.webp" />

[Warhammer 40,000: Battle Sister](https://www.meta.com/en-gb/experiences/warhammer-40000-battle-sister/3231001163584154/) was a launch title for Quest 2 (as well as releasing on Quest 1 and later [Steam](https://store.steampowered.com/app/1733890/Warhammer_40000_Battle_Sister/)). Battle Sister used a novel render engine, combining an edit time baking solution and runtime batcher that achieved extremely low drawcall counts for environments as well as optimised fillrate and minimal overdraw.

This blog post describes the background for BLAST’s development as well as a high level technical overview of how it worked. Please note this is written from memory and there may be some minor inaccuracies.

Posted with kind permission of [Soul Assembly](https://soulassembly.com/).

# The Problem Space

Following the release of Drop Dead: Dual Strike as a launch title for Oculus Quest in 2019, we started pre-production on the next game. At that time, we weren’t sure what the next title would be but we understood certain challenges on standalone VR that we wanted to research systemic solutions to.

We knew that particular constraints were:

* Draw Calls  
* Overdraw and Fill Rate Limits

Common solutions to these constraints include static batching, atlases and occlusion culling.

Atlases are a good solution but they make it more complicated to handle tiling and they can’t be that large before you are forced to create a new atlas. If hand authored, they are also inflexible when you decide to add or remove assets to levels.

We had evaluated Unity’s occlusion culling several times on mobile and not found a consistent performance benefit, often it resulted in poorer performance.

Static batching is well implemented in Unity, but did not give us the level of control to achieve our overall vision for performance.

In addition we were also interested in seamless level streaming for considerably bigger maps and longer levels, wanting to avoid being constrained to short levels like Drop Dead or having repeated hard load screens.

Around this time Unity announced Native Mesh Access (Unity 2019.3), an efficient way to manipulate mesh data via CPU jobs that works well on mobile devices where CPU and GPU RAM is shared.

We were also evaluating the use of texture arrays as a solution for collapsing draw calls with similar material/shader setup that differed principally only in textures.

Other considerations included optimising vertex attribute size, order and splitting streams for better binning shader performance.

# Introducing BLAST

Around the time that this initial research concluded, we had a proof of concept working as well as a green lit game (Warhammer 40,000: Battle Sister) to use it on\!

During Battle Sister’s development we evolved this research into a technology called BLAST \- a loose backronym for Batching, Lodding and Streaming Technologies.

BLAST is a rendering engine designed for Quest supporting extreme levels of batching, fine levels of occlusion and sorting, streaming assets in and out. Consisting of an editor baking tooling and a runtime engine using Jobs with Native Mesh Access and texture arrays. The majority of static environments in Battle Sister render in just 4 drawcalls and are aggressively optimised for fillrate compared to other typical approaches. BLAST essentially sidesteps most of Unity’s rendering stack and provides an alternate paradigm for rendering environments.

Levels were created with modular pieces, generally using a 3x3 meter footprint (both for floor pieces and walls). 3x3 was chosen as it accommodates typical door sizes and seems a good trade off between being too granular (resulting in tedious placement) and too large (resulting in poor culling or sorting opportunities). Snap points were also placed on many kit pieces allowing them to be snapped together in a footprint friendly way. Meshes were authored so that the wall thickness was baked in relative to the pivot/snap points, so they could be placed easily back to back and have the correct wall thickness. 

[Fallout 4’s Modular Level Design](https://www.youtube.com/watch?v=QBAM27YbKZg) talk (as well as it’s [Skyrim predecessor](http://blog.joelburgess.com/2013/04/skyrims-modular-level-design-gdc-2013.html)) was highly inspirational and is required watching for anyone making level design tools.

## A Note on Level Streaming

Side by side with BLAST, a level streaming system was also implemented, inspired by Unreal’s model. Each mission was broken into multiple chunks, each with Art and Gameplay versions to allow for better source control concurrency. Cuboid level streaming volumes are hand placed. Multiple level streaming volumes can reference the same level chunk and reference counting is used, this means that concave play spaces can be made out of multiple volumes.

BLAST consisted of two elements:

1. Editor  
2. Runtime

# Editor

The editor consisted of a tooling window that allowed you to kick off BLAST bakes as well as inspect the output of the current bake (mainly how many merge groups it produced and what was packed in them).
<figure>
<img src="/Images/BLAST/Image1-Chapter6NavMesh.jpg" />
<figcaption>Chapter 6: Nav Mesh Input</figcaption>
</figure>

<figure>
<img src="/Images/BLAST/Image2-Chapter6Voxels.jpg" />
<figcaption>Chapter 6: Voxelised</figcaption>
</figure>  

<figure>
<img src="/Images/BLAST/Image3-UniqueColors.jpg" />
<figcaption>Unique Color Preview Of A Test Level (3D Game Kit)</figcaption>
</figure>  

The bake process involved:

1. A validation pre-step runs which checks that eligible assets (those not marked for exclusion, either explicitly or through a rule) are correctly configured. This prevents a lengthy wait to a bake failure due to a missing checkbox on an asset.  
2. Voxelise the nav mesh \- Battle Sister used the nav mesh, not just for NPC movement but also for player teleportation. The nav mesh was considered a good analogue for where the player could go. BLAST’s first step is to voxelise the Nav Mesh into 2 meter cubes, with a limited stacking capability too in case the player is particularly tall or physically jumps. For lift shafts or areas the player can climb, voxelised volumes could also be placed, providing additional input.  
3. BLAST then walks through each voxel:  
   1. For each voxel, the world that should be loaded corresponding to the level streaming volumes including the voxel is rendered using special replacement materials as a cubemap. As the cubemap camera reacts to mesh renderer LOD levels differently to a VR camera, we also calculate the correct LOD levels for VR and force the LOD Groups to use that.  
      1. For opaque objects each object is given a unique colour in its replacement material, then a compute shader counts how many pixels of each object is visible. An object is considered visible if it exceeds a certain (fairly low) count. Multiple samples are taken within each voxel to allow for player/head movement (since the voxels are fairly large).  
      2. Transparent objects (such as billboard quads used for lights) are then drawn, with each object representing a bit in a bit field. By using a bit field we can have multiple transparent objects layered together and still calculate their visibility. Multiple passes are applied as the number of transparent objects may exceed the size of the bit field. Opaque objects are still drawn, but in black.  
      3. Opaque objects are then sorted front to back (as it’s more efficient when depth buffer rejection is used. Transparent objects are sorted back to front as this is required for correct layering.  
4. Voxel visibility is then locally merged by having each voxel inherit its neighbours visibility too, this avoids an issue where putting your head round a corner quickly could allow you to see the world pop in as it takes approximately 4 frames to update what is visible. While this increases overdraw in some cases, the game’s nature of push forward combat and the general performance benefits of BLAST meant this was an acceptable compromise.  
5. In addition, a very simple cell to cell visibility mapping was created. This had a limited view distance, but was used as a form of occlusion culling on dynamic objects. (If the object is inside a voxel that can be seen from your voxel then draw it). This was added very late in development and was a fairly simple/crude implementation, but had a tangible benefit.  
6. Texture array sizes are pre-calculated for each level so that they can be allocated when the level is loaded.  
7. The visibility data is written to disk as a proprietary binary format with the .bpvs extension (Binary Potentially Visible Set).  
8. Mesh data is written to disk as a proprietary binary format. In addition to optimising vertex attribute layout (and size), certain material data is also packed in as vertex attributes to allow more aggressive draw call merging. For example if two materials only differ in textures and a tint colour, then the textures are packed into the texture array and the tint colour is packed into a vertex attribute. For billboard quads, the world position of the pivot point is also baked, allowing the quads to be rotated independently even when merged.  
9. Asset bundles for the textures are generated or updated containing the assets required, to make it easy to load the textures at runtime

The editor tooling also included ways of previewing visibility at edit time by dropping in a gizmo “camera” that could be moved or rotated.

## Build Time Stripping

When the level is built (including play in editor) it is automatically stripped of static mesh renderers that are rendered by BLAST. This process was fairly aggressive, stripping empty game objects and containers, leaving typically static colliders behind.

Collision is also simplified where possible. Coplanar colliders such as box colliders on walls were also automatically merged up to a certain distance limit.

This automatic stripping was one of the key requirements to ensure main thread integration time remained low when loading chunks in. (Note that scene load was entirely separate to BLAST loading in its own dependencies as you walk around cells and was mainly for the collision as well as dynamic objects not covered by BLAST).

# Runtime

<figure>
<img src="/Images/BLAST/Image4-CullingView.jpg" />
<figcaption>Player Perspective</figcaption>
</figure>

<figure>
<img src="/Images/BLAST/Image5-NoCulling.jpg" />
<figcaption>No Culling</figcaption>
</figure>

<figure>
<img src="/Images/BLAST/Image6-Culling.jpg" />
<figcaption>BLAST's Culling</figcaption>
</figure>

<figure>
<img src="/Images/BLAST/Image7-OverdrawDefault.jpg" />
<figcaption>Unity's Default Renderer</figcaption>
</figure>

<figure>
<img src="/Images/BLAST/Image8-OverdrawBlast.jpg" />
<figcaption>BLAST's Sorted</figcaption>
</figure>

On mission load, BLAST initialises by loading the binary potentially visible set, which is then parsed into an efficient runtime data structure. I believe binary meshes and possibly some other resources (like reflection probes) also get loaded on mission load due to how fast it was to load them and the relatively high available memory on the Quest headset. Binary meshes are often repeated kit pieces, so many of them would be needed within the first few voxels anyway. Texture arrays are also allocated to the size of the maximum number of textures that are visible from a voxel, plus a small buffer so that we aren’t forced to immediately unload textures at high water moment areas and potentially end up in a streaming ping pong situation if a player is ducking in and out of cover.

As the player progresses through cells a number of operations occur. Firstly, we load any missing resources. Textures are streamed in from asset bundles, blitted into the correct texture array cell then the source texture unloaded. If textures have been previously loaded into texture array cells then the cell that hasn’t been used in the longest time is allocated to the new texture. BLAST tracks what textures are loaded in each texture array cell, marking cells as available if the player moves to a cell which doesn’t require that texture. The texture is still retained in memory however, in case the player then moves to a cell which needs it again. Note that BLAST intentionally doesn’t load all textures from a level into memory in order to keep memory usage more balanced.

A number of Burst compiled jobs are then initialised, with some dependencies between them. These jobs update the native mesh buffers, copying in the correct mesh data and setting the texture array index as a vertex attribute for all the vertices that match each material. Once the meshes are ready they are applied and begin rendering instead of the previous merged mesh. This process takes approximately 4 frames inclusive of the frame starting the jobs and the frame applying the final mesh.

Like the addition of culling dynamic objects based on containing cell visibility, another late addition was a form of coarse frustum culling. Meshes are classified in relation to the cell centre into 8 horizontal directions (cardinals \+ diagonals). Meshes that are very close or above/below are classified as visible in all directions. As the player turns their head we turn on/off the directions and update the mesh. This is fairly conservative in its tolerances due to the 4 frame delay, but again provided a small but measurable benefit.

# Limitations

Certain assumptions in BLAST’s approach were made in 2019, assuming that the team would be working in an office full-time. Of course in early 2020, with Covid-19 these assumptions were instantly invalidated. In hindsight, I would have taken a different approach with some of BLAST’s architecture knowing that we were going to need to work remote first with people’s variable internet connections between their development machines and the office.

BLAST would also have benefited from a longer lead time before being used in production. A lot of the workflow challenges we faced were symptomatic of “flying the plane while building it”. Though this is a common problem in game development and certainly not unique to Battle Sister’s development. As a counter argument, BLAST’s development benefited from feedback from developers using it actively and with real world levels.

While the bucketing approach worked relatively well, not allowing looser texture sizes was somewhat constraining. In certain cases smaller textures were upscaled to prevent a bucket split. Looking forward, bindless rendering may be a better solution in the future to these kinds of problems.

While BLAST made rendering vastly more efficient it didn’t have any handling of vistas or playable level areas in the distance. Some form of HLOD integration (with proxy mesh generation via Simplygon) would have greatly expanded the quality and potential of large environments.

A considerable pain point for level design and environment art were that BLAST bakes were slow. Taking multiple samples per voxel across the size of Battle Sister’s levels, then classifying all the data could take a couple of hours for a large level. Considerable effort was put into optimising BLAST bakes but as levels grew in complexity and BLAST’s new capabilities also extended baking requirements it became a one step forward \- one step backward situation.

Further to the bake time problems, BLAST did not provide an effective way to play these levels natively on Quest without a BLAST bake, resulting in a large iteration time loop. This was compounded with a similar problem with lightbakes, so wasn’t isolated to BLAST, but was far from an ideal workflow.

BLAST was also very picky about the format of meshes and textures it would ingest, requiring them to match in certain rules. For example BLAST could only merge textures into texture arrays at runtime if the ASTC compression level matched. This resulted in a very complicated validator to pick up issues that would prevent baking or cause runtime mismatches, given more time BLAST could have been developed to be more lenient and flexible with the data it ingested as well as creating better import pipelines.

# Conclusion

Looking at RenderDoc captures from Battle Sister on Oculus Quest showed just how optimal environments were, not just in draw call count but in how tightly culled and well sorted the geometry being rendered was. This efficiency gave us huge amounts of budget to spend elsewhere on gameplay and VFX.

On reflection, I feel that BLAST over-optimised for the problem space and that we could have accepted a simpler workflow with coarser merging and less optimal fillrate. While this would have meant less budget for the rest of the game, the iteration speed boosts would have been worth it.

Finally, creating BLAST is one of the highlights of my career. I am hugely proud of what we accomplished with Battle Sister, as well as BLAST’s role in getting the game to performance.
