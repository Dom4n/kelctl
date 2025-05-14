# KELctl

Python 3 library for controlling Korad KEL103/KEL102 electronic loads over a serial connection.

- Built using the [`aenum` library](https://github.com/ethanfurman/aenum).
- Minimum Python version required: 3.10.
- Initially based on the [`py-korad-serial` project](https://github.com/starforgelabs/py-korad-serial).

## Installation

Available as a package on PyPI: [`py-kelctl`](https://pypi.org/project/py-kelctl/).

## Hardware

Supports [Korad KEL103/KEL102](https://www.koradtechnology.com/product/81.html), also sold under other brands like RND.

## Usage

Basic usage examples are included in the `test.py` file.

The object supports the `with` statement for automatic serial port release:

```python
from kelctl import KELSerial

with KELSerial('/dev/ttyACM0') as load:
    print("Model: ", load.model)
    print("Status: ", load.status)
```

## Documentation

### `KELSerial` Class

| Method/Attribute        | Description                                                                                                    | Arguments/Return Type                                     | Notes                                                                                                                                                              |
| ----------------------- | -------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `__init__`              | Constructor.                                                                                                   | `port` (str), `rate` ([BaudRate](#baudrate-class), optional, default=115200), `debug` (bool, optional, default=False) | `debug=True` prints sent/received data.                                                                                                                            |
| `input`                 | Controls or gets the input state.                                                                              | `on()`, `off()`, `get()` (returns [OnOffState](#onoffstate-class)) |                                                                                                                                                                    |
| `memories`              | Array (0-99) for saving and recalling memory settings (1-100 on unit).                                            | `memories[index].save()`, `memories[index].recall()`     | Saved values cannot be directly retrieved. Recalling sets mode and value.                                                                                         |
| `model`                 | Read-only property to get load model information.                                                              | Returns str (e.g., "RND 320-KEL103 V2.60 SN:01234567")     |                                                                                                                                                                    |
| `device_info`           | Read-only property to get device information.                                                                  | Returns multiline str.                                    | Includes network and serial settings.                                                                                                                              |
| `status`                | Property to get current device status.                                                                         | Returns [Status](#status-class) object.                   | Represents `:STAT ?` command output.                                                                                                                               |
| `voltage`               | Sets/gets constant voltage value (Volts). Switches to CV mode when set.                                       | Sets float, gets float or None.                             | Raises [ValueOutOfLimitError](#valueoutoflimiterror-class) if above limits when setting.                                                                           |
| `current`               | Sets/gets constant current value (Amps). Switches to CC mode when set.                                        | Sets float, gets float or None.                             | Raises [ValueOutOfLimitError](#valueoutoflimiterror-class) if above limits when setting.                                                                           |
| `power`                 | Sets/gets constant power value (Watts). Switches to CW mode when set.                                         | Sets float, gets float or None.                             | Raises [ValueOutOfLimitError](#valueoutoflimiterror-class) if above limits when setting.                                                                           |
| `resistance`            | Sets/gets constant resistance value (Ohms). Switches to CR mode when set.                                       | Sets float, gets float or None.                             | Raises [ValueOutOfLimitError](#valueoutoflimiterror-class) if above limits when setting.                                                                           |
| `measured_voltage`      | Read-only property to get measured voltage (Volts).                                                            | Returns float or None.                                    |                                                                                                                                                                    |
| `measured_power`        | Read-only property to get measured power (Watts).                                                             | Returns float or None.                                    |                                                                                                                                                                    |
| `measured_current`      | Read-only property to get measured current (Amps).                                                            | Returns float or None.                                    |                                                                                                                                                                    |
| `function`              | Sets/gets the load's operating mode/function.                                                                  | Sets [Mode](#mode-class) (limited to CV, CC, CR, CW, Short), gets [Mode](#mode-class). | Raises [NoModeSetError](#nomodeseterror-class) when trying to set read-only modes.                                                                               |
| `set_list`              | Sets and saves a list to a save slot and optionally recalls it.                                                | `list` ([LoadList](#loadlist-class)), `recall` (bool, optional, default=True) | Validates the list. Raises ValueError or [ValueOutOfLimitError](#valueoutoflimiterror-class) on failure.                                                           |
| `get_list`              | Recalls and retrieves a list by save slot.                                                                     | `save_slot` (int) -> returns [LoadList](#loadlist-class) | Raises ValueError for invalid save slots (1-7).                                                                                                                    |
| `recall_list`           | Recalls an existing list on the device by save slot.                                                           | `save_slot` (int)                                         | Raises ValueError for invalid save slots (1-7). Recalling non-existing list results in device error.                                                               |
| `set_ocp`               | Sets and saves an OCP list to a save slot and optionally recalls it.                                           | `ocp_list` ([OCPList](#ocplist-class)), `recall` (bool, optional, default=True) | Validates the list. Raises ValueError or [ValueOutOfLimitError](#valueoutoflimiterror-class) on failure.                                                           |
| `get_ocp`               | Recalls and retrieves an OCP list by save slot.                                                                | `save_slot` (int) -> returns [OCPList](#ocplist-class)   | Raises ValueError for invalid save slots (1-10).                                                                                                                   |
| `recall_ocp`            | Recalls an existing OCP list on the device by save slot.                                                       | `save_slot` (int)                                         | Raises ValueError for invalid save slots (1-10). Recalling non-existing list results in device error.                                                               |
| `set_opp`               | Sets and saves an OPP list to a save slot and optionally recalls it.                                           | `opp_list` ([OPPList](#opplist-class)), `recall` (bool, optional, default=True) | Validates the list. Raises ValueError or [ValueOutOfLimitError](#valueoutoflimiterror-class) on failure.                                                           |
| `get_opp`               | Recalls and retrieves an OPP list by save slot.                                                                | `save_slot` (int) -> returns [OPPList](#opplist-class)   | Raises ValueError for invalid save slots (1-10).                                                                                                                   |
| `recall_opp`            | Recalls an existing OPP list on the device by save slot.                                                       | `save_slot` (int)                                         | Raises ValueError for invalid save slots (1-10). Recalling non-existing list results in device error.                                                               |
| `set_batt`              | Sets and saves a Battery test list to a save slot and optionally recalls it.                                 | `batt_list` ([BattList](#battlist-class)), `recall` (bool, optional, default=True) | Validates the list. Raises ValueError or [ValueOutOfLimitError](#valueoutoflimiterror-class) on failure.                                                           |
| `get_batt`              | Recalls and retrieves a Battery test list by save slot.                                                        | `save_slot` (int) -> returns [BattList](#battlist-class) | Raises ValueError for invalid save slots (1-10).                                                                                                                   |
| `recall_batt`           | Recalls an existing Battery test list on the device by save slot.                                              | `save_slot` (int)                                         | Raises ValueError for invalid save slots (1-10). Recalling non-existing list results in device error.                                                               |
| `get_batt_cap`          | Gets the current capacity measured during a battery test.                                                      | Returns float (AH) or None.                               |                                                                                                                                                                    |
| `get_batt_time`         | Gets the current time elapsed during a battery test.                                                           | Returns float (minutes) or None.                          |                                                                                                                                                                    |
| `set_dynamic_mode`      | Sets and saves a dynamic mode list and optionally recalls it.                                                  | `dynamic_list` ([dynamic-list object](#dynamic-lists)), `recall` (bool, optional, default=True) | Validates the list. Raises [ValueOutOfLimitError](#valueoutoflimiterror-class) on failure.                                                                          |
| `get_dynamic_mode`      | Recalls and gets the currently set dynamic mode list.                                                          | Returns [dynamic-list object](#dynamic-lists).            | Raises [InvalidModeError](#invalidmodeerror-class) if the device is not in a dynamic mode.                                                                        |
| `recall_dynamic_mode`   | Recalls a dynamic mode on the device.                                                                          | Returns str.                                              | Required after setting a dynamic mode before use (done by `set_dynamic_mode` by default).                                                                         |
| `settings.beep`         | Controls or gets the "beep" setting.                                                                           | `on()`, `off()`, `get()` (returns [OnOffState](#onoffstate-class)) |                                                                                                                                                                    |
| `settings.lock`         | Controls or gets the "lock" setting (locks/unlocks front panel keys).                                          | `on()`, `off()`, `get()` (returns [OnOffState](#onoffstate-class)) |                                                                                                                                                                    |
| `settings.dhcp`         | Controls or gets the "DHCP" setting.                                                                           | `on()`, `off()`, `get()` (returns [OnOffState](#onoffstate-class)) |                                                                                                                                                                    |
| `settings.trigger`      | Controls or gets the "Trigger" setting (enables/disables external trigger input).                             | `on()`, `off()`, `get()` (returns [OnOffState](#onoffstate-class)) | Does not affect serial or device trigger commands.                                                                                                                 |
| `settings.compensation` | Controls or gets the "Compensation" setting (enables/disables external compensation).                         | `on()`, `off()`, `get()` (returns [OnOffState](#onoffstate-class)) | Memories are unavailable with compensation enabled (according to docs).                                                                                             |
| `settings.ipaddress`    | Sets/gets the device's IP address.                                                                             | Sets str, gets str.                                       | Validates format. Raises ValueError on improper format.                                                                                                              |
| `settings.subnetmask`   | Sets/gets the device's subnet mask.                                                                            | Sets str, gets str.                                       | Validates format. Raises ValueError on improper format.                                                                                                              |
| `settings.gateway`      | Sets/gets the device's gateway address.                                                                        | Sets str, gets str.                                       | Validates format. Raises ValueError on improper format.                                                                                                              |
| `settings.macaddress`   | Sets/gets the device's MAC address.                                                                            | Sets str, gets str.                                       | Validates format (accepts `:` or `-`). Raises ValueError on improper format.                                                                                       |
| `settings.baudrate`     | Sets/gets the device's baud rate.                                                                              | Sets [BaudRate](#baudrate-class) Enum, gets [BaudRate](#baudrate-class) Enum. |                                                                                                                                                                    |
| `settings.port`         | Sets/gets the device's network port.                                                                           | Sets int, gets int.                                       |                                                                                                                                                                    |
| `settings.voltage_limit`| Sets/gets the device's voltage limit (Volts).                                                                   | Sets float, gets float or None.                             |                                                                                                                                                                    |
| `settings.current_limit`| Sets/gets the device's current limit (Amps).                                                                    | Sets float, gets float or None.                             |                                                                                                                                                                    |
| `settings.power_limit`  | Sets/gets the device's power limit (Watts).                                                                     | Sets float, gets float or None.                             |                                                                                                                                                                    |
| `settings.resistance_limit`| Sets/gets the device's resistance limit (Ohms).                                                               | Sets float, gets float or None.                             |                                                                                                                                                                    |
| `settings.factoryreset` | Resets the device to factory settings.                                                                         |                                                           | May disrupt serial connection.                                                                                                                                     |

### `Status` Class

| Attribute  | Description                                 | Type                        |
| ---------- | ------------------------------------------- | --------------------------- |
| `beep`     | State of the "beep" setting.                | [OnOffState](#onoffstate-class) |
| `lock`     | State of the "lock" setting.                | [OnOffState](#onoffstate-class) |
| `trigger`  | State of the "Trigger" setting.             | [OnOffState](#onoffstate-class) |
| `comm`     | Communication status.                       | [OnOffState](#onoffstate-class) |
| `baudrate` | Current baud rate.                          | [BaudRate](#baudrate-class)   |
| `__str__`  | Returns a readable string representation. | str                         |

### `ListStep` Class

Represents a single step in a `LoadList`.

| Attribute       | Description                                             | Type  |
| --------------- | ------------------------------------------------------- | ----- |
| `current`       | Current value for the step.                           | float |
| `current_slope` | Rate at which current changes to the defined value. | float |
| `duration`      | Duration of the step.                                 | float |

### `LoadList` Class

Represents the list function settings.

| Attribute       | Description                                                                     | Type                        | Notes                                                                 |
| --------------- | ------------------------------------------------------------------------------- | --------------------------- | --------------------------------------------------------------------- |
| `save_slot`     | Save slot (1-7) for the list.                                                  | int                         |                                                                       |
| `current_range` | Current limit for values within the list (list values are not restricted by settings limits). | float                       |                                                                       |
| `steps`         | Array of [ListStep](#liststep-class) objects.                                    | list of [ListStep](#liststep-class) | Minimum 2, maximum 84 steps.                                          |
| `loop_number`   | Number of times the steps will be repeated.                                     | int                         |                                                                       |
| `validate`      | Validates the list parameters.                                                  |                             | Raises ValueError or [ValueOutOfLimitError](#valueoutoflimiterror-class) on failure. |
| `__str__`       | Returns the string required to set the list on the device.                      | str                         |                                                                       |

### `OCPList` Class

Represents the OCP function settings.

| Attribute          | Description                                                                                             | Type  | Notes                                                                 |
| ------------------ | ------------------------------------------------------------------------------------------------------- | ----- | --------------------------------------------------------------------- |
| `save_slot`        | Save slot (1-10) for the list.                                                                           | int   |                                                                       |
| `on_voltage`       | Voltage above which the test starts.                                                                    | float |                                                                       |
| `on_delay`         | Delay after reaching `on_voltage` before the test starts.                                              | float |                                                                       |
| `current_range`    | Current limit for values within the list (list values are not restricted by settings limits).          | float |                                                                       |
| `initial_current`  | Current at which the test starts.                                                                       | float |                                                                       |
| `step_current`     | Current value by which each step will decrease.                                                         | float |                                                                       |
| `step_delay`       | Delay after each step before the next step down.                                                        | float |                                                                       |
| `off_current`      | Current below which the test stops.                                                                     | float |                                                                       |
| `ocp_voltage`      | Voltage above which voltage must rise for a successful test.                                            | float |                                                                       |
| `max_overcurrent`  | Maximum current value at which OCP must disengage for a successful test.                                  | float |                                                                       |
| `min_overcurrent`  | Minimum current value at which OCP must disengage for a successful test.                                  | float |                                                                       |
| `validate`         | Validates the list parameters.                                                                          |       | Raises ValueError or [ValueOutOfLimitError](#valueoutoflimiterror-class) on failure (refer to protocol docs for value requirements). |
| `__str__`          | Returns the string required to set the list on the device.                                              | str   |                                                                       |

### `OPPList` Class

Represents the OPP function settings.

| Attribute         | Description                                                                                              | Type  | Notes                                                                 |
| ----------------- | -------------------------------------------------------------------------------------------------------- | ----- | --------------------------------------------------------------------- |
| `save_slot`       | Save slot (1-10) for the list.                                                                            | int   |                                                                       |
| `on_voltage`      | Voltage above which the test starts.                                                                     | float |                                                                       |
| `on_delay`        | Delay after reaching `on_voltage` before the test starts.                                               | float |                                                                       |
| `current_range`   | Current limit for values within the list (list values are not restricted by settings limits).           | float |                                                                       |
| `initial_power`   | Power at which the test starts.                                                                          | float |                                                                       |
| `step_power`      | Power value by which each step will decrease.                                                            | float |                                                                       |
| `step_delay`      | Delay after each step before the next step down.                                                         | float |                                                                       |
| `off_power`       | Power below which the test stops.                                                                        | float |                                                                       |
| `opp_voltage`     | Voltage above which voltage must rise for a successful test.                                             | float |                                                                       |
| `max_overpower`   | Maximum power value at which OPP must disengage for a successful test.                                   | float |                                                                       |
| `min_overpower`   | Minimum power value at which OPP must disengage for a successful test.                                   | float |                                                                       |
| `validate`        | Validates the list parameters.                                                                           |       | Raises ValueError or [ValueOutOfLimitError](#valueoutoflimiterror-class) on failure (refer to protocol docs for value requirements). |
| `__str__`         | Returns the string required to set the list on the device.                                               | str   |                                                                       |

### `BattList` Class

Represents the Battery test function settings.

| Attribute           | Description                                                                     | Type  | Notes                                                                 |
| ------------------- | ------------------------------------------------------------------------------- | ----- | --------------------------------------------------------------------- |
| `save_slot`         | Save slot (1-10) for the list.                                                 | int   |                                                                       |
| `current_range`     | Current limit for values within the list (list values are not restricted by settings limits). | float |                                                                       |
| `discharge_current` | Current at which the battery will be discharged.                                 | float | Must be within `current_range`.                                       |
| `cutoff_voltage`    | Voltage at which the test will stop.                                            | float |                                                                       |
| `cutoff_capacity`   | Capacity (AH) at which the test will stop.                                     | float |                                                                       |
| `cutoff_time`       | Time (minutes) after which the test will stop.                                  | float |                                                                       |
| `validate`          | Validates the list parameters.                                                  |       | Raises ValueError on failure.                                         |
| `__str__`           | Returns the string required to set the list on the device.                      | str   |                                                                       |

### Dynamic Lists

These classes represent the settings for the six dynamic modes.

| Class       | Description                                                                                                | Attributes (`validate` requires `limit` [float])                      | Notes                                                                         |
| ----------- | ---------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `CVList`    | Constant Voltage dynamic mode.                                                                             | `voltage1` (float), `voltage2` (float), `frequency` (float), `duty_cycle` (float) | `validate` checks voltage values against limit and duty cycle (<100%). |
| `CCList`    | Constant Current dynamic mode.                                                                             | `slope1` (float), `slope2` (float), `current1` (float), `current2` (float), `frequency` (float), `duty_cycle` (float) | `validate` checks current values against limit, slopes against device limit, and duty cycle (<100%). |
| `CRList`    | Constant Resistance dynamic mode.                                                                          | `resistance1` (float), `resistance2` (float), `frequency` (float), `duty_cycle` (float) | `validate` checks resistance values against limit and duty cycle (<100%). |
| `CWList`    | Constant Power dynamic mode.                                                                               | `power1` (float), `power2` (float), `frequency` (float), `duty_cycle` (float) | `validate` checks power values against limit and duty cycle (<100%).   |
| `PulseList` | Pulse dynamic mode (triggered).                                                                            | `slope1` (float), `slope2` (float), `current1` (float), `current2` (float), `duration` (float) | `validate` checks current values against limit and slopes against device limit. |
| `ToggleList`| Toggle dynamic mode (switches between two current levels on trigger).                                       | `slope1` (float), `slope2` (float), `current1` (float), `current2` (float) | `validate` checks current values against limit and slopes against device limit. |

Each dynamic list class also has:
- `validate(limit: float)`: Validates against the provided limit and other constraints. Raises [ValueOutOfLimitError](#valueoutoflimiterror-class) on failure.
- `__str__`: Returns the string required to set the list on the device.

### Enums

MultiValueEnums from `aenum` providing more readable names for device values.

| Enum           | Description                                        | Members                                                                                                                                                                                                                                                                                                                        | Notes                                                                                                |
| -------------- | -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------- |
| `Mode`         | Represents device operating modes/functions.       | `constant_voltage ("CV")`, `constant_current ("CC")`, `constant_resistance ("CR")`, `constant_power ("CW")`, `battery ("BATTERY", read only)`, `short ("SHORt")`, `OCP ("OCP", read only)`, `LIST ("LIST", read only)`, `OPP ("OPP", read only)`, `dynamic_cv ("CONTINUOUS CV", read only)`, `dynamic_cc ("CONTINUOUS CC")`, `dynamic_cr ("CONTINUOUS CR")`, `dynamic_cw ("CONTINUOUS CW")`, `dynamic_pulse ("PULSE")`, `dynamic_toggle ("TOGGLE")` | Read-only modes cannot be set using the `function` property.                                         |
| `BaudRate`     | Represents available baud rates.                   | `R9600 (0, 9600)`, `R19200 (1, 19200)`, `R38400 (2, 38400)`, `R57600 (3, 57600)`, `R115200 (4, 115200)`                                                                                                                                                                                                                             | Members have two values (`a` and `b`) representing different formats from the load.              |
| `OnOffState`   | Represents on/off state.                           | `off (0, "OFF", "0")`, `on (1, "ON", "1")`                                                                                                                                                                                                                                                                                           | Members have multiple values representing different string/integer formats from the load.              |

### Custom Errors

| Error Class           | Description                                                                                                  | Attributes        |
| --------------------- | ------------------------------------------------------------------------------------------------------------ | ----------------- |
| `InvalidModeError`    | Raised when trying to get information about a mode that is not currently active (e.g., non-dynamic when getting dynamic info). | `mode`, `message` |
| `ValueOutOfLimitError`| Raised for input values exceeding device limits (checked by the library).                                     | `value`, `limit`, `message` |
| `NoModeSetError`      | Raised when trying to set a mode that is read-only in the `Mode` Enum.                                        | `mode`, `message` |
