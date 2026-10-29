# Zonal gateway

The [Introduction](title-page.md) draws a zone controller: CAN toward nearby sensors and actuators, Automotive Ethernet as the 10 Mb/s backbone, and a CPU that bridges the two. On the Arty A7-100T that is three ports on one LiteX Linux SoC.

```
sensors / actuators ── CAN pair ── CAN PMOD @ JC ── CTUCAN ──┐
                                                              ├─ VexRiscv Linux
zonal backbone ──── 10BASE-T1S pair ── 10BASE-T1S PMOD @ JB ── LiteEth ┘
lab / TFTP ──────── RJ45 ────── onboard PHY @ J9
```

CAN bring-up is [CAN communication](Communication-with-CAN-devices.md). The T1S PHY and LiteEth plan are [10BASE-T1S bringup](10BASE-T1S-bringup.md). Pin maps are [Hardware PMODs](Hardware-PMODs.md).

## Lab gateway already on J9

[Milestone 4](https://librecar.dev/2024/04/milestone-four-complete/) brought up the Arty onboard Ethernet in LiteX Linux and ran [cannelloni](https://github.com/mguentner/cannelloni). Cannelloni carries CAN frames over UDP/TCP/SCTP between a SocketCAN interface and a remote IP host.

That path is:

```
can0 (CTUCAN + CAN PMOD) ── cannelloni ── eth0 (J9) ── IP network ── remote CAN
```

It is a CAN–Ethernet gateway on the lab LAN. It is the right tool for PC tooling, logging and a Raspberry Pi on the bench. J9 is office Ethernet. The zonal diagram’s backbone is 10BASE-T1S.

[Milestone 5](https://librecar.dev/2024/04/milestone-five/) and the [XCP update](https://librecar.dev/2024/04/milestone-five-update/) add UDS and XCP on that same Linux user-space. Those protocols can ride CAN or IP once the sockets exist.

## Automotive gateway on JB

Replace the IP hop with the mixing segment when the LAN8670 PMOD is up:

```
can0 (JC) ── user-space gateway ── eth1 (JB, 10BASE-T1S)
```

Same SocketCAN and Ethernet sockets. The cable becomes a single pair with PLCA instead of an RJ45 switch. Fit CAN JP1 and both 10BASE-T1S PMOD jumpers on end nodes. Leave them open on drop nodes.

Useful first checks, once both interfaces exist:

1. `cansend` / `candump` on `can0` as in the CAN chapter.
2. Ping or a raw Ethernet test toward another 10BASE-T1S node on `eth1`.
3. cannelloni or a small user-space bridge between `can0` and `eth1`.
4. XCP/UDS against an ECU on CAN, with the calibration tool on the T1S side or the other way around.

## What is still open

The lab gateway on J9 is documented on librecar.dev. The zonal T1S gateway needs the [10BASE-T1S PMOD](https://github.com/disdi/10BaseT1S-PMOD), LiteEth RMII/PLCA on JB, and a second Linux netdev. Those pieces are designed and not yet a proven link. Keep J9 connected while bringing T1S up so TFTP and SSH stay on the lab PHY.
