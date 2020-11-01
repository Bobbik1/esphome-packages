# esphome-packages
A collection of reusable esphome packages

## Package: JB Irrigation

**Required hardware**: [JointBox Heating controller](https://easyeda.com/jointbox/jb-heating-controller_copy)
**Example config**: [irrigation_example.yml](irrigation_example.yml)

Turns [JointBox Heating Controller](https://easyeda.com/jointbox/jb-heating-controller_copy) into irrigation controller. 

### Features

* 4 independent zones capable to controll any AC powered valve
* Ability to connect  ds18b20 temperature sensors
* Ability to complete irrigation cycle after connection loss
* Flood prevention logic

### Operations principle

Controller registers HomeAssistant service which runs irrigation cycle for specified duration. Once esphome recieves the command to start irrigation cycle for specific valve it opens corresponding  valve starts the countdown from `duration` seconds to 0.  Once counter reaches zero valve will be closed. This makes esphome fully responsible for closing valve and minimizes risk to keep valve open for longer then expected because of network Home Assistant failure.

### Protection logic

1. Valve can't be open for longer then `max_irrigation_duration` seconds. Controller will close it automatically.

1. All valves will be closed on start\after reboot.

1. Controller keeps functional and able to finish the cycle even when home assistant connection was lost.

1. Home Assistant event will be emitted in case when irrigation cycle was interrupted because of shutdown\reboot.

1. In case someone opens valve manualy by toggling home assistant switch controller will a) limit open time to `max_irrigation_duration` seconds b) generate special home assistant event

### Configuration

```yaml
# Id of the controller (will be used as domain name)
node_name: switchboard2
# Verbose name of the controller (user friendly)
friendly_node_name: Switchboard dev. 2
# Log verbosity
log_level: DEBUG

# Preffix to add to each entity id generated by this package (e.g. valve switch)
prefix_entity_id: "irrigation_test"
# Preffix to add to each entity name generated by this package (e.g. valve switch)
prefix_name: "IrrigationTest"
# Wheather or not to expose valves to home assistant (enables\disables manual valve control)
not_expose_valves: "false"
# Max duration of the irrigation cycle in seconds
# 2 hours = 2 * 3600 sec = 7200 sec
max_irrigation_duration: '7200'

# Zones
zone1_id: zone1
zone1_name: Zone1

zone2_id: zone2
zone2_name: Zone2

zone3_id: zone3
zone3_name: Zone3

zone4_id: zone4
zone4_name: Zone4

```

### Home Assistant Usage

In order to start irrigation cycle invoke the serive like this:

Service: `esphome.NODENAME_start_irrigation`
Data:
```yaml
valve_id: PREFIX_zone1_valve
# Duration is in seconds
duration: 30
```

#### Notifications

If you would like to configure notifications you might consider the following automation examples.

```yaml
- alias: 'Irrigation: Irrigation failure detected'
  trigger:
    platform: event
    event_type: esphome.irrigation_failure

  action:
    service: notify.telegram
    data_template:
      message: >


        It seems like controller {{trigger.event.data.controller_id}} was interrupted\rebooted during
        irrigation cycle so it wasn't completed. You could check valve state history to identify actual
        duration of the cycle.
      title: '🌊 ⚠️ *Attention: Irrigation failure detected!*'

- alias: 'Irrigation: Irrigation valve opened manually'
  trigger:
    platform: event
    event_type: esphome.irrigation_manual_valve_open

  action:
    service: notify.telegram
    data_template:
      message: >


        For some reason valve {{trigger.event.data.zone_name}} was opened manually. This is dangerous and generally
        is not recommended as the time for irrigation is unlimited. To prevent flood controller applied time limit
        automatically ({{trigger.event.data.limit}} seconds).


        Please take an action if this happened accidentally.
      title: '🌊 ⚠️ *Valve was opened manually*'

- alias: 'Irrigation: Irrigation started'
  trigger:
    platform: event
    event_type: esphome.irrigation_started

  action:
    service: notify.telegram
    data_template:
      message: >
        🌊 Irrigation started for zone {{trigger.event.data.zone_name}}


- alias: 'Irrigation: Irrigation finished'
  trigger:
    platform: event
    event_type: esphome.irrigation_stopped

  action:
    service: notify.telegram
    data_template:
      message: >
        🌊 ✔️ Irrigation finished for zone {{trigger.event.data.zone_name}}
```

# Credits
Dmitry Berezovsky

# Disclaimer
This module is licensed under GPL v3. This means you are free to use in non-commercial projects.

The GPL license clearly explains **that there is no warranty** for this free software. Please see the included LICENSE file for details.

