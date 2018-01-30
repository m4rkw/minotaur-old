# Minotaur, Fanotaur and Excavataur by m4rkw

## Warning

MINOTAUR SUPPORTS OVERCLOCKING. OVERCLOCKING CAN DAMAGE YOUR HARDWARE IF
ENABLED. YOU ASSUME ANY AND ALL RISK ASSOCIATED WITH OVERCLOCKING BY THE USE OF
THIS SOFTWARE. WE ARE NOT RESPONSIBLE FOR ANY DAMAGE THAT MAY ARISE FROM THE USE
OF THIS SOFTWARE. SEE THE LICENSE TERMS FOR MORE INFORMATION.

## Projects

These three projects are designed to work together:

- Minotaur - this one you're looking at :)
- Fanotaur - https://github.com/m4rkw/fanotaur
- Excavataur - https://github.com/m4rkw/excavataur

## Overview

Minotaur is a miner management system designed to maximise profit. It has been
primarily designed to work with Nicehash and the Excavator miner but has support
for ccminer and support is planned for as many miners as possible.

For now Minotaur is closed-source and donation-supported, as I'd quite like to
have time to really focus on making it the best miner-management system there
is. The default donation is 5% (5 minutes in 100 minutes) and can be changed in
the config file using the "donation_level" setting with a minimum of 1%.
No donation occurs during calibration runs.

Depending on how the project develops I may consider open-sourcing it at some
point but for now I think a small donation tier is a good way to give me the
time to work on it.

Minotaur does four important things:

## Power calibration

Minotaur runs a series of tests with a given algorithm on a given device in
order to determine the optimum hashrate at the optimum power level. This process
is highly configuration but I recommend using the defaults.  

These tests can take some time to complete, especially if you run them on all of
the available algorithms, but the results are worth it. We often find that some
devices run algorithms faster at lower power limits, and frequently find that a
lower power limit doesn't affect the hashrate - saving you money.

## Algorithm selection

Minotaur takes four sources of data:

- the current market rates for all of the algorithms on Nicehash
- your calibrated hashrate for each algorithm
- your optimal power limit for each algorithm observed during calibration
- your configured power costs

Using this data it can decide which is the most profitable algorithm to mine
from moment to moment base on market price, hashrate and power consumption vs
power cost. Currently it checks the Nicehash API every 15 seconds.

## Device/algorithm profiles

Minotaur lets you create profiles that target either a device, an algorithm or a
device/algorithm combination. In these profiles you can set:

- power limit (although its best to use the calibration data and not explicitly
set this)
- gpu clock offset
- memory clock offset

We do not endorse or recommend overclocking - do so at your own risk!

## Monitoring

Minotaur has a top-style ncurses interface for monitoring your devices.

Invoke it with:

````
./minotaur --gs
````

![Minotaur](https://a.rkw.io/minotaur.png)


# Compatibility

- Hardware: Nvidia only for now. CPU miner support is planned soon.
- Miners:

- Excavator - https://github.com/nicehash/excavator
- xmrig-nvidia - https://github.com/xmrig/xmrig-nvidia
- ccminer (tpruvot) - https://github.com/tpruvot/ccminer.git
- ccminer2 (alexis78) - https://github.com/alexis78/ccminer.git

Note: "ccminer2" is just the name of the alexis78 fork within Minotaur and
Excavataur.

We can easily add other miners so you are welcome to open github issues with
requests.

# Usage

1. Set up a Xorg display such that you can use nvidia-settings. Make sure the
user you will run Minotaur as is able to use it and change card settings.

2. (Highly recommend but optional) - install and configure Fanotaur:

https://github.com/m4rkw/fanotaur

This will control your GPU fans independently of Minotaur in order to keep
temperatures under control. Minotaur does not ever touch fan speeds, but by
default it will throttle the power limit in order to try keep the GPU under
80C.

3. If you want to use any miners other than Excavator, you'll need to install
and configure Excavataur. This is a related project which provides a
trimmed-down clone of the Excavator JSON API as a wrapper around cli miners.
This shim interface makes it very easy for us to add additional miner tools to
the project.

https://github.com/m4rkw/excavataur

4. Copy minotaur.conf and edit it to your liking (full config reference below).

Minotaur will look for its config file at the following locations:

- /etc/minotaur.conf
- ./minotaur.conf
- ~/.minotaur/minotaur.conf

5. Before you can start mining with Minotaur you need to run the calibration
process for each algorithm/device combination. Currently a device model name is
treated as a device class (eg 1080ti). In the future we plan to add support for
device classes within the same model name. You only need to calibrate each
device class once for each algorithm, so if you have 4 1070s you can distribute
the calibration across them to get it done quicker.


## Calibration

To see the calibration options:

````
$ ./minotaur --calibrate
````

Calibrate all excavator algorithms in eu region on device 0:

````
$ ./minotaur --calibrate 0 excavator all eu
````

Device classes are shortened, so for example "Geforce GTX 1070 Ti" becomes
"1070Ti".

Minotaur's calibration process allows you to automatically determine your
nominal hashrate at the lowest power limit that doesn't compromise the hashrate.
See the config params under the "calibration" section. The most important value
to look at is "acceptable_loss_percent" which defaults to 1%. This means we will
accept a loss of 1% hashrate if we can run an algorithm at a lower power limit.

Calibration files are stored in simple plaintext YAML in ~/.minotaur/.

The full calibration process is:

1. Initial warmup to get the card up to operating temperature

config:

````
calibration:
  initial_warmup_time_mins: 5
````

2. Initial 10-minute run to establish a baseline hashrate

config:

````
calibration:
  initial_sample_time_mins: 10
````

3. If power tuning is enabled Minotaur will then attempt to determine the lowest
power limit that we can run the algorithm with without losing more than our
acceptable hashrate %. The first step is to decrease the power limit to the
level observed at the max power draw in step 2.

4. If the rate doesn't fall by more than the configured loss % then it will keep
going in 10W decrements until the rate falls by more than 1% (or whatever you've
set it to). When this happens it will try dialling back 5W to see if that
mitigates the loss.

5. This process is repeated in 5-min runs until the optimum power limit is
found. In testing we frequently found that hashrates would sometimes go *up*
rather than down when the power limit was lowered.

Relevant config params:

````
calibration:
  initial_warmup_time_mins: 5
  initial_sample_time_mins: 10
  acceptable_loss_percent: 1
  algorithm_start_timeout: 180
  power_tuning:
    enable: true
    decrement_watts: 10
    sample_time_mins: 5
````

Calibration runs are logged to /var/log/minotaur/calibration_{device_id}.log.

If you're just interested in getting up and running as fast as possible you can
specify --quick when calibrating to skip the power calibration phase and
optionally you can also set the warm-up cycle to 0 in the config which will
disable it.


# Mining

Once you have some calibration data for at least one algorithm you can start
mining. Simply run minotaur and it will talk to the Nicehash API and mine the
most profitable algorithms based on the current market rates and your calibrated
hashrates.

Minotaur periodically reloads its calibration data and you can also trigger this
with a HUP which will also reload the config. This means if you start with one
algorithm and then start calibrating others, Minotaur will pick up the new
calibrated algorithms more or less as soon as they are available.

If you want to mine with some cards and calibrate with others you can explicitly
set cards as ignored in the config file:

````
ignore_devices: [0,1]
````

This will ignore GPUs 0 and 1 when running with --mine but you can still target
them with --calibrate. A another way is using the --force parameter to with
--calibrate. By default if a card is mining, --calibrate will refuse to run.
However if you specify --force it will stop the mining worker and mark the card
as being used for calibration until its finished. Minotaur will detect this and
ignore the card until its free for user again.

Although this mostly works, there is the potential for some race condition
issues so I would advise using the ignore_devices setting rather than this. The
benefit of using --force is that if Minotaur is running then the card will be
immediately returned to mining as soon as the calibration run is complete.


# Device profiles

Minotaur allows you to configure device profiles that are assigned to a device
class, an algorithm or a combination of the two. Currently device profiles allow
you to specify a power limit, gpu clock offset and memory clock offset.

In order to change the clocks you will also need to explicitly disable the
"leave_graphics_clocks_alone" setting.

It is recommended not to set the power limit in a device profile but rather use
the power limit obtained via calibration. The power limit set in a device
profile will override that found via calibration. Profile settings from the
default profile are inherited by other profiles.


# GS display

Minotaur comes with a top-style monitoring interface which provides metrics on
all of your devices. You can start it with:

````
minotaur --gs
````

Relevant config params:

````
live_data:
  profitability_averages:
    - 900
    - 3600
    - 86400
  power_draw_averages:
    - 300
    - 600
    - 900
electricity_per_kwh: 0.1194
electricity_currency: GBP
````


# Full configuration reference

````
nicehash.primary_region
````

Primary Nicehash region to use. Currently only eu and usa are supported as the
other regions are just proxies and the payrate data only differentiates between
eu/usa.

````
nicehash.failover_regions
````

A list of Nicehash regions to fail over to if the primary region fails.
Currently only eu and usa are supported.

````
nicehash.user
````

Your nicehash username.

````
nicehash.worker_name
````

Your nicehash worker name

````
nicehash.append_device_id_to_worker_name
````

If enabled this will append the device id of your Nvidia card to the worker
name.

````
nicehash.timeout
````

Timeout for interaction with the Nicehash API


````
nicehash.profit_switch_threshold
````

The profit gain potential in % required before we will switch to a different
algorithm. A value of 0.04 means 4%.

````
nicehash.ports
````

Configured list of ports for Nicehash statrum endpoints.

````
algorithms:
  single:
    - <list>
  double:
    - <list>
````

Algorithms to use with Minotaur. Algorithms will only be used if you have a
miner enabled that supports them and calibrated hashrate data.

````
hashrate_alert_threshold_percent: 3
````

Warn if the observed hashrate for an algorithm is 3% lower than the calibrated
rate.

````
suppress_hashrate_for_seconds: 10
````

Suppress displaying the hashrate for this many seconds after an algorithm starts up.

````
algo_warmup_period_mins: 2
````

Suppress hashrate warnings for this period after an algorithm starts.

````
miners:
  ccminer:
    enable: true
    port: 3457
    ip: 127.0.0.1
    timeout: 10
  excavator:
    enable: true
    ip: 127.0.0.1
    port: 3456
    timeout: 10
````

Here we define the miners that Minotaur will execute. Minotaur will always
choose the most profitable algorithm and miner combination. Excavator is
supported directly via its API, other miners (currently ccminer, ccminer2 and
xmrig-nvidia) are supported via a shim project called Excavataur available here:

https://github.com/m4rkw/excavataur

````
logging:
  log_file: /var/log/minotaur/minotaur.log
  calibration_log_dir: /var/log/minotaur
  max_size_mb: 100
  log_file_count: 7
````

Logging parameters, fairly self-explanatory.

````
live_data:
  profitability_averages:
    - 900
    - 3600
    - 86400
  power_draw_averages:
    - 300
    - 600
    - 900
````

For the GS display - time periods to show profitability and power draw averages for.

````
xorg_display_no: 0
````

This is required for nvidia-settings to work.

````
electricity_per_kwh: 0.1194
electricity_currency: GBP
````

Here you should specify your power costs. These are factored into
algorithm-switching decisions and are also used to calculate profit for the GS
display.

````
calibration:
  initial_warmup_time_mins: 5
  initial_sample_time_mins: 10
  acceptable_loss_percent: 1
  algorithm_start_timeout: 180
  power_tuning:
    enable: true
    decrement_watts: 10
    sample_time_mins: 5
````

Settings for calibration runs. See the calibration section above.

````
leave_graphics_clocks_alone: true
````

Disable this to enable overclocking via device profiles. USE AT YOUR OWN RISK.

````
ignore_devices: [0,1,3]
````

Minotaur will ignore these GPU ids when running with --mine. You can also specify
device classes here

````
regulate_card_temperatures: true
card_temperature_limit: 80
````

If missing or set to true Minotaur will throttle the power limit in order to try
to keep the card at the limit temperature. This only occurs during mining, NOT
during calibration. Also there is no guarantee that it will succeed because it
really depends on the thermal situation the card is in. Minotaur can only
throttle the power down to the minimum power limit of the device.


### Credits

@gordan-bobic for enormous amounts of help
