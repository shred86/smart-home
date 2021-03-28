
# Home Assistant Security System with Konnected.io

This is the setup I used to create a security system within [Home Assistant](http://home-assistant.io) using [Konnected.io](http://konnected.io) which replaced a [Concord 4](https://www.interlogix.com/intrusion/product/concord-4) traditional security system. It had hardwired door/window sensors, motion sensors and a siren that I was able to use with konnected.io.

## Create manual alarm control panel

Add the following to the configuration.yaml file to create a [manual alarm control panel](https://www.home-assistant.io/integrations/manual/):

```
alarm_control_panel:
  - platform: manual
    name: Security System
    arming_time: 0
    delay_time: 30
    trigger_time: 600
    armed_away:
      arming_time: 30
    disarmed:
      trigger_time: 0
```

Some highlights for this setup: 1) 30 second arming delay only when set to armed_away, 2) 30 second delay timer (i.e. pending state) when the alarm is tripped and 3) the alarm panel will never set the state to triggered if in a disarmed state.

## Add konnected.io integration

Configure zones, ALARM1, OUT1 and OUT2/ALARM2 as desired.

For ALARM1, set "Output when on" to high and leave everything else defaulted to 0.

For OUT1, create two states. The first one is a single beep with a 24 ms pulse duration which will be used in an automation for a count down timer. The second one is a double beep with a 24 ms pulse duration, 54 ms pause between pulses and repeats twice, which will be used when doors or windows are opened.

## Create automations for core functionality

### Automations: Trigger

**Trigger the security system when armed_away**
* Name: Security System: Trigger when armed_away
* Triggers: Door/windows, motion sensors
* Condition:
  * Condition type: State
  * Entity: alarm_control_panel.security_system
  * State: armed_away
* Action:
  * Action type: Call service
  * Service: alarm_control_panel.security_system
  * Targets: Security System

**Trigger the security system when armed_home**
* Name: Security System: Trigger when armed_home
* Triggers: Door/windows (with exceptions as desired)
* Condition:
    * Condition type: State
    * Entity: alarm_control_panel.security_system
    * State: armed_home
* Action:
    * Action type: Call service
    * Service: alarm_control_panel.security_system
    * Targets: Security System

**Trigger the security system when armed_night**
* Name: Security System: Trigger when armed_night
* Triggers: Door/windows (with exceptions as desired)
* Condition:
    * Condition type: State
    * Entity: alarm_control_panel.security_system
    * State: armed_night
* Action:
    * Action type: Call service
    * Service: alarm_control_panel.security_system
    * Targets: Security System

### Automations: Notifications

**Send a mobile notification when the security system is armed to armed_away**
* Name: Security System: Notify when armed_away
* Triggers:
    * Trigger type: State
    * Entity: alarm_control_panel.security_system
    * To: armed_away
* Action:
    * Action type: Call service
    * Service: notify.mobile_app_*
    * Data: message: Security system armed to away mode

**Send a mobile notification when the security system is armed to armed_home**
* Name: Security System: Notify when armed_home
* Triggers:
    * Trigger type: State
    * Entity: alarm_control_panel.security_system
    * To: armed_home
* Action:
    * Action type: Call service
    * Service: notify.mobile_app_*
    * Data: message: Security system armed to home mode

**Send a mobile notification when the security system is armed to armed_night**
* Name: Security System: Notify when armed_night
* Triggers:
    * Trigger type: State
    * Entity: alarm_control_panel.security_system
    * To: armed_night
* Action:
    * Action type: Call service
    * Service: notify.mobile_app_*
    * Data: message: Security system armed to night mode

**Send a mobile notification when security system is disarmed**
* Name: Security System: Notify when disarmed
* Triggers:
    * Trigger type: State
    * Entity: alarm_control_panel.security_system
    * To: disarmed
* Action:
    * Action type: Call service
    * Service: notify.mobile_app_*
    * Data: message: Security system disarmed

**Send a mobile notification when security system is pending**
* Name: Security System: Notify when pending
* Triggers:
    * Trigger type: State
    * Entity: alarm_control_panel.security_system
    * To: pending
* Action:
    * Action type: Call service
    * Service: notify.mobile_app_*
    * Data: message: Security system trigger delay in progress!

**Send a mobile notification when security system is triggered**
* Name: Security System: Notify when triggered
* Triggers:
    * Trigger type: State
    * Entity: alarm_control_panel.security_system
    * To: triggered
* Action:
    * Action type: Call service
    * Service: notify.mobile_app_*
    * Data: message: Security system has been triggered!

### Automations: Actions

**Create a count down beeper when the security system is pending**
* Name: Security System: Action when pending
* Triggers:
    * Trigger type: State
    * Entity: alarm_control_panel.security_system
    * To: pending
* Action:
    * Action type: Repeat
    * Repeat type: Until
      * Until conditions:
        * Condition type: State
        * Entity: alarm_control_panel.security_system
        * State: disarmed
      * Actions:
        * Action type: Call service
        * Service: switch.toggle
        * Targets: switch.beeper
        * Action type: Delay (1 second)

**Turn on the siren and lights when security system is triggered**
* Name: Security System: Action when triggered
* Triggers:
    * Trigger type: State
    * Entity: alarm_control_panel.security_system
    * To: triggered
* Action:
    * Action type: Call service
    * Service: switch.turn_on
    Targets: Alarm Siren, Switches, Lights

**Turn off the siren when security system is disarmed**
* Name: Security System: Action when disarmed
* Triggers:
    * Trigger type: State
    * Entity: alarm_control_panel.security_system
    * From: triggered
    * To: disarmed
* Action:
    * Action type: Call service
    * Service: switch.turn_off
    * Targets: Alarm Siren

## Create automations for misc functionality

**Sound double beeper on door/window open when system disarmed**
* Name: Security System: Sound double beeper on door/window open
* Triggers:
    * Trigger type: State
    * Entity: All window and door sensors
    * To: on
* Conditions:
    * Condition type: State
    * Entity: alarm_control_panel.security_system
    * State: disarmed
* Action:
    * Action type: Call service
    * Service: switch.turn_on
    * Targets: Double Beeper

_Note: The reason this only functions in the disarmed state is because I was having an issue with the count down timer automation if I tried to have the double beeper and the count down beeper operating at the same time since they both use the same piezo sounder device._

## Make notifications actionable

Add the following to the configuration.yaml to create a category (type of notification) and action:

```
ios:
  push:
    categories:
      - name: "Security System"
        identifier: 'security_system_notification'
        actions:
          - identifier: 'DISARM_SECURITY_SYSTEM'
            title: 'Disarm'
            authenticationRequired: true
            destructive: true
```

The `authenticationRequired` and `destructive` lines are optional. See the [Home Assistant Actionable Notifications documentation](https://companion.home-assistant.io/docs/notifications/actionable-notifications) for more details.

Modify the existing "Security System: Notify when pending" and "Security System: Notify when triggered" automations that were created above to include the actionable notification by adding the following data to the `notify.mobile_app_*` service.

```
push:
  category: security_system_notification
  sound:
    name: default
    critical: 1
    volume: 1
```

Lastly, create an automation that will perform the desired behavior when the actionable notification action is selected.

* Name: Security System: Disarm from actionable notification
* Triggers:
    * Trigger type: Event
    * Event type: ios.notification_action_fired
    * Event data: actionName: DISARM_SECURITY_SYSTEM
* Action:
    * Action type: Call service
    * Service: alarm_control_panel.alarm_disarm
    * Targets: Security System
