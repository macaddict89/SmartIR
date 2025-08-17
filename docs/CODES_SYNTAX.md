# SmartIR Devices Codes Files syntax

## Disclaimer

- I do not provide any support for you to record your own device codes. Please do not create any issues to ask how to do it as it will immediately closed.
- [You can ask for help at SmartIR Climate (Home Assistant Community)](https://community.home-assistant.io/t/smartir-control-your-climate-tv-and-fan-devices-via-ir-rf-controllers/).
- If you think something is missing or wrong in the docs and you would like to submit PR with docs amendments you are welcome.

## Device codes JSON file

If you decide to modify or create your own device file you need to do it in the `custom_codes/[climate|fan|media_player|/` directory. Any changes inside `codes` directory will be lost upon SmartIR HACS update!

- If you need to modify existing file from the `codes` directory please copy it into `custom_codes` first.
- If you need to create new device file please choose free for digit file name (not used by existing json file int the `codes/your_class` directory) and create json file with given name in the `custom_codes/your_class`

Each different class (Climate, Fan, Media Player) device file has common part and then class specific part. Please take look to some existing files as examples. It is very easy to make error in JSON file structure when editing. Use any available JSON format checker to validate it after editing - for example [JSON Lint](https://jsonlint.com/)

## Common declaration part

```yaml:
{
    "manufacturer": "Toyotomi",
    "supportedModels": [
        "AKIRA GAN/GAG-A128 VL"
    ],
    "supportedController": "Broadlink",
    "commandsEncoding": "Base64",
```

| json attribute        | mandatory |        type        | description                                                                                                                                                   |
| --------------------- | :-------: | :----------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `manufacturer`        |   `yes`   |      `string`      | Manufacturer name of the device you intend to control                                                                                                         |
| `supportedModels`     |   `yes`   | `array of strings` | List of devices names supported by this file                                                                                                                  |
| `supportedController` |   `yes`   |      `string`      | Controller type used. Can be one of the `Broadlink`, `Xiaomi`, `MQTT`, `LOOKin`, `ESPHome`, `ZHA`, `UFOR11`                                                   |
| `commandsEncoding`    |   `yes`   |      `string`      | Each controller type supports given type/s of the IR command encodings. For actual supported encodings for your controller refer to the `controller_const.py` |

## Climate specific

### Climate declaration part

```yaml:
    "temperatureUnit": "C",
    "minTemperature": 16,
    "maxTemperature": 30,
    "precision": 1,
    "operationModes": [
        "auto"
        "heat",
        "cool",
        "heat_cool",
        "fan_only",
        "dry"
    ],
    "presetModes": [
        "none",
        "eco",
        "hi_power"
    ],
    "fanModes": [
        "low",
        "mid",
        "high",
        "auto"
    ],
    "swingModes": [
        "off",
        "",
        "high",
        "auto"
    ],
```

| json attribute    | mandatory |        type        | description                                                                                                                                                                                                                                                                                                                |
| ----------------- | :-------: | :----------------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `temperatureUnit` |   `yes`   |      `string`      | Temperature unit used by you device (this can be different to the HA default temperature unit). Can be one of rhe `C`, `F`, `K`                                                                                                                                                                                            |
| `minTemperature`  |   `yes`   |    `int, float`    | Minimum device supported temperature. Can be float or integer depending on the `precision`                                                                                                                                                                                                                                 |
| `maxTemperature`  |   `yes`   |    `int, float`    | Maximum device supported temperature. Can be float or integer depending on the `precision`                                                                                                                                                                                                                                 |
| `precision`       |   `yes`   |    `int, float`    | This is size of the temperature steps you can set in the target temperature (if you set you temperature in 1 degree steps it will be `1`, if in half degrees it will be `0.5`, etc). Supported precisions are `2`, `1`, `0.5`, `0.1`                                                                                       |
| `operationModes`  |   `yes`   | `array of strings` | List of the modes your device can operate in. This has to be subset of the HA climate [HVAC modes](https://developers.home-assistant.io/docs/core/entity/climate#hvac-modes). No customs modes are possible! Do not list `off` mode here it is included by default.                                                        |
| `presetModes`     |   `no`    | `array of strings` | List of the preset modes your device can operate in. Does you device support different ways to operate under any given operationMode? For example can your AC units can do `eco` and `hi-power` cooling? Then those shall be listed here. The preset mode names could be freely defined to match naming used by your device. |
| `fanModes`        |   `no`    | `array of strings` | List of the fan modes your device can operate in. Does you device support fan modes? High, low, auto, etc.? Then those shall be listed here. The fan mode names could be freely defined to match naming used by your device.                                                                                              |
| `swingModes`      |   `no`    | `array of strings` | List of the fan swing modes your device can operate in. Does you device support different ways how your fan grills move? Then those shall be listed here. The swing mode names could be freely defined to match naming used by your device.                                                                               |

### Climate commands

Now when all modes supported by your device (operation, presets, fan, swing) are declared, you need to record IR command for their combination for any target temperature. There may be some tools to help you to help with the process.

#### Climate `off` commands

Most of the controller devices have a single `off` command, which is used to switch the device off. In some more rare cases, there is a different command for each operational mode the device is currently working in. So for `cool` operational mode there could be `off_cool` command, for `heat` operational mode `off_heat`, etc. You need to have either `off` command or `off_MODE` command for all you operational codes declared.

```yaml:
    "commands": {
        "off": "JgCSAAABKHQKDGgDTFS.............",
```

or

```yaml:
    "commands": {
        "off_cool": "JgCSAAABKHQKDGgDEht.............",
        "off_heat": "JgCSAAABKHQKDGgDFth.............",
        "off_dry": "JgCSAAABKHQKDGgDFht.............",
        ...
```

#### Climate `on` commands

In some cases, controlled device have a dedicated `on` command (Otherwise the device is switched on by the operation command and a dedicated IR code to switch it on is not required). In case your `on` and `off` command are the same, SmartIR will not send `on` or `off` command again, if the device is assumed to be already in desired state. As without power sensor devices state is only assumed it is highly suggested to use the power sensor feature.

```yaml:
    "commands": {
        "on": "JgCSAAABKHQKDGgHtSq.............",
```

#### Climate operation commands

These are command to set controlled device into the desired work state. Due to the nature of how HVAC units works, all the modes and temperature are contained in a single IR command. Therefore, you need to record and declare IR commands for all the combinations of modes and temperatures. The complete structure would look like:

`operation mode` -> `preset mode` -> `fan mode` -> `swing mode` -> `temperature` -> `recorded IR command`
ƒ
```yaml:
    "commands": {
        "cool": {
            "eco": {
                "low": {
                    "swing": {
                        "16": "JgCSAAABKZIXEBcRFz.............",
                        "17": "JgCSAAABKHQKDGgDHz.............",
                        ...
                    },
```

- If your device doesn't support some of the modes types at all (and you did not declare them in the declaration part - for example presetModes are not supported) then simply skip given level (`operation mode` -> `fan mode` -> `swing mode` -> `temperature` -> `recorded IR command`). Only `operation mode` level is required, all lower levels are optional.

```yaml:
    "commands": {
        "cool": {
            "low": {
                "swing": {
                    "16": "JgCSAAABKZIXEBcRFz.............",
                    "17": "JgCSAAABKHQKDGgDHz.............",
                    ...
                },
```

- If your device doesn't support some mode under a given higher level mode (for example temperatures are not supported under fan_only operation mode, or presetModes are not supported under dry operation mode), you have three possibilities how to address this:

  1. Use `-` as a 'catch all' mode name/temperature. Using this approach has additional behavior - if you change mode/temperature to a value covered by `-` the value will not change in HA. Example: temperatures are not supported under fan_only operation mode - when you try to change temperature using the `-` key the temperature in HA stays same:

  ```yaml:
  "commands": {
      "fan_only": {
          "-": {
              "low": {
                  "swing": {
                      "-": "JgCSAAABKZIXEBcRFz............."
                  },
  ```

  2. Define some default mode (i.e. something like `none` or `default` but any name will do) as first in the declaration section and use it as default failover mode. Example: if you are running operation mode `cool` with preset mode `eco` and you change operation mode to `dry` which doesn't support presets, then it will try to select first available preset in the order presets are declared in the `presetModes` - and it will therefore choose your first one - `none`:

  ```yaml:
  "commands": {
      "cool": {
          "eco": {
              "low": {
                  "swing": {
                      "16": "JgCSAAABKZIXEBcRFz............."
                  },
          ...
      "dry": {
          "none": {
              "low": {
                  "swing": {
                      "16": "JgCSAAABKZIXEBcRFz............."
                  },
  ```

  3. Duplicate same IR command to all keys. This approach is not suggested and multiple occurence of the same IR command will be logged by the validator to the HA log. Example: temperature setting is not supported in the fan_only mode and all commands are same:

  ```yaml:
  "commands": {
      "fan_only": {
          "normal": {
              "low": {
                  "swing": {
                      "16": "JgCSAAABKZIXEBcRFz.............",
                      "17": "JgCSAAABKZIXEBcRFz.............",
                      "18": "JgCSAAABKZIXEBcRFz.............",
                      ...
                  },
  ```

## Light Specific

### Light declaration part

```yaml:
    "brightness": [26, 51, 77, 102, 128, 153, 179, 204, 230, 255],
    "colorTemperature": [
      2700, 4600, 6500
    ],
```

| json attribute     | mandatory |      type      | description                                                                 |
| ------------------ | :-------: | :------------: | --------------------------------------------------------------------------- |
| `brightness`       |   `no`    | `array of int` | List of supported brightness levels for the light in the range of 0 to 255. |
| `colorTemperature` |   `yes`   | `array of int` | List of supported color temperatures for the light in Kelvin (K).           |

### Light commands

#### Light `off` commands

The off command specifies the IR code used to turn off the light. This command is necessary for any basic light control setup.

```yaml:
    "commands": {
        "off": "JgCSAAABKHQKDGgHtSq.............",
```

#### Light `on` commands

The on command is used to power on the device. In some cases, a dedicated on command is not required if the device can be turned on by setting a brightness or color temperature value.
Since the device state is only assumed without a power sensor, it is highly recommended to use the power sensor feature in this case.

```yaml:
    "commands": {
        "on": "JgCSAAABKHQKDGgHtSq.............",
```

#### Light `night` commands

If a night command is specified, there is a special case when the requested brightness is 1, using the defined command:

```yaml:
"commands": {
    "night": "JgCSAAABKZIXEBcRFz.............",
```

#### Light brightness control

The brightness control functionality enables dynamic adjustments to the light’s intensity. This can be achieved through general brighten and dim commands or by specifying individual commands for each supported brightness level.

##### Light brighten and dim commands

The brighten and dim commands incrementally adjust the brightness up or down, respectively.

```yaml:
"commands": {
    "brighten": "JgCSAAABKZIXEBcRFz.............",
    "dim": "JgCSAAABKZIXEBcRFz.............",
```

##### Light brightness commands

Alternatively, each supported brightness level can have a dedicated command.

```yaml:
"commands": {
    "brightness": {
      "26": "JgCSAAABKZIXEBcRFz.............",
      "51": "JgCSAAABKZIXEBcRFz.............",
      ...
    }
```

#### Light color temperature control

Color temperature control allows users to adjust the warmth or coolness of the light. Similar to brightness, this can be managed with general commands or specific values.

##### Light colder and warmer commands

The colder and warmer commands provide a general way to adjust the color temperature incrementally.

```yaml:
"commands": {
    "colder": "JgCSAAABKZIXEBcRFz.............",
    "warmer": "JgCSAAABKZIXEBcRFz.............",
```

##### Light colorTemperature commands

Alternatively, commands can be defined for each supported color temperature value.

```yaml:
"commands": {
    "colorTemperature": {
      "2700": "JgCSAAABKZIXEBcRFz.............",
      "4600": "JgCSAAABKZIXEBcRFz.............",
      "6500": "JgCSAAABKZIXEBcRFz.............",
    }
```

## Fan Specific

TBD

## Media Player specific

TBD
