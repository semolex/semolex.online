+++
date = '2025-03-18T18:55:45+02:00'
draft = false
title = 'Writing Zha Quirks'
+++

## Introduction
If you are up-to home automation and using Zigbee devices, you probably heard about ZHA (Zigbee Home Automation). It's a Python library that allows you to interact with Zigbee devices using Zigpy. It's a great library, but it has some quirks that you should be aware of.
<!--more-->
Pun intended. It has QUIRKS. WHich is like the custom "translator" for unsupported devices.
And if you wrote a good one it might end up in the official repository.
There a just few tutorials on how to write a quirk, so I decided to write one myself.

## Starting a Quirk
I assume you know what we are talking about.

Also, I assume you are using Home Assistant.
Please find how to "interview" your device to get datapoints/endpoints/clusters etc.
So to start adding a quirk you need to add a configuration to your `configuration.yaml` file.

```yaml
zha:
  enable_quirks: true
  custom_quirks_path: custom_components/zha_quirks/
```
You path might be different, but you get the idea.

Then you place your quirks in the `custom_components/zha_quirks/` directory.
Let me guess - you are probably found this because you are using Tuya device :)

## Quirk Structure
There are few version of quirks. I ended up with using the lates approach.
First of all, clone the official repository:
```bash
git clone git@github.com:zigpy/zha-device-handlers.git
```
Then you can find the quirks in the `zha-device-handlers/zhaquirks` directory.
Take a look at the manufacturers, structure, code itself.
Now you have 2 ways:
1. Copy the quirks you need to your `custom_components/zha_quirks/` directory and modify them.
2. Create a new quirk from scratch.

I recommend to add your quirks to your fork of the repository and then create a PR to the official repository in any case.

## Example

For example this is an example of a quirk for a Tuya thermostat that is not supported by ZHA (Zigbee2MQTT either as of March 2025):
```python
"""Tuya TS0601 Thermostat with EP Attribute Fix."""

import logging

from zigpy.quirks.v2.homeassistant import UnitOfTemperature
from zigpy.types import t
from zigpy.zcl.clusters.hvac import RunningState, Thermostat

from zhaquirks.tuya.builder import TuyaQuirkBuilder
from zhaquirks.tuya.mcu import TuyaAttributesCluster

_LOGGER = logging.getLogger(__name__)


class ProgramMode(t.enum8):
    """Tuya program mode enum."""
    Manual = 0x00
    Program = 0x01


class SensorMode(t.enum8):
    """Tuya sensor mode enum."""
    Internal = 0x00
    External = 0x01
    Both = 0x02

class RunningMode(t.enum8):
    """Tuya running mode enum."""
    Heat = 0x00
    Cool = 0x01


class TuyaThermostat(Thermostat, TuyaAttributesCluster):
    """Tuya local thermostat cluster."""

    _CONSTANT_ATTRIBUTES = {
        Thermostat.AttributeDefs.ctrl_sequence_of_oper.id: Thermostat.ControlSequenceOfOperation.Heating_Only
    }

    def __init__(self, *args, **kwargs):
        """Init a TuyaThermostat cluster."""
        super().__init__(*args, **kwargs)
        self.add_unsupported_attribute(
            Thermostat.AttributeDefs.setpoint_change_source.id
        )
        self.add_unsupported_attribute(
            Thermostat.AttributeDefs.setpoint_change_source_timestamp.id
        )
        self.add_unsupported_attribute(Thermostat.AttributeDefs.pi_heating_demand.id)


# Tuya thermostat quirk with fixed missing ep_attribute
(
    TuyaQuirkBuilder("_TZE200_6kijc7nd", "TS0601")
    # System mode (on/off) - DP 1
    .tuya_dp(
        dp_id=1,
        ep_attribute=TuyaThermostat.ep_attribute,
        attribute_name=TuyaThermostat.AttributeDefs.system_mode.name,
        converter=lambda x: {
            True: Thermostat.SystemMode.Heat,
            False: Thermostat.SystemMode.Off,
        }[x],
        dp_converter=lambda x: {
            Thermostat.SystemMode.Heat: True,
            Thermostat.SystemMode.Off: False,
        }[x],
    )
    # Mode (manual/program) - DP 2
    .tuya_enum(
        dp_id=2,
        attribute_name="preset_mode",
        enum_class=ProgramMode,
        translation_key="preset_mode",
        fallback_name="Preset mode",
    )
    # Working status - DP 3
    .tuya_dp(
        dp_id=3,
        ep_attribute=TuyaThermostat.ep_attribute,
        attribute_name=TuyaThermostat.AttributeDefs.running_state.name,
        converter=lambda x: RunningState.Heat_State_On if x else RunningState.Idle,
    )
    # Window check - DP 8
    .tuya_binary_sensor(
        dp_id=8,
        attribute_name="window_detection",
        translation_key="window_detection",
        fallback_name="Window Detection",
    )
    # Frost protection - DP 10
    .tuya_binary_sensor(
        dp_id=10,
        attribute_name="frost_protection",
        translation_key="frost_protection",
        fallback_name="Frost Protection",
    )
    # Set temperature - DP 16
    .tuya_dp(
        dp_id=16,
        ep_attribute=TuyaThermostat.ep_attribute,
        attribute_name=TuyaThermostat.AttributeDefs.occupied_heating_setpoint.name,
        converter=lambda x: x * 10,  # Convert from 0.1°C to 0.01°C
        dp_converter=lambda x: x // 10,  # Convert from 0.01°C to 0.1°C
    )
    # Set temperature ceiling - DP 19 - FIXED
    # MANY LINES OMMITED
    .adds(TuyaThermostat)
    .skip_configuration()
    .add_to_registry()
)
```

## Conclusion
This is just a simple example of a quirk. You can add more features, more attributes, more enums, more binary sensors, etc.)
Also note that sometimes it is just enough to add a name/ID of your device to the quirks and it will work out of the box.
