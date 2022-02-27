#Overlay System
In EoS, the game manages overlays via a hierarchy system, where some overlays need another one to be loaded to be loaded themselves. For each hierarchy tier, there can only be one overlay loaded, and there are 3 tiers. The overlay hierarchy is structured as follows: (WIP)
- Overlay 0
    - Overlay 1
        - Overlay 9 (sky jukebox) 
        - Overlays 3-9
    - Overlay 2
- Overlay 10 (game data/mechanics)
    - Overlay 11 (script engine)
        - Overlay 15 (duskull bank) 
        - Overlays 12-28
    - Overlay 29 (dungeon mode)
        - Overlay 31 (dungeon menu) 
        - Overlays 30-33
    - Overlay 34 
- Overlay 35

In addition to the overlay loading mechanism, the game also features a system that allows easy managing of overlays of tier 3 (as these are the ones that change most during gameplay and require little specialized setup, unlike overlays of tier 2.) This system takes of care of not only initializing and freeing data when the overlays are loaded, but it also takes care of calling the update routines.
