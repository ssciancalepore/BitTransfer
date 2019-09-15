## BitTransfer: Mitigating Reactive Jamming in Electronic Warfare Scenarios
<p align="center">
     <img alt="License" src="https://img.shields.io/static/v1.svg?label=license&message=BSD3&color=brightgreen">
     <img alt="Release" src="https://img.shields.io/static/v1.svg?label=release&message=1.0&color=blue">
     <img alt="License" src="https://img.shields.io/static/v1.svg?label=build&message=passing&color=brightgreen">
     <img alt="Organization" src="https://img.shields.io/static/v1.svg?label=org&message=CRI-LAB&color=blue">
</p>

BitTransfer is a lightweight, powerful, and efficient anti-jamming protocol, particularly suitable to defeat reactive jamming in Electronic Warfare scenarios. Thanks to its simplicity and flexibility, BitTransfer can be implemented efficiently in any Commercial Off-The-Shelf (COTS) device, to enable protection against reactive jamming.

This proof-of-concept has been developed within the well-known <a href="https://github.com/openwsn-berkeley/">OpenWSN</a> protocol stack, integrating the IEEE 802.15.4-2015 standard PHY/MAC layer specification, and it has been evaluated using the <a href="http://www.openmote.com/">OpenMote-b</a> hardware platform.

## Configuration of the BitTransfer Protocol

The BitTransfer protocol can be configured by tuning some parameters available in the firmware of the OpenWSN protocol stack.

Specifically, all the files to be edited are located in the folder `./openwsn-fw-antiJamming/openstack/02a-MAClow/`.

In the file `IEEE802154E.h`, you can configure the byte-size of the message to be exchanged by the involved devices, by defining the `MSG_LENGTH` variable.
```c
//antiJamming protocol parameters definitions

#define MSG_LENGTH 640
```

Then, in the file `IEEE802154E.c`, it is possible to define the starting time of the protocol, and the message to be exchanged between the two devices. For instance, the default configuration message exchanged in the proof-of-concept is the following:
```c
//antiJamming protocol configuration

ieee154e_vars.synch_starting_slot = 1250;

ieee154e_vars.local_bit_counter_tx = 0;
ieee154e_vars.local_bit_counter_rx = 0;

memset(&ieee154e_vars.message_Tx[0], 0, MSG_LENGTH);
memset(&ieee154e_vars.message_Rx[0], 0, MSG_LENGTH);
ieee154e_vars.message_Tx[0] = 1;
ieee154e_vars.message_Tx[MSG_LENGTH/2] = 1;
ieee154e_vars.message_Tx[MSG_LENGTH/2+200] = 1;
ieee154e_vars.message_Tx[MSG_LENGTH-1] = 1;
```

It is also possible to define the paramters for the configuration of the synchronization protocol, based on the nodes in the network, as in the following:
```c
ieee154e_vars.numAcks_expected = 1;
ieee154e_vars.synchronization_delay = 8; //slots
```

You can setup the message that you prefer or extend the code to dynamically establish the message to be sent, on purpose.

## Execution of the BitTransfer Protocol

The following steps need to be executed to run the protocol using Openmote-b devices and the OpenWSN protocol stack. Note that the following steps assume that you have correctly configured the toolchain to run OpenWSN on the Openmote-b hardware platform.

1. Compile the code via a Linux terminal and load it into Openmote-b devices. Open the terminal in the path `./openwsn-fw-antiJamming/` and execute the following:
```c
scons board=openmote-b toolchain=armgcc oos_openwsn; 
mv ./build/openmote-b_armgcc/projects/common/03oos_openwsn_prog.bin ./bootloader/openmote-cc2538/; 
cd bootloader/openmote-cc2538/; 
sudo python cc2538-bsl.py --bootloader-invert-lines -e -w -v -p /dev/ttyUSB1 03oos_openwsn_prog.bin; cd ../..;
```

2. Then, run the OpenVisualizer software from the path: `./openwsn-sw/software/openvisualizer`, by launching the command `sudo scons runweb`.

3. Open the web browser and go the following URL <a href="http://127.0.0.1:8080">http://127.0.0.1:8080</a>. Then, select one of the nodes, and set it as a DAG root, by pressing the button `Toggle DAG Root`.

4. In your terminal you will see that the devices will run the BitTransfer protocol, exchanging the bits of the message one by one.

## License
Being developed as a sub-project of the <a href="https://github.com/openwsn-berkeley/">OpenWSN</a> project, `BitTransfer` is released under the BSD3 Clause "New" or "Revised" License <a href="LICENSE">license</a>.
