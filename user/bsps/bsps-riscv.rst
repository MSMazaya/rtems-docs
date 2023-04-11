.. SPDX-License-Identifier: CC-BY-SA-4.0

.. Copyright (C) 2018 embedded brains GmbH

riscv (RISC-V)
**************

riscv
=====

This BSP offers 12 variants:

* rv32i

* rv32iac

* rv32im

* rv32imac

* rv32imafc

* rv32imafd

* rv32imafdc

* rv64imac

* rv64imafd

* rv64imafdc

* frdme310arty

* mpfs64imafdc

Each rv* variant corresponds to a GCC multilib.  A particular variant reflects an
ISA with ABI and code model choice. All rv64 BSPs have medany code model by
default, while rv32 BSPs are medlow. The reason is that RV32 medlow can access
the entire 32-bit address space, while RV64 medlow can only access addresses
below 0x80000000. With RV64 medany, it's possible to perform accesses above
0x80000000.

The BSP must be started im machine mode.

The reference platform for this BSP is the QEMU `virt` machine.

The reference platform for the mpfs64imafdc BSP variant is the Microchip
PolarFire SoC Icicle Kit.

Build Configuration Options
---------------------------

The following options can be used in the BSP section of the ``waf``
configuration INI file. The ``waf`` defaults can be used to inspect the values.

``BSP_PRESS_KEY_FOR_RESET``
    If defined to a non-zero value, then print a message and wait until pressed
    before resetting board when application terminates.

``BSP_RESET_BOARD_AT_EXIT``
    If defined to a non-zero value, then reset the board when the application
    terminates.

``BSP_PRINT_EXCEPTION_CONTEXT``
    If defined to a non-zero value, then print the exception context when an
    unexpected exception occurs.

``BSP_FDT_BLOB_SIZE_MAX``
    The maximum size of the device tree blob in bytes (default is 65536).

``BSP_DTB_IS_SUPPORTED``
    If defined to a non-zero value, then the device tree blob is embedded in
    the BSP.

``BSP_DTB_HEADER_PATH``
    The path to the header file containing the device tree blob.

``BSP_CONSOLE_BAUD``
    The default baud for console driver devices (default 115200).

``RISCV_MAXIMUM_EXTERNAL_INTERRUPTS``
     The maximum number of external interrupts supported by the BSP (default
     64).

``RISCV_ENABLE_HTIF_SUPPORT``
     Enable the Host/Target Interface (HTIF) support (enabled by default).

``RISCV_CONSOLE_MAX_NS16550_DEVICES``
     The maximum number of NS16550 devices supported by the console driver (2
     by default).

``RISCV_RAM_REGION_BEGIN``
     The begin of the RAM region for linker command file (default is 0x80000000).

``RISCV_RAM_REGION_SIZE``
     The size of the RAM region for linker command file (default 64MiB).

``RISCV_ENABLE_FRDME310ARTY_SUPPORT``
     Enables support sifive Freedom E310 Arty board if defined to a non-zero
     value,otherwise it is disabled (disabled by default)

``RISCV_ENABLE_MPFS_SUPPORT``
     Enables support Microchip PolarFire SoC if defined to a non-zero
     value, otherwise it is disabled (disabled by default).

``RISCV_BOOT_HARTID``
     The boot hartid (processor number) of risc-v cpu by default 0.

Interrupt Controller
--------------------

Exactly one Core Local Interruptor (CLINT) and exactly one Platform-Level
Interrupt Controller (PLIC) are supported.  The maximum number of external
interrupts supported by the BSP is defined by the
``RISCV_MAXIMUM_EXTERNAL_INTERRUPTS`` BSP option.

Clock Driver
------------

The clock driver uses the CLINT timer.

Console Driver
--------------

The console driver supports devices compatible to

* "ucb,htif0" (depending on the ``RISCV_ENABLE_HTIF_SUPPORT`` BSP option),

* "ns16550a" (see ``RISCV_CONSOLE_MAX_NS16550_DEVICES`` BSP option), and

* "ns16750" (see ``RISCV_CONSOLE_MAX_NS16550_DEVICES`` BSP option).

* "sifive,uart0" (see ``RISCV_ENABLE_FRDME310ARTY_SUPPORT`` BSP option).

They are initialized according to the device tree.  The console driver does not
configure the pins or peripheral clocks.  The console device is selected
according to the device tree "/chosen/stdout-path" property value.

QEMU
----

All of the BSP variants that start with rv can be run on QEMU's virt
and spike machines. For instance, to run the ``rv64imafdc`` BSP with the
following "config.ini" file.

.. code-block:: none
    [riscv/rv64imafdc]

Run the following QEMU command.

.. code-block:: shell
    $ qemu-system-riscv64 -M virt -nographic -bios $RTEMS_EXE
    $ qemu-system-riscv64 -M spike -nographic -bios $RTEMS_EXE

Spike
----
All of the BSP variants that start with rv can be run on Spike.
For instance, to run the ``rv64imafdc`` BSP with the following
"config.ini" file.

.. code-block:: none
    [riscv/rv64imafdc]

Run the following Spike command.

.. code-block:: shell
    $ spike --isa=rv64imafdc $RTEMS_EXE

Unlike QEMU, Spike supports enabling/disabling a subset of the imafdc extensions
and has support for further RISC-V extensions as well. A fault will be triggered
if an executable built with rv64imafdc RISC-V's -march option run on Spike with
--isa=rv64i option. If no --isa option is specified, the default is rv64imafdc.

Microchip PolarFire SoC
-----------------------

The PolarFire SoC is the 4x 64-bit RISC-V U54 cores and a 64-bit RISC-V
E51 monitor core SoC from the Microchip.

The ``mpfs64imafdc`` BSP variant supports the U54 cores but not the E51 because
the E51 monitor core is reserved for the first stage bootloader
(Hart Software Services). In order to boot from the first U54 core,
``RISCV_BOOT_HARTID`` is set to 1 by default.

The device tree blob is embedded in the ``mpfs64imafdc`` BSP variant by default
with the ``BSP_DTB_IS_SUPPORTED`` enabled and the DTB header path
``BSP_DTB_HEADER_PATH`` is set to bsp/mpfs-dtb.h.

**SMP test procedure for the Microchip PolarFire Icicle Kit:**

The "config.ini" file.

.. code-block:: none

    [riscv/mpfs64imafdc]
    BUILD_TESTS = True
    RTEMS_POSIX_API=True
    RTEMS_SMP = True
    BSP_START_COPY_FDT_FROM_U_BOOT=False
    BSP_VERBOSE_FATAL_EXTENSION = False

Build RTEMS.

.. code-block:: shell

    $ ./waf configure --prefix=$HOME/rtems-start/rtems/6
    $ ./waf

Convert .exe to .elf file.

.. code-block:: shell

    $ riscv-rtems6-objcopy build/riscv/mpfs64imafdc/testsuites/smptests/smp01.exe build/riscv/mpfs64imafdc/testsuites/smptests/smp01.elf

Generate a payload for the `smp01.elf` using the `hss-payload-generator <https://github.com/polarfire-soc/hart-software-services/blob/master/tools/hss-payload-generator>`_.

* Copy `smp01.elf` file to the HSS/tools/hss-payload-generator/test directory.

* Go to hss-payload-generator source directory.

.. code-block:: shell

    $ cd hart-software-services/tools/hss-payload-generator

* Edit test/uboot.yaml file for the hart entry points and correct name of the
  binary file.

.. code-block:: none

    set-name: 'PolarFire-SoC-HSS::RTEMS'
    hart-entry-points: {u54_1: '0x1000000000', u54_2: '0x1000000000', u54_3: '0x1000000000', u54_4: '0x1000000000'}
    payloads:
     test/smp01.elf: {exec-addr: '0x1000000000', owner-hart: u54_1, secondary-hart: u54_2, secondary-hart: u54_3, secondary-hart: u54_4, priv-mode: prv_m, skip-opensbi: true}

* Generate payload

.. code-block:: shell

    $ ./hss-payload-generator -c test/uboot.yaml payload.bin

Once the payload binary is generated, it should be copied to the eMMC/SD.

`FPGA design with HSS programming file <https://github.com/polarfire-soc/polarfire-soc-documentation/blob/master/boards/mpfs-icicle-kit-es/updating-icicle-kit/updating-icicle-kit-design-and-linux.md>`_.

Program the eMMC/SD with the payload binary.

* Power Cycle the Microchip PolarFire Icicle Kit and stop at the HSS.

* type "mmc" and then "usbdmsc" on the HSS terminal(UART0).

* Load the payload.bin from the Host PC.

.. code-block:: shell

    $ sudo dd if=payload.bin of=/dev/sdb bs=512

Reset the Microchip PolarFire SoC Icicle Kit.

Serial terminal UART1 displays the SMP example messages

.. code-block:: none

    *** BEGIN OF TEST SMP 1 ***
    *** TEST VERSION: 6.0.0.ef33f861e16de9bf4190a36e4d18062c7300986c
    *** TEST STATE: EXPECTED_PASS
    *** TEST BUILD: RTEMS_POSIX_API RTEMS_SMP
    *** TEST TOOLS: 12.1.1 20220622 (RTEMS 6, RSB 3cb78b0b815ba05d17f5c6
		5865d246a8333aa087, Newlib ea99f21)

    CPU 3 start task TA0
    CPU 2 running Task TA0
    CPU 3 start task TA1
    CPU 1 running Task TA1
    CPU 3 start task TA2
    CPU 0 running Task TA2

    *** END OF TEST SMP 1 ***

noel
====

This BSP supports the `NOEL-V <https://gaisler.com/noel-v>`_ systems from Cobham
Gaisler. The NOEL-V is a synthesizable VHDL model of a processor that
implements the RISC-V architecture. It is part of the open source
`GRLIB <https://gaisler.com/grlib>`_ IP Library. The following BSP
variants correspond to common NOEL-V configurations:

* noel32im

* noel32imafd

* noel64imac

* noel64imafd

* noel64imafdc

The start of the memory is set to 0x0 to match a standard NOEL-V system,
but can be changed using the ``RISCV_RAM_REGION_BEGIN`` configuration
option. The size of the memory is taken from the information available
in the device tree.

Reference Designs
-----------------

The BSP has been tested with NOEL-V reference designs for
`Digilent Arty A7 <https://gaisler.com/noel-artya7>`_,
`Microchip PolarFire Splash Kit <https://gaisler.com/noel-pf>`_,
and `Xilinx KCU105 <https://gaisler.com/noel-xcku>`_.
See the accompanying quickstart guide for each reference design
to determine which BSP configuration to use.

Build Configuration Options
---------------------------

The following options can be used in the BSP section of the ``waf``
configuration INI file. The ``waf`` defaults can be used to inspect the values.

``BSP_CONSOLE_USE_INTERRUPTS``
     Use the Termios interrupt mode in the console driver (true by default).

``BSP_FDT_BLOB_SIZE_MAX``
    The maximum size of the device tree blob in bytes (262144 by default).

``RISCV_CONSOLE_MAX_APBUART_DEVICES``
     The maximum number of APBUART devices supported by the console driver
     (2 by default).

``RISCV_RAM_REGION_BEGIN``
     The begin of the RAM region for linker command file (0x0 by default).

``RISCV_MAXIMUM_EXTERNAL_INTERRUPTS``
     The maximum number of external interrupts supported by the BSP (64 by
     default).

griscv
======

This RISC-V BSP supports chips using the
`GRLIB <https://www.gaisler.com/products/grlib/grlib.pdf>`_.
