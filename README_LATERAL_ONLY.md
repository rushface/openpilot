# Openpilot Lateral-Only Patch for Non-SCC Hyundai Vehicles

This is a patched version of [phr00t/openpilot@new_scc](https://github.com/phr00t/openpilot/tree/new_scc) specifically designed for **Hyundai vehicles WITHOUT Smart Cruise Control (SCC)** that want lateral steering control while maintaining full functionality of their stock cruise control.

## What This Fixes

The base `new_scc` fork attempts to simulate SCC functionality by injecting CAN messages and spamming cruise control buttons. For vehicles without SCC hardware, this causes:
- ❌ Cruise speed jumping to 80-100+ mph uncontrollably
- ❌ Random CAN errors and dashboard warning lights ("Christmas tree" effect)
- ❌ Conflicts between openpilot and stock cruise control
- ❌ Errors especially noticeable at stoplights

This patch disables all SCC simulation when `OpkrVariableCruise` is set to `0` (off), allowing:
- ✅ Lateral steering control (lane keeping) via SET button
- ✅ Stock cruise control working independently and normally
- ✅ Clean CAN bus with no error messages
- ✅ Auto lane changes

## Tested On

- **Vehicle**: 2021 Hyundai Kona Electric Limited (without SCC)
- **Device**: Comma Two (C2)
- **Control Type**: Torque steering

Should work on other non-SCC Hyundai/Kia/Genesis vehicles.

## Installation

### Via Comma Device Custom URL:
```
https://github.com/rushface/openpilot/tree/lateral-only-kona
```

### Via SSH:
```bash
cd /data
mv openpilot openpilot.bak
git clone -b lateral-only-kona https://github.com/rushface/openpilot.git
reboot
```

## Configuration

**Critical**: Make sure `OpkrVariableCruise` is set to `0` (OFF) in your UI settings:
- Go to **Driving Menu** → **Use Cruise Button Spamming** → Set to **OFF**

With this setting off, the SCC simulation is completely disabled.

## Usage

### Method 1: Lateral-Only (Recommended)
1. Drive normally without any cruise active
2. Tap **SET** button on steering wheel
3. Lateral steering engages immediately
4. Your throttle/brake remain fully manual

### Method 2: Lateral + Stock Cruise
1. Press **CRUISE ON/OFF** button
2. Tap **SET** to set desired cruise speed
3. Lateral steering engages automatically
4. Stock cruise maintains your set speed
5. No speed jumping, no interference

## What Was Changed

This patch wraps the following in `OpkrVariableCruise` checks:

1. **SCC11/SCC12 messages** - Cruise speed and acceleration commands
2. **SCC13/SCC42a messages** - Supplementary SCC status data  
3. **CANCEL button spam** - Virtual cruise cancel button presses
4. **RES_ACCEL button spam** - Virtual resume/accel button presses

When `OpkrVariableCruise=0`, none of these messages are sent, eliminating CAN conflicts.

## Technical Details

- **Panda Safety Mode**: `noOutput` (required for non-SCC vehicles on this fork)
- **Lateral Control**: LKAS11 steering messages (unmodified)
- **Longitudinal Control**: Disabled (`openpilotLongitudinalControl=False`)
- **PCM Cruise**: Enabled (`pcmCruise=True`)

## Files Modified

- `selfdrive/car/hyundai/carcontroller.py` - All SCC message sends and button spamming wrapped in feature flag checks

## Credits

- Base fork: [phr00t/openpilot](https://github.com/phr00t/openpilot)
- Original: [commaai/openpilot](https://github.com/commaai/openpilot)
- Patch development: Collaborative debugging via Claude

## Support

This is a personal patch for my specific vehicle. Use at your own risk. If you have a similar non-SCC Hyundai and want to try it, feel free! Open an issue if you find problems.

## License

Same as base openpilot - MIT License
