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
````
