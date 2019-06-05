**********************************
Internet of Things Workshop Manual
**********************************

.. class:: center

| Michael Jonathan Lee
| June 2019


Contents
########

.. contents::

Setting up the virtual machine
##############################

Download the virtual machine image from 

`<link>`_

The code on the machine was written for the 
`Heltec WiFi Kit 32 <https://heltec.org/project/wifi-kit-32/>`_

.. image:: img/helltec-wifi-kit.jpg
    :align: center
    :width: 150

For communication between the virtual machine and the device ensure the following two
settings:

* the UART controller connected to the virtual machine

.. image:: img/virtualbox-uart.png
    :align: center
    :width: 200

* give the virtual machine its own IP on the network through a bridged network adapter
        
.. image:: img/virtualbox-bridged-network-adapter.png
    :align: center
    :width: 200

Running the Example
###################

What it does
------------

The central idea of the example is to have smart screens that display a message which we can
remotely be altered through an app, without physical access to the devices.

.. image:: img/things.jpg
    :align: center
    :width: 300

What we technically demonstrate is typical IoT 3 actor communication:

* up to three devices (``smartscreen-1, smartscreen-2, smartscreen-3``)
* a central backend
* an app for the user to control the devices

.. image:: img/communication-triangle.png
    :align: center
    :width: 150

The following communication flow occurs:

* the backend has a list of known devices
* the backend tracks the last known IP address of each device
* when a device boots up, after connecting to the WiFi it will notify the backend of its status
* when the app loads it receives the device list from the backend
* if the app wishes to change the message on one of the screens it sends a request to the backend
* requests to change the message will be forwarded by the backend to the specific device via the
last known IP of that device

The educational focus of the example is not a set of best practices or libraries, but to
give an idea of where to get started with IoT.

Step 1: Startting the backend
-----------------------------

Go to the ``~/code/backend`` directory:

.. code:: bash

    cd ~/code/backend

Build the backend

.. code:: bash

    $ go build

Start the backend

.. code:: bash

    $ ./iot-backend

This will start a backend serving the app specific endpoints on port ``:3000`` and
the device specific endpoints on port ``:3001``

Step 2: Building and Flashing the Firmware
------------------------------------------

.. note:: The full name of the ESP-IDF framworks central tool command is ``idf.py``. On the virtual machine the
    alias ``idf`` can be used.

Go to the ``~/code/firmware`` directory:

.. code:: bash

    $ cd ~/code/firmware

Start the menuconfig config editor:

.. code:: bash

    $ idf menuconfig

Make sure the entry

.. code::

    Component config -> ESP32-specific -> Main XTAL frequency

is set to 26MHz:

.. image:: img/idf-menuconfig.png

.. image:: img/menuconfig-esp32-specific.png

.. image:: img/menuconfig-main-xtal.png

You will also have to set up the WiFi and backend address config in

.. cdoe::

    Component config -> smartscreen

* set the SSID and password of the WiFi network you want to use
* determine the virtual machine's IP address and set it as the ``home address``
* set which of the 3 available device IDs the device should have


.. image:: img/menuconfig-smartscreen.png

You can determine the IP address using ``ifconfig``:

.. code:: bash

    $ ifconfig

.. image:: img/determine-ip.png

Compile the code:

.. code:: bash

    $ idf build

Flash the device with your firmware:

.. code:: bash

    $ idf flash

To view the log output:

.. code:: bash

    $ idf monitor

Step 3: Bundling and Building the App
-------------------------------------

Go to the ``~/code/app`` directory:

.. code:: bash

    $ cd ~/code/app

Install all dependencies:

.. code:: bash

    $ npm i

Start the app dev server:

.. code:: bash

    $ npm start

Now open a browser and launch the app by opening

.. code::

    http://localhost:8080

.. image:: img/app-and-backend.png

Debugging the ESP32
###################

Applications on the ESP32 can become very complex:

* the network stack requires multitasking
* the dual core setup can cause some complexity
* most likely SD or flash storage will occur
* frequent dynamic memory alloocation/deallocation is required in many of the ESP32's use cases

Error tracing through debug logs can become close to impossible if your firmware reaches
a certain complexity. Especially modern C++ tequniques can become very hard to debug
using logging. Also one could argue that logging is a terrible error tracing technique
anyway.

So we need a way of actually debugging the firmware on-chip. Fortunately the ESP32
supports
`JTAG debugging <https://blog.senr.io/blog/jtag-explained>`_
The
`setup process <https://docs.espressif.com/projects/esp-idf/en/latest/api-guides/jtag-debugging/>`_
is explained in detail in the ESP32 docs.
The biggest problem with JTAG debugging is, that your board layout has to support it
and that certain pins on the SoC become occupied by the debugger.
The Helltec WiFi Kit 32 does not have JTAG support out of the box. If you wish to debug
this specific board you would need to create a custom board with JTAG debugging support
and the same components the WiFi Kit 32 uses.

We will demonstrate the process of JTAG debugging using the
`ESP Wrover Kit 4.1 <https://docs.espressif.com/projects/esp-idf/en/latest/get-started/get-started-wrover-kit.html>`_
manufactured by Espressif.

.. image:: img/wrover-4.jpg

.. note:: All required tools are pre-installed on the virtual machine. No further setup is required.

NOTE: Unlike the WiFi Kit23, the Wrover Kit has a main crystal frequency of 40MHz so re-check your setting:

.. code::

    Component config -> ESP32-specific -> Main XTAL frequency

First of all make sure the highlighted jumpers on the back of the board are set for JTAG debugging:

.. image:: img/wrover-4-back.jpg

The actual debugging is done via
`OpenOCD <http://openocd.org/>`_.
Will need to install the
`ESP32 specific version of OpenOCD <https://github.com/espressif/openocd-esp32/releases>`_.
Make sure you set the
``OPENOCD_SCRIPTS`` environmant variable to

.. code::

    <open ocd installation dir>/share/openocd/scripts

On linux systems configure the group of the

.. code::

    /dev/ttyUSB*

device files to bbe ``plugdev``.

If you want to use the virtual machine, connect to board:

.. image:: img/connect-wrover-to-esp.png

To start the debug server run:

.. code:: bash
    openocd -f interface/ftdi/esp32_devkitj_v1.cfg -f board/esp-wroom-32.cfg

.. note:: On the virtual machine there is a pre-defined alias ``start-jtag`` for the command above.

OpenOCD will start a debug server that can be accessed be the
`GDB debugger frontend <https://www.gnu.org/software/gdb/>`_
debugger frontend

Go to the simple debugging sample project:

.. code:: bash

    cd ~/code/debug-sample


Build the code and flash the device:

.. code:: bash

    idf build && idf flash


and start the monitoring mode:

.. code:: bash

    idf monitor


Note that you will see some activity in the debugger window:

.. image:: img/openocd-running.png

At this point you could simply run

.. code:: bash

    xtensa-esp32-elf-gdb build/debug-sample.elf


Let's be honest, running GDB in pure command line mode does not sound like much fun.
Thus the sample project comes with a debugging config for Visual Studio Code:

.. code:: json

    {
        "version": "0.2.0",
        "configurations": [{
            "name": "remote Wrover Kit debugging",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/debug-sample.elf",
            "miDebuggerServerAddress": "localhost:3333",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": true,
            "miDebuggerPath": "xtensa-esp32-elf-gdb",
            "MIMode": "gdb",
            "setupCommands": [{
                "description": "Enable pretty-printing for gdb",
                "text": "-enable-pretty-printing",
                "ignoreFailures": true
            }]
        }]
    }

Inside any ESP32 project, place this snippet in a

.. code::

     <project root>/.vscode/launch.json

file. Just make sure you replace the ``build/debug-sample.elf`` with your
executable name: ``build/<project name>.elf``

Now you can start the debugger from within Visual Studio Code and debug with all the
conveience of common debugger UIs:

.. image:: img/debugger-ui.png
