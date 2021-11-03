# Инструкция по прошивке платы TF307

## Внимание!!!

1. **Уровень сигналов UART и SPI на TF307 составляет 1,8 вольт. ** 
2. * Не используйте адаптеры (USB-UART, SPI, ETC) с уровнями 3.3 или 5 вольт, поскольку может быть повреждена плата и процессор!**


## Необходимое оборудование

1. [Olimex ARM-USB-OCD-H JTAG](https://www.olimex.com/Products/ARM/JTAG/ARM-USB-OCD-H)
2. [FT232 USB UART модуль](https://www.chipdip.ru/product/ft232-usb-uart-board-type-a)
3. ПК х86 (далее в документе хост-компьютер)


## Необходимый софт (на хост-компьютере)

1. flashrom, версия 1.2 или новее. Предыдущие версии не работают
2. picocom, версия 2.2 (более старые версии также можно использовать)
3. sudo and sudo доступ
4. python3, версия 3.6 или новее, предыдущие версии не работают
5. python3-serial, версия 3.4
6. `lsusb` из пакета usbutils, версия 012 (более старые версии также можно использовать)
7. `udevadm` из пакета udev, версия 246 (более старые версии также можно использовать)


## Подключение jtag и USB UART модуля к плате

**Предупреждение #1**: **Не подключайте стандартный 20-контактный кабель jtag напрямую к разъему `XP8`**!
**Предупреждение #2**: **Убедитесь что уровени сигналов USB UART модуля составляют 1.8В, в противном случае плата и процессор будут повреждены!**

И jtag, и переходник usb-uart должны быть подключены к разъему `XP8`.


| XP8 TF307 pin |       |  ARM-USB-OCD-H pin  |        |
| :-----------: | :---: | :-----------------: | :----: |
|  `BOOT_SS`    |   5   |      `TTMS`         |   7    |
|  `BOOT_CLK`   |   7   |      `TTCK`         |   9    |
|  `BOOT_MISO`  |   9   |      `TTDO`         |   13   |
|  `BOOT_MOSI`  |   11  |      `TTDI`         |   5    |
|  `VREF1V8`    |   18  |      `Vref`         |   1    |
|  `GND`        |   6   |      `GND`          | 4 - 20 |


|     XP8 TF307 pin        |       | FT232 USB UART pin |
| :----------------------: | :---: | :----------------: |
|  `CONN_UART_TX_TO_BMC`   |   17  |       RX           |
|  `CONN_UART_RX_FROM_BMC` |   19  |       TX           |
|   GND                    |   14  |       GND          |


Также возможно подключение UART консоли BE-M1000 
(чтобы проверить, может ли плата загружаться и т. д.).
Примечание: требуется дополнительный адаптер USB-UART 

|     XP8 TF307 pin           |       | FT232 USB UART pin |
| :-------------------------: | :---: | :----------------: |
|  `BM_UART0_TX_TO_CONSOLE`   |   13  |       RX           |
|  `BM_UART0_RX_FROM_CONSOLE` |   15  |       TX           |
|  `GND`                      |   10  |       GND          |


## Прошивка

Исходное состояние: 

* плата физически выключена (шнур питания отключен) 
* Olimex ARM-USB-OCD-H JTAG подключен к хост-компьютеру 
* FT232 USB UART модуль подключен к хост-компьютеру


1. Подключите кабель питания к БП ATX, только дежурное питание
2. Выясните узел устройства, который соответствует ft232 usb uart (тот, который подключен к/от контактам bmc).
   Например, если один ft232 usb uart подключен к хост-компьютеру.
   ```
   ls -l /dev/serial/by-id/ | grep FTDI
   lrwxrwxrwx 1 root root 13 Jun 29 13:19 usb-FTDI_FT232R_USB_UART_A50285BI-if00-port0 -> ../../ttyUSBNNN
   ```
   Узел устройства `/dev/ttyUSBNNN`
3. Подключитесь к BMC консоли командой:
   ```
   picocom -b115200 /dev/ttyUSBNNN
   ```
4. Для плат TP-TF307-MB-A0, TF307-MB-S-C: Выполните следующие команды в BMC консоли:
   1. `pins set 7`
   При успешном выполнении команды BMC консоль выдаст `Set pin[7] BM_SPI_SEL`
   2. `pins set 19`
   При успешном выполнении команды BMC консоль выдаст `Set pin[19] ATX_PSON`
   3. pins set 23
   При успешном выполнении команды BMC консоль выдаст `Set pin[23] EN_1V8`
5. Для платы TF307-MB-S-D команды в BMC консоли будут следующие:
   1. `pins set 11`
   При успешном выполнении команды BMC консоль выдаст `Set pin 11 (BM_SPI_SEL)`
   2. `pins set 16`
   При успешном выполнении команды BMC консоль выдаст `Set pin 16 (ATX_PSON)`
   3. `pins set 26`
   При успешном выполнении команды BMC консоль выдаст `Set pin 26 (EN_1V8)`
6. Сделайте резервную копию прошивки, командой:
   ```
   sudo flashrom -p ft2232_spi:type=arm-usb-ocd-h,port=A,divisor=8 -c MT25QU256 -r tf307-firmware.bak.bin
   ```
7. Прошивка новой версии прошивки:
   ```
   sudo flashrom -p ft2232_spi:type=arm-usb-ocd-h,port=A,divisor=8 -c MT25QU256 -w mbm20.full.img
   ```
   `flashrom` предупредит, что чип не тестировался:
   ```
    flashrom v1.2 on Linux 5.8.0-53-generic (x86_64)
    flashrom is free software, get the source code at https://flashrom.org

    Using clock_gettime for delay loops (clk_id: 1, resolution: 1ns).
    Found Micron flash chip "MT25QU256" (32768 kB, SPI) on ft2232_spi.
    ===
    This flash part has status UNTESTED for operations: PROBE READ ERASE WRITE
    The test status of this chip may have been updated in the latest development
    version of flashrom. If you are running the latest development version,
    please email a report to flashrom@flashrom.org if any of the above operations
    work correctly for you with this flash chip. Please include the flashrom log
    file for all operations you tested (see the man page for details), and mention
    which mainboard or programmer you tested in the subject line.
    Thanks for your help!
   ```
   Не стоит обращать внимание на это предупреждение.
   Вывод в консоли будет следующим:
   ```
   Reading old flash chip contents...
   ```
   (это занимает около 30 секунд)
   далее
   ```
   Erasing and writing flash chip...
   Erase/write done
   Verifying flash...
   VERIFIED
   ```
8. Введите в BMC консоли следующую команду
   1. `pins bootseq`
   При успешном выполниении команды BMC консоль выдаст
   ```
   L: [SHELL] Starting boot sequence
   E: [MB1BM1_PINS] PWG is active when 1.8 V voltage regulator is disabled
   L: [SHELL] Boot sequence finished
   ```
   2. `pins board_off`
   При успешном выполнении команды BMC консоль выдаст
   ```
   L: [SHELL] Pins are reset to board off state
   L: [MB1BM1_PINS] Wake up requested
   L: [RTC] Current date 12.03.21, time 07:32:09
   ```
   3. `pins board_off` (Снова повторите ту же самую команду)
   При успешном выполнении команды BMC консоль выдаст
   ```
   L: [SHELL] Pins are reset to board off state
   ```
