# Three-phase motor

LibreCar drives a three-phase motor from the Arty. Milestone 3 already spun a brushless motor, but the Arty only sent CAN commands to an [ODrive S1](https://odriverobotics.com/). The S1 ran the commutation. That firmware is not public. The [ODrive repository](https://github.com/odriverobotics/ODrive) supports the older v3 boards only.

The carrier is the LiteX PWM core, [`litex/soc/cores/pwm.py`](https://github.com/enjoy-digital/litex/blob/master/litex/soc/cores/pwm.py). `MultiChannelPWM` shares one `period` and gives each pad its own `width`. The pad is high while the counter is below `width`. For a motor, three channels are the three phase duties. A LiteX Arty system clock is usually 100 MHz, so a 20 kHz carrier is a period of 5000 cycles.

```
Linux setpoint (speed or torque)
        │
        ▼
gateware commutation  ← 3 hall inputs
        │
        ▼
3 PWM channels ── PMOD ── gate driver ── 3 half-bridges ── motor
```

Linux writes the setpoint. It does not update the three widths on every carrier cycle. A gateware block reads the hall sensors and chooses the sector. The first mode that turns the shaft is six-step commutation: two phases are driven, the third floats, and the hall code advances the sector. `width` sets how hard the two active phases are driven.

The stock PWM core has no dead time and no complementary output. The gate driver has to make the low-side switch and insert the dead time from each phase PWM. A DRV8323-class driver does that from three inputs. The published ODrive v3.6 power stage and an open [SimpleFOC](https://github.com/simplefoc/Arduino-FOC) inverter are the same shape of board. The PMOD pin connects to that driver, not to a motor winding.

JA or JD can carry the three PWM outputs and the three hall inputs. JC stays the CAN PMOD. JB stays the 10BASE-T1S PMOD.

Field-oriented control is the step after the motor turns. It adds phase-current measurements, sampled in step with the carrier, and it updates the three widths from that current. That is the loop the S1 was running. Six-step is enough to drive the motor from LibreCar.

## Status

`MultiChannelPWM` is upstream LiteX. The LibreCar bitstream does not instantiate it yet, and this repository has no gate-driver board. Milestone 3 remains the CAN demonstration with the S1.

When the bitstream changes:

1. Instantiate `MultiChannelPWM` with three channels on JA or JD.
2. Add the hall inputs and the six-step sector block beside the PWM core.
3. Give Linux one setpoint register.
4. Connect those three pads to a gate driver that inserts dead time, then to the three half-bridges.
