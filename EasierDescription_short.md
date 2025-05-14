```markdown
# KELctl

Python 3 library for controlling Korad KEL103/KEL102 electronic loads via serial connection.  
Originally created for [KELgui](https://github.com/vorbeiei/kelgui).  
Uses [aenum library](https://github.com/ethanfurman/aenum). Requires Python ≥ 3.10.  

Package on PyPI: [py-kelctl](https://pypi.org/project/py-kelctl/).  
Hardware info: [Korad KEL103/KEL102](https://www.koradtechnology.com/product/81.html).

## Example Usage

```python
from kelctl import KELSerial

with KELSerial('/dev/ttyACM0') as load:
    print("Model: ", load.model)
    print("Status: ", load.status)
```

## Class & Methods Summary

| Class / Method          | Description                                                                                           | Usage Example / Notes                                                                                                                                  |
|------------------------|---------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| `KELSerial(__init__)`  | Init serial connection to device. Params: port (str), rate (BaudRate Enum, default 115200), debug (bool). | `KELSerial('/dev/ttyACM0')`                                                                                                                            |
| `input`                | Control load input on/off; get state as OnOffState Enum.                                           | `load.input.on()`, `load.input.off()`, `state = load.input.get()`                                                                                      |
| `memories`             | Array of memory slots (0-99) to save/recall load settings (no readback).                          | `load.memories[0].recall()`, `load.memories[99].save()`                                                                                               |
| `model` (property)     | Returns load model string.                                                                         | `load.model` → e.g. `"RND 320-KEL103 V2.60 SN:01234567"`                                                                                              |
| `device_info` (property) | Multiline device info string (IP, MAC, DHCP, etc.).                                              | `load.device_info`                                                                                                                                     |
| `status` (property)    | Returns Status object reflecting current settings and states.                                     | `load.status`                                                                                                                                          |
| `voltage` (property)   | Set/get voltage in CV mode (float, volts). Setting switches mode. Raises `ValueOutOfLimitError`.  | `load.voltage = 30.05`; `v = load.voltage`                                                                                                           |
| `current` (property)   | Set/get current in CC mode (float, amps). Setting switches mode. Raises `ValueOutOfLimitError`.   | `load.current = 10.05`; `i = load.current`                                                                                                           |
| `power` (property)     | Set/get power in CW mode (float, watts). Setting switches mode. Raises `ValueOutOfLimitError`.    | `load.power = 20.05`; `p = load.power`                                                                                                               |
| `resistance` (property)| Set/get resistance in CR mode (float, ohms). Setting switches mode. Raises `ValueOutOfLimitError`.| `load.resistance = 2000.05`; `r = load.resistance`                                                                                                   |
| `measured_voltage`     | Read measured voltage (float, volts). Read-only.                                                  | `v = load.measured_voltage`                                                                                                                           |
| `measured_power`       | Read measured power (float, watts). Read-only.                                                   | `p = load.measured_power`                                                                                                                             |
| `measured_current`     | Read measured current (float, amps). Read-only.                                                  | `i = load.measured_current`                                                                                                                           |
| `function` (property)  | Set/get load mode (Mode Enum). Setting unsupported modes raises `NoModeSetError`.                 | `load.function = Mode.constant_voltage`; `mode = load.function`                                                                                       |
| `set_list`             | Set & optionally recall LoadList on device. Validates input. Raises `ValueError` or `ValueOutOfLimitError`. | `load.set_list(LoadList(...), recall=True)`                                                                                                          |
| `get_list`             | Retrieves LoadList by slot (1-7).                                                                 | `lst = load.get_list(3)`                                                                                                                              |
| `recall_list`          | Recall existing LoadList by slot (1-7). Raises `ValueError` on invalid slot.                      | `load.recall_list(3)`                                                                                                                                  |
| `set_ocp`, `get_ocp`, `recall_ocp` | Same as above but for OCPList (slots 1-10).                                         | `load.set_ocp(OCPList(...))`                                                                                                                          |
| `set_opp`, `get_opp`, `recall_opp` | Same as above but for OPPList (slots 1-10).                                         | `load.set_opp(OPPList(...))`                                                                                                                          |
| `set_batt`, `get_batt`, `recall_batt` | Same as above but for BattList (slots 1-10).                                       | `load.set_batt(BattList(...))`                                                                                                                        |
| `get_batt_cap`         | Get current battery test capacity (float, AH).                                                  | `cap = load.get_batt_cap()`                                                                                                                           |
| `get_batt_time`        | Get battery test elapsed time (float, minutes).                                                 | `time = load.get_batt_time()`                                                                                                                         |
| `set_dynamic_mode`     | Set & recall one of 6 dynamic mode lists (CVList, CCList, CRList, CWList, PulseList, ToggleList). Validates values. | `load.set_dynamic_mode(CVList(...))`                                                                                                                  |
| `get_dynamic_mode`     | Get currently active dynamic mode list. Raises `InvalidModeError` if not in dynamic mode.       | `dyn = load.get_dynamic_mode()`                                                                                                                       |
| `recall_dynamic_mode`  | Recall dynamic mode on device (required after set to activate).                                | `load.recall_dynamic_mode()`                                                                                                                          |
| `settings`             | Access device settings (beep, lock, dhcp, trigger, compensation, ipaddress, subnetmask, gateway, macaddress, baudrate, port, voltage_limit, current_limit, power_limit, resistance_limit, factoryreset). | `load.settings.beep.on()`, `load.settings.ipaddress = "192.168.0.100"`, `load.settings.factoryreset()`                                                 |

## Key Classes and Data Structures

| Class       | Description                                                                                                                          |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------|
| `Status`    | Represents state variables: beep, lock, trigger, comm (OnOffState enums) and baudrate (BaudRate Enum). Can `str()` to readable text.  |
| `ListStep`  | Represents one step in LoadList: current (A), current_slope (A/µs), duration (s).                                                    |
| `LoadList`  | Contains save-slot (1-7), current_range (A), list of ListStep, loop count. Has `validate()` and `str()` to device command string.     |
| `OCPList`   | OCP test settings with save-slot (1-10) and parameters for voltage/current thresholds and timings. Has `validate()` and `str()`.       |
| `OPPList`   | OPP test settings (save-slot 1-10) with power-related parameters. Same interface as OCPList.                                           |
| `BattList`  | Battery test settings with save-slot (1-10), current, cutoff voltage/capacity/time.                                                    |
| Dynamic Lists (`CVList`, `CCList`, `CRList`, `CWList`, `PulseList`, `ToggleList`) | For various dynamic modes with validate(limit) and str() methods.                               |

## Enums

| Enum         | Purpose                                                                                                           | Values / Usage                                                                                      |
|--------------|-------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------|
| `Mode`       | Represents device modes/functions.                                                                               | e.g. `Mode.constant_voltage = "CV"`, `Mode.battery = "BATTERY"`, dynamic modes like `"CONTINUOUS CV"` |
| `BaudRate`   | Baud rates supported by device. Multivalue enum with IDs and rates.                                               | e.g. `BaudRate.R115200 = (4, 115200)`                                                             |
| `OnOffState` | Represents on/off states. Multivalue enum.                                                                       | `off = (0, "OFF", "0")`, `on = (1, "ON", "1")`                                                   |

## Custom Exceptions

| Exception           | Description                                                                                                      | Attributes                                                                                     |
|---------------------|------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------|
| `InvalidModeError`  | Raised when requesting dynamic mode info while device is not in a dynamic mode.                                | `.mode` (current mode), `.message`                                                           |
| `ValueOutOfLimitError` | Raised when setting values outside device limits (voltage, current, resistance, power).                       | `.value` (requested), `.limit` (device limit), `.message`                                     |
| `NoModeSetError`    | Raised when trying to directly set an unsupported mode (read-only in Mode Enum).                              | `.mode` (requested), `.message`                                                               |

---

For detailed examples, refer to the provided `test.py` file in the repository.

Protocol documentation is highly recommended for in-depth understanding of parameter limits and device behavior.
```
