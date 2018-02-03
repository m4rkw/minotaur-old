````
30/01/2018 0.2 - initial release
30/01/2018 0.3 - bugfix
30/01/2018 0.4 - changes:
 - added more algorithms to the ccminer2 shim (alexis78)
 - removed non-working algorithms
 - bugfix: shim algorithms were not stopped if the hashrate was not detected
 - added P0/P1/P2 state to the gs display
 - fixed profile selection based on device/algorithm
 - ensure default profile is always loaded before stopping workers
 - don't crash if the miner backends all go away
 - apply reloaded profile settings for active devices on HUP
30/01/2018 0.5 - minor fixes
30/01/2018 0.6 - changes:
 - support for custom device classes
 - support for overclocking in calibration runs
 - ignore GPU/memory clock offsets in the default profile
 - bugfix: SIGHUP crashes gs
 - bugfix: fixed some gs display artefacts
 - added option to clean up workers for a specific device
 - bugfix: fixed setting of power limit during calibration
 - if calibrating without power tuning enabled, store the default power limit
   rather than the maximum observed power draw
 - fixed a crash when HUPing after removing an active calibration profile
 - tidied up gs display columns
 - fixed ccminer lyra2rev2 compatibility
30/01/2018 0.7 - changes:
- minor bugfix in nicehash library
- fix for ccminer jobs not being cleaned up on SIGINT when calibrating
31/01/2018 0.7.1 - changes:
- fixed minor gs rendering bug
- fixed crash when running blended algorithms
- enabled scrypt, sia, x11 and x13 in ccminer
31/01/2018 0.7.2 - changes:
- fixed an issue that can cause a crash under certain conditions
31/01/2018 0.7.3 - changes:
- bugfix: --cleanup not stopping workers
31/01/2018 0.7.4 - changes:
- show separate mBTC rates when running dual algos
- minor fixes and improvements
- fixed power throttling
- fixed margin display under low hashrate conditions
31/01/2018 0.7.5 - changes:
- added support for device pinning
- fixed minotaur not stopping workers on SIGINT
01/02/2018 0.8.4 - changes:
- rewrote a lot of the core code to make it cleaner, faster and more stable
- resolved some edge-cases
- fixed crash when theres no calibration data
- dont touch the power limit when calibrating in quick mode
02/02/2018 0.8.5 - changes:
- various stability fixes
- sensible config defaults
- added more flexible calibration parameters
02/02/2018 0.8.6 - changes:
- fixed an edge-case crash
- added quickstart mode
- added turbo calibration
02/02/2018 0.8.7 - fixed setting of power limits
02/02/2018 0.8.8 - changes:
- indicate when nicehash is down
- default to equihash in eu when nicehash is down
02/02/2018 0.8.9 - changes:
- use threading and queues to execute algo switches much faster
- added default algorthm to nicehash config for when the API is unavailable
- added workarounds for older cards that don't support setting a power limit
02/02/2018 0.8.9.1 - changes:
- misc bugfixes
- fixed calibration algorithm
````
