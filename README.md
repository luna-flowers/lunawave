# Magic Hue lightbulb CLI utility

## Usage
### lunawave
```bash
USAGE:
    lunawave [OPTIONS]

OPTIONS:
    -p, --program    Runs the specified program (builtin or custom function): vaporwave, off
```

### MagicHue
Once your lightbulb is on your network, you can use the `magichue.py` script with the following arguments:
```
  -h, --help  show help message and exit
  -ip         provide the IP for the lightbulb; i.e. -ip 192.168.2.2
  -raw        accept colon separated raw hex string; i.e. -raw 71:23:0f
  -rgb        accept comma separated rgb values; i.e. -rgb 100,155,75
  -warm       accept value of warm white (0-255); i.e. -warm 150
  -power      accept 'on' or 'off'; i.e. -power on
  -status     get the bulb's status
```
The user must provide the lightbulb's IP address and either comma separated RGB values or a colon separated list of HEX values. An updated list of known combinations and rules of HEX values can be found below.

#### RGB Example:
This will set the lightbulb to be red:
`python3 magichue.py -ip 192.168.2.2 -rgb 200,0,0`

#### WARM WHITE Example:
This will set the lightbulb to half power of warm white:
`python3 magichue.py -ip 192.168.2.2 -warm 123`

#### RAW Hex Example:
This will turn the lightbulb on with its last setting:
`python3 magichue.py -ip 192.168.2.2 -raw 71:23:0f`

## Understanding the bulb's HEX codes:
I've been able to sniff a few codes, which the app sends to the lightbulb. The structure of the hex list seems to follow a pattern. The first bit defines the type of action. Then follow informational bits, which define colors, brightness, etc. The entire list is followed by a checksum bit, masked by 255. For example, this is the broken down list for turning the bulb on:

|Action bit|Body bit list|Finish bit|Checksum bit|
|---|---|---|---|
|`71`|`24`|`0f`|`a4`|

And here is how the data looks for magenta / purple color:

|Action bit|Body bit list|Finish bit|Checksum bit|
|---|---|---|---|
|`31`|`ff:2f:ff:00:00:f0`|`0f`|`5d`|

**It's important to note that this script will calculate the checksum bit. You MUST NOT include it in the -raw argument.**

There are several different types of commands (Action Bits) that the bulb seems to accept:
`31` - Color options
`61` - Pulse / gradual options
`71` - Power options
`81` - Status options

The `31` and `61` action bit commands seem to follow a structure in the body bit list: 
`31:RR:GG:BB:?RGBSECTIONEND?:?BRIGHTNESS?:?WARMTH?:0f`
`61:PRESET:SPEED:0f`

The following examples can help you futher undertand the structure of the body bit list:

**HEX examples**
*Note that the final checksum bit is not included here. The script will add it automatically.*
|Action Type|Values|
|---|---|
|On|`71:23:0f`|
|Off|`71:24:0f`|
|||
|RGB (255,47,255)|`31:ff:2f:ff:00:00:f0:0f`|
|RGB (255,126,0)|`31:ff:7e:00:00:00:f0:0f`|
|RGB (0,79,255)|`31:00:4f:ff:00:00:f0:0f`|
|||
|Warm White 100%|`31:00:00:00:ff:0f:0f`|
|Warm White 50%|`31:00:00:00:80:0f:0f`|
|Warm White 1%|`31:00:00:00:02:0f:0f`|
|||
|7 Color range, 100% speed|`61:25:1f:0f`|
|7 Color range, 50% speed|`61:25:10:0f`|
|7 Color range, 1% speed|`61:25:01:0f`|
|Red gradual, 100% speed|`61:26:1f:0f`|
|Red gradual, 50% speed|`61:26:10:0f`|
|Green gradual, 100% speed|`61:27:1f:0f`|
|Blue gradual, 100% speed|`61:28:01:0f`|
|First custom setting, 100% speed|`61:60:01:0f`|
