# Miles Per Hour Support Plan

This document summarizes how speed unit conversions are handled in this carrotpilot repository and outlines steps to ensure correct MPH behavior.

## Current Conversion Logic
- Python conversions are defined in `common/conversions.py` where constants such as `MPH_TO_KPH` and `KPH_TO_MPH` are declared.
- C++ conversions mirror these values in `common/util.h`.
- The cruise control logic uses these constants to convert units depending on the `IsMetric` parameter.
- Example constants:
```python
MPH_TO_KPH = 1.609344
KPH_TO_MPH = 1. / MPH_TO_KPH
MS_TO_KPH = 3.6
```
- The UI retrieves `IsMetric` from persistent parameters to determine whether to display speeds in mph or km/h.

## Potential Issues
- If the `IsMetric` flag is inconsistent across modules, speeds may be interpreted in the wrong unit leading to jumps of ~8 mph when 5 mph is expected.
- Carrot-specific additions in `selfdrive/car/cruise.py` also convert button presses using `CV.MPH_TO_KPH` when `is_metric` is false.
- Double conversions could occur if a value already in km/h is multiplied by `CV.MPH_TO_KPH` again before being used.

## Verification Steps
1. **Unit tests** – run `selfdrive/car/tests/test_cruise_speed.py` to validate cruise speed increments.
2. **Parameter check** – ensure the `IsMetric` parameter is stored correctly via the offroad settings screen. Verify UI and control code consistently use this flag.
3. **Code review** – trace all uses of `CV.MPH_TO_KPH`, `CV.KPH_TO_MPH`, `MS_TO_MPH`, and `MS_TO_KPH` across the repo. Key areas include `selfdrive/car/cruise.py` and `selfdrive/controls/controlsd.py`.
4. **Comparison with upstream openpilot** – compare these files with the official openpilot repository to confirm conversion code is unchanged where expected.
5. **Vehicle testing** – on a U.S. vehicle set to mph, test cruise adjustments and verify increments match expectations.

## Recommendations
- Maintain a single source for unit conversions to prevent mismatches.
- Add regression tests for both metric and imperial modes to cover button-based speed changes.
- Document the expected unit flow (user input → internal representation → CAN messages) to avoid double conversion mistakes.
