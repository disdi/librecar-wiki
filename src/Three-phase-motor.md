# Three-phase motor

LibreCar can drive a three-phase motor directly from the Arty. [Milestone 3](https://saketsinha.de/librecar-demo/2024/03/milestone-three-complete/) already spun a brushless motor, but the Arty only sent CAN commands to an external [ODrive S1](https://odriverobotics.com/). The long-term path is for the FPGA itself to generate the PWM and close the loop, using the same high-level CAN interface.

```
CAN setpoint (speed / torque / position)
        │
        ▼
Linux or softcore ── writes one setpoint register
        │
        ▼
gateware commutation ← hall inputs
        │              ← optional current-sense ADC
        ▼
3 (or 6) PWM channels ── PMOD (control signals only)
        │
        ▼
external gate driver ── 3 half-bridges ── motor
```

## Hardware split

- **PMOD (JA or JD)** carries only low-power control signals: 3–6 PWM, 3 hall inputs, enable, fault, and optional current-sense analog or SPI.
- The **power stage** is a separate board (DRV8323-class, IR2136-style open-source board, [SimpleFOC](https://github.com/simplefoc/Arduino-FOC) inverter, or [VESC](https://github.com/vedderb/bldc-hardware) power stage). The PMOD never carries motor current.
- JC remains the CAN PMOD. JB remains the 10BASE-T1S PMOD.

## Control modes

The carrier is the LiteX PWM core, [`litex/soc/cores/pwm.py`](https://github.com/enjoy-digital/litex/blob/master/litex/soc/cores/pwm.py). `MultiChannelPWM` shares one `period` and gives each pad its own `width`. The pad is high while the counter is below `width`. A LiteX Arty system clock is usually 100 MHz, so a 20 kHz carrier is a period of 5000 cycles.

Linux or the softcore writes one setpoint register. It does not update the widths on every carrier cycle.

The first mode is six-step commutation driven by the halls. Two phases are driven, the third floats, and the hall code advances the sector. `width` sets how hard the two active phases are driven.

The stock PWM core has no dead time and no complementary output. The external gate driver makes the low-side switch and inserts the dead time. A DRV8323-class driver does that from three PWM inputs. Six PWM channels are for a driver that expects a complementary pair per phase.

The later mode is field-oriented control with phase-current sampling synchronized to the PWM carrier, in the style of a [VESC](https://vesc-project.com/) controller. That is the loop the S1 was running. Six-step is enough to turn the shaft from LibreCar.

Reference boards are VESC-controller style designs and SimpleFOC inverters. The published ODrive v3.6 power stage is the same shape. The [ODrive repository](https://github.com/odriverobotics/ODrive) supports the older v3 boards only. The S1 firmware is not public.

## Status

`MultiChannelPWM` is upstream LiteX. The LibreCar bitstream does not instantiate it yet, and this repository has no gate-driver board. [Milestone 3](https://saketsinha.de/librecar-demo/2024/03/milestone-three-complete/) remains the CAN demonstration with the S1.

The hardware pattern (PMOD or header for control signals + external power stage) is already common in the SimpleFOC and open BLDC communities. No custom LibreCar power stage is required to start.

When the bitstream changes:

1. Instantiate `MultiChannelPWM` with three or six channels on JA or JD.
2. Add the hall inputs and the six-step sector block.
3. Add optional current-sense ADC inputs.
4. Expose one setpoint register to Linux or the softcore.
5. Wire the PMOD control signals to an external gate-driver board that inserts dead time.
6. Receive the setpoint over the existing CAN interface.
