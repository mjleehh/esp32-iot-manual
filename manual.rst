**********************************
Internet of Things Workshop Manual
**********************************

.. class:: center

| Michael Jonathan Lee
| June 2019


Setting up the virtual machine
##############################

Download the virtual machine image from 

`<https://drive.google.com/file/d/1Nnj282Mw2e3J-T9aMRZox5r36YlDWPKb/view?usp=sharing>`_

The code on the machine was written for the 
`Heltec WiFi Kit 32<https://heltec.org/project/wifi-kit-32/>`

.. image:: img/helltec-wifi-kit.jpg
    :align: center
    :width: 200

For communication between the virtual machine and the device ensure the following two
settings:
* the UART controller connected to the virtual machine

..image:: img/virtualbox-uart.png
    :width: 300

* give the virtual machine its own IP on the network through a bridged network adapter
        
..image:: img/virtualbox-bridged-network-adapter.png
    :width: 300