# Overlay System
In EoS, the game manages overlays via a hierarchy system, where some overlays need another one to be loaded to be loaded themselves. For each hierarchy tier, there can only be one overlay loaded, and there are 3 tiers. In the explanations, we'll refer to tier 0 as the core tier, tier 1 as the main tier, and tier 2 as the sub tier.
## Sub Overlays
Sub-overlays are used to store the code and data of game mechanics that are temporary, mutually exclusive, and bound to a main overlay. The overlays that store the mechanics of the Treasure Town shops are a great example of what this means: you use them for a short time before being being unloaded (temporary), you can only ever be in one shop at a time (mutually exclusive), and you aren't able to access them at all outside of ground mode (bound to a main overlay). Sub-overlays are the lowest on the hierarchy scale, and always depend on a main overlay being loaded.

(WIP)
Due to them being used frequently 

In addition to the overlay loading mechanism, the game also features a system that allows easy managing of overlays of tier 3 (as these are the ones that change most during gameplay and require little specialized setup, unlike overlays of tier 2.) This system takes of care of not only initializing and freeing data when the overlays are loaded, but it also takes care of calling the update routines.

To load an overlay, the main overlay passes a struct 


The overlay hierarchy is structured as follows: 
- Overlay 0
    - Overlay 1
        - Overlay 9 (sky jukebox) 
        - Overlays 3-9
    - Overlay 2
- Overlay 10 (game data/mechanics)
    - Overlay 11 (ground engine)
        - Overlay 15 (duskull bank) 
        - Overlays 12-28
    - Overlay 29 (dungeon mode)
        - Overlay 31 (dungeon menu) 
        - Overlays 30-33
    - Overlay 34 
- Overlay 35 (unused)

