# Hardware PMODs

LibreCar uses two 12-pin PMODs on the Arty A7-100T. The CAN module hangs off **JC**. The 10BASE-T1S module hangs off **JB**. The onboard RJ45 at **J9** stays the lab Ethernet port used for TFTP in [Linux Bringup](Linux-bringup.md).

Both modules are 3.3 V. The FPGA supplies power on the PMOD 3.3 V pins. The host supplies the protocol controller: CTUCAN for CAN, LiteEth RMII plus MDIO for 10BASE-T1S.

## Connector map

```
                  Arty A7-100T
  JC ── CAN PMOD ── twisted pair ── CAN device
  JB ── Step 0 PMOD ── single pair ── 10BASE-T1S segment
  J9 ── onboard PHY ── RJ45 ── lab LAN / TFTP
  J10 ─ USB ── console and programming
```

JB is a high-speed PMOD (no 200 Ω series resistors). That is why the 50 MHz RMII clock uses JB. JA and JD have series resistors and are a poor fit for that clock.

The VexRiscv SWD debug path also uses JB. Keep 10BASE-T1S and SWD off the same header in one sitting.

## CAN transceiver PMOD (JC)

Project: [disdi/canpmod](https://github.com/disdi/canpmod). Active sources are KiCad 9.0.8. License is MIT, copyright 2024 disdi. The module is an SN65HVD230 plus a 2-pin CAN terminal and a 120 Ω termination jumper. The FPGA implements the CAN controller.

![Current CAN PMOD front copper and silkscreen](canpmod-front.svg)

Viewed from the PCB top with the plug pointing left, pins 1–6 run top to bottom in the inboard row and pins 7–12 run top to bottom in the outboard row. Pin 1 is square and marked on silkscreen.

| PMOD pin | Signal | Direction relative to host |
| --- | --- | --- |
| 1, 2, 3 | NC | Unconnected |
| 4 | CAN RX | Input to host |
| 5 | GND | Ground |
| 6 | 3.3 V | Supply from host |
| 7, 8, 9 | NC | Unconnected |
| 10 | CAN TX | Output from host |
| 11 | GND | Ground |
| 12 | 3.3 V | Supply from host |

J1 on the module is the 5.08 mm CAN terminal: pin 1 CAN_L, pin 2 CAN_H. Fit JP1 for an end node. Leave JP1 open for an intermediate node. The removable shunt is a separate accessory.

R1 is 4.7 kΩ slope control. R2 pulls TX high. C1 is 100 nF supply bypass. R3/C2 and R4/C3 are 100 Ω / 560 pF RC branches from each bus line to ground. There is no CAN controller, isolation, extra TVS device or bus reference-ground terminal on the module.

The 3D render on the CAN page is the original drawing. The front plot above is the current corrected board.

## 10BASE-T1S PMOD (JB)

Project: [disdi/10BaseT1S-PMOD](https://github.com/disdi/10BaseT1S-PMOD). Active sources are KiCad 9.0.8, revision 0.2. License is MIT, copyright 2024 disdi, the same text as canpmod. The PHY is a Microchip LAN8670 in RMII mode. The cable side is a 2-pin screw terminal on a 10BASE-T1S mixing segment. Roadmap notes remain in [Open Single Pair Ethernet](https://saketsinha.de/open-single-pair-ethernet/#step-0-).

![Step 0 silkscreen](step0-silk.svg)

Revision 0.2 is the Arty JB direct-plug order. CLK50 is PMOD pin 1 so it reaches Arty E15 (SRCC). TXD0 is pin 10 (J15). Earlier Katze/LiteX pinouts differ; this revision needs its own LiteX platform constraints.

| Module J1 / Arty JB | Signal | FPGA package pin | Direction at FPGA |
| --- | --- | --- | --- |
| 1 | CLK50 | E15, SRCC | Input clock |
| 2 | TXD1 | E16 | Output |
| 3 | TXEN | D15 | Output |
| 4 | CRS_DV | C15 | Input |
| 7 | RXD0 | J17 | Input |
| 8 | RXD1 | J18 | Input |
| 9 | RST_N | K15 | Output |
| 10 | TXD0 | J15 | Output |
| 5, 11 | GND | Ground | — |
| 6, 12 | +3V3 | 3.3 V | Supply to module |

Package pins are from the [Digilent Arty A7-100 Rev D/E master XDC](https://github.com/Digilent/digilent-xdc/blob/master/Arty-A7-100-Master.xdc).

The FPGA receives CLK50 from the module. Keep TXEN inactive during configuration and reset. Release PHY reset after 3.3 V and the 50 MHz oscillator are stable.

MDIO is a separate 3-pin header J3 on the module: pin 1 MDC, pin 2 MDIO, pin 3 GND. Those nets are off J1. Wire MDC and MDIO to two spare 3.3 V FPGA GPIOs and a common ground. IRQ_N stays on the module.

PHY address straps pull down, so the MDIO address is 0. PLCA node ID is a register, written over MDIO after reset.

J2 is the cable connector: pin 1 TRX_P, pin 2 TRX_N. Fit both JP1 and JP2 for an end-node 100 Ω termination. Leave both open for a drop node.

## Mating and status

Plug the CAN module into JC and the 10BASE-T1S module into JB with the PMOD keys aligned and both grounds connected. Check connector type, orientation and mechanical clearance before assembly.

The CAN chapter is a working SocketCAN path once the transceiver is fitted. The 10BASE-T1S chapter is the planned LiteEth path; there is no proven Arty bitstream for that PHY in this wiki yet.
