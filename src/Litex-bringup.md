# Litex Bringup

**Steps to bringup CAN bus on Litex!**

* Checkout  [Litex Repo](https://github.com/disdi/litex) .

* Install Litex dependencies

```
./litex_setup.py --dev --init
```
* Build/Install litex with CAN support
```
 ./litex_setup.py --init --install --user 
```
* Current CAN Litex support is added to [Arty board](https://digilent.com/shop/arty-a7-100t-artix-7-fpga-development-board/).
Checkout  [Litex Board Repo](https://github.com/disdi/litex-boards) .
 
 * Build bitstream for Arty board.
 ```
 python3 litex_boards/targets/digilent_arty.py --variant=a7-100 --build

 ```

 * Connet Arty board over USB and load the bitstream
 ```
 python3 litex_boards/targets/digilent_arty.py --variant=a7-100 --load

 ```

 * Open ttyUSB port to read serial logs.
 Litex should bootup and *mem_list* should show CAN peripheral.
