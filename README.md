# pc-nrfutil

[![Latest version](https://img.shields.io/pypi/v/nrfutil.svg)](https://pypi.python.org/pypi/nrfutil)
[![License](https://img.shields.io/pypi/l/nrfutil.svg)](https://pypi.python.org/pypi/nrfutil)

nrfutil is a Python package that includes the `nrfutil` command line utility and the `nordicsemi` library.

## Introduction

This application and its library offer the following features:

* Device Firmware Update package generation
* Cryptographic key generation, management and storage
* Bootloader DFU settings generation and display
* Device Firmware Update procedure over Bluetooth Low Energy

## License

See the [license file](LICENSE) for details.

## Versions

There are 2 different and incompatible DFU package formats:

* legacy: used a simple structure and no security
* modern: uses Google's protocol buffers for serialization and can be cryptographically signed

The DFU package format transitioned from legacy to modern in SDK 12.0. Depending on the SDK version
that you are using you will need to select a release of this tool compatible with it:

* Version 0.5.2 generates legacy firmware packages compatible with **nRF SDK 11.0 and older**
* Versions 1.5.0 and later generate modern firmware packages compatible with **nRF SDK 12.0 and newer**

## Installing from PyPI

To install the latest published version from the Python Package Index simply type:

    pip install nrfutil

This will also retrieve and install all additional required packages.

**Note**: Please refer to the [pc-ble-driver-py PyPI installation note on Windows](https://github.com/NordicSemiconductor/pc-ble-driver-py#installing-from-pypi) if you are running nrfutil on this operating system.

**Note**: To use the `dfu ble` option you will need to set up your boards to be able to communicate with your computer.  You can find additional information here: [Hardware setup](https://github.com/NordicSemiconductor/pc-ble-driver/tree/self_contained_driver#hardware-setup). 

## Running and installing from source

You will need to clone the present repository first to run or install nrfutil from source.

### Prerequisites

To install nrfutil from source the following prerequisites must be satisfied:

* [Python 2.7 (2.7.6 or newer, not Python 3)](https://www.python.org/downloads/)
* [pip](https://pip.pypa.io/en/stable/installing.html)
* setuptools (upgrade to latest version): `pip install -U setuptools`

Additionally, if you want to generate a self-contained executable:  

* PyInstaller: `pip install pyinstaller`

**IMPORTANT NOTE**: py2exe is no longer supported and you must use PyInstaller instead to generate an executable

### Requirements

To obtain and install all required Python packages simply run:

```
pip install -r requirements.txt
```

### Running from source

You can run the program directly without installing it by executing:
```
python nordicsemi/__main__.py
```

### Installing from source

To install the library to the local Python site-packages and script folder:  
```
python setup.py install
```

To generate a self-contained executable version of the utility:  
```
pyinstaller nrfutil.spec
```

**Note**: Some anti-virus programs will stop PyInstaller from executing correctly when it modifies the executable file.

**Note**: Please refer to the [pc-ble-driver-py PyPI installation note on Windows](https://github.com/NordicSemiconductor/pc-ble-driver-py#installing-from-pypi) if you are running nrfutil on this operating system.

**Note**: To use the `dfu ble` option you will need to set up your boards to be able to communicate with your computer.  You can find additional information here: [Hardware setup](https://github.com/NordicSemiconductor/pc-ble-driver/tree/self_contained_driver#hardware-setup). 

## Usage

To get info on usage of nrfutil:  
```
nrfutil --help
```

### Commands
There are several commands that you can use to perform different tasks related to DFU:

#### pkg
This set of commands allow you to generate a package for Device Firmware Update.

##### generate
Generate a package (.zip file) that you can later use with a mobile application or any other means to update the firmware of an nRF5x IC over the air. This command takes several options that you can list using:
```
nrfutil pkg generate --help
```
Below is an example of the generation of a package in debug mode from an application's `app.hex` file:
```
nrfutil pkg generate --debug-mode --application app.hex --key-file key.pem app_dfu_package.zip
```
When using debug mode you don't need to specify versions for hardware and firmware, so you can develop without having to worry about versioning your application. If you want to generate a package for production, you will need to do so without the `--debug-mode` parameter and specify the versions:
```
nrfutil pkg generate --hw-version 1 --sd-req 0x80 --application-version 4 --application app.hex --key-file key.pem app_dfu_package.zip
```

The following table lists the FWIDs which are used to identify the SoftDevice versions both included in the package and installed on the target device to perform the required SoftDevice version check:

SoftDevice            | FWID (sd-req)
----------------------| -------------
`s130_nrf51_1.0.0`    | 0x67
`s130_nrf51_2.0.0`    | 0x80
`s132_nrf52_2.0.0`    | 0x81
`s130_nrf51_2.0.1`    | 0x87
`s132_nrf52_2.0.1`    | 0x88
`s132_nrf52_3.0.0`    | 0x8C

Not all combinations of Bootloader, SoftDevice and Application are possible when generating a package. The table below summarizes the support for different combinations.

The following conventions are used on the table:

* BL: Bootloader
* SD: SoftDevice
* APP: Application

Combination   | Supported | Notes
--------------| ----------|-------
BL            | Yes       |
SD            | Yes       | **SD must be of the same Major Version**
APP           | Yes       |
BL + SD       | Yes       |
BL + APP      | No        | Create two .zip packages instead
BL + SD + APP | Yes       |
SD + APP      | Yes       | **SD must be of the same Major Version**

##### display
Use this option to display the contents of a DFU package in a .zip file.
```
nrfutil pkg display package.zip
```

#### dfu
This set of commands allow you to perform an actual firmware update over a serial or BLE connection.

##### ble
Perform a full DFU procedure over a BLE connection. This command takes several options that you can list using:
```
nrfutil dfu ble --help
```
Below is an example of the execution of a DFU procedure of the file generated above over BLE using a connectivity IC connected to COM3, where the remote BLE device to be upgraded is called "MyDevice":
```
nrfutil dfu ble -pkg app_dfu_package.zip -p COM3 -n "MyDevice" -f
```
The `-f` option instructs nrfutil to actually program the board connected to COM3 with the connectivity software required to operate as a serialized SoftDevice. Use with caution as this will overwrite the contents of the IC's flash memory.

##### serial
**Note**: DFU over a serial line is currently disabled

Perform a full DFU procedure over a serial (UART) line. This command takes several options that you can list using:
```
nrfutil dfu serial --help
```
Below is an example of the execution of a DFU procedure of the file generated above over COM3 at 115200 bits per second:
```
nrfutil dfu serial -pkg app_dfu_package.zip -p COM3 -b 115200
```

#### keys
This set of commands allow you to generate and display cryptographic keys used to sign and verify DFU packages.

##### generate
Generate a private (signing) key and store it in a file in PEM format.
The following will generate a private key and store it in a file named `private.pem`:
```
nrfutil keys generate private.pem
```

##### display
Display a private (signing) or public (verification) key from a PEM file taken as input. This command takes several options that you can list using:
```
nrfutil keys display --help
```
Below is an example of displaying a public key in code format from the key file generated above:
```
nrfutil keys display --key pk --format code private.pem
```

#### settings
This set of commands allow you to generate and display Bootloader DFU settings, which must be present on the last page of available flash memory for the bootloader to function correctly.

##### generate
Generate a flash page of Bootloader DFU settings  and store it in a file in .hex format. This command takes several options that you can list using:
```
nrfutil settings generate --help
```
You can generate a .hex file with Bootloader DFU settings matching a particular flashed application by providing the application .hex to nrfutil:
```
nrfutil settings generate --family NRF52 --application app.hex --application-version 3 --bootloader-version 2 --bl-settings-version 1 sett.hex
```
The `--bl-settings-version` depends on the SDK version used. Check the following table to find out which version to use:

SDK Version   | BL Settings Version
------------- | -------------------
12.0          | 1

The Bootloader DFU settings version supported and used by the SDK you are using can be found in `nrf_dfu_types.h` in the `bootloader` library.

##### display

Use this option to display the contents of the Bootloader DFU settings present in a .hex file. The .hex file might be a full dump of the IC's flash memory, obtained with `nrfjprog`:
```
nrfjprog --readcode flash_dump.hex
```
After you have obtained the contents of the flash memory, use nrfutil to decode the Bootloader DFU settings section:
```
nrfutil settings display flash_dump.hex
```
**Note**: nrfutil will autodetect the IC family when displaying the contents of the Bootloader DFU settings.

#### version
This command displays the version of nrfutil.

## Init Packet customisation

If you want to modify the Init Packet, which is the packet that contains all of the metadata and that is sent before the actual firmware images, you will need to recompile the Google Protocol Buffers `.proto` file and adapt `nrfutil` itself.

Note that if you modify the format of the Init Packet, you *will need to do the same in the bootloader*, meaning that you will have to recompile it to adapt it to the new format.

### Modifying the Protocol Buffers file

Edit `dfu-cc.proto` and modify the Init packet to suit your needs. Additional information on the format of `.proto` files can be found [here](https://developers.google.com/protocol-buffers/).

### Compiling the Protocol Buffers file

After you have modified the `.proto` file you will need to compile it to generate the corresponding Python file that will be then usable from the `nrfutil` source code. To do that install the Protocol Buffers compiler from [here](https://developers.google.com/protocol-buffers/docs/downloads) and then execute:

```
$ protoc --python_out=<dest_folder> dfu-cc.proto
```
Where `<dest_folder>` is an empty folder where the Protocol Buffers compiler will write its output.

After compilation is complete, a file named `<dest_folder>/dfu_cc_pb2.py` will be created. You can then use this file to overwrite the one in [nordicsemi/dfu](nordicsemi/dfu) to start using the new Init Packet format.

### Adapting nrfutil to the new Init Packet format

Once you have the customized `dfu_cc_pb2.py` file in your repository you will need to adapt the actual tool to conform to the new format you have designed. To do that you will need to alter several of the Python source files included, as well as potentially having to modify the command-line options to fit the contents of your Init Packet.
Refer to [init_packet_pb.py](nordicsemi/dfu/init_packet_pb.py) and [package.py](nordicsemi/dfu/package.py) for the contents themselves, and to [\_\_main\_\_.py](nordicsemi/__main__.py) for the command-line options.

### Adapting the bootloader to the new Init Packet format

Since you have modified the Init Packet format you will have to do the same with the embedded bootloader, which can be found in the Nordic nRF5 SDK under `examples/dfu/bootloader_secure`.

