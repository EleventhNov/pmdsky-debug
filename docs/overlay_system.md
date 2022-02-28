# Overlay System
In EoS, the game manages overlays via a hierarchy system, where some overlays need another one to be loaded to be loaded themselves. For each hierarchy tier, there can only be one overlay loaded, and there are 3 tiers. In the explanations, we'll refer to tier 0 as the core tier, tier 1 as the main tier, and tier 2 as the sub tier.
## Sub-Overlays
Sub-overlays are used to store the code and data of game mechanics that are temporary, mutually exclusive, and bound to a main overlay. The overlays that store the mechanics of the Treasure Town shops are a great example of what this means: you use them for a short time before being being unloaded (temporary), you can only ever be in one shop at a time (mutually exclusive), and you aren't able to access them at all outside of ground mode (bound to a main overlay). Sub-overlays are the lowest on the hierarchy scale, and always depend on a main overlay being loaded.
## The Sub-Mechanic System
As mentioned before, overlays are useful to store temporary mechanics. The game features a small system to load a single mechanic and ensuring the overlay required to use it is loaded. This system can also take care of initializing and destructing required globals, as well as calling the update function. For now, this will be called **the sub-mechanic system**.

In this system, a sub-mechanic is defined of four things: an overlay group ID, and three function pointers (entry, free and update). Even if the overlay group IDs are the same, a sub-mechanic won't be considered equal to another unless the functions are also the same, and this is with good reason. A great example can be found in Spinda Cafe, for which ground mode stores two identical mechanics, save for the entry function. {But what is the example? TODO here XD, I need to do research first}

This system is not only restricted to _loading_ overlays: a main overlay can request itself to be loaded, and while this does nothing on its own, it can also let the sub-mechanic system take care of calling the sub-mechanic functions instead of doing it manually. The ARM9 binary also makes use of this system, albeit instead of loading itself, it simply doesn't load anything (using an overlay group of zero).

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

