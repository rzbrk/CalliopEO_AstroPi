**CalliopEO**

*[README](README.md)* | [Program Description](ProgramDescription.md) | [Testcases](testcases/testcases.md)
# Readme

## Description
CalliopEO is a Python script to facilitate ineraction between a Raspberry Pi microcomputer and a [Calliope Mini microcontroller board](https://calliope.cc/). If executed, the script detects, if a Calliope Mini is attached to a USB board of the Raspbery Pi and determins the serial port to communicate the Calliope Mini.

Place any program(s) to be executed by the Calliope Mini as zipped file(s) in the directory where the script `CalliopEO.py` resides. If executed the script will search for all zip archives, unpack the Calliope Mini program (HEX file) from the archive and flash the Calliope Mini with this program. After flashing, the Calliope Mini will reboot automatically and execute the program.

In the directory a sub-folder named `run_YYYYMMDD-HHMMSS` will be created. The HEX files flashed and executed on the Calliope Mini will be copied to this folder along with any data sent back by the program (files will end with `.data`). The initial zip archive in the main folder is renamed (additional suffix `.done`) to exclude this file from being processed again.

The `CalliopEO.py` script can collect data sent by the program on the Calliope Mini via the USB serial port. Therefore, prepare the Calliope Mini program to wait for the string `@START@`. Then, the Calliope Mini program should respond by sending `@START@` back to the `CalliopEO.py` script and only after this sending the data. After sending the data the Calliope Mini program should send the
message `@END@`.

## Execute
```
$ python3 CalliopEO.py [--max-data-size=bytes] [--max-script-execution-time=seconds]
```
`--max-data-size` is the maximum number of characters to be read from the Calliope Mini (except newline characters). With `--max-script-execution-time` you can specify a maximum time to accept input from the Calliope Mini program before terminating the connection.

## Hardware Setup
The Calliope Mini should be connected via USB to the Raspberry Pi.
```
     +--------------+
     |              |      USB      o---o---o
     | Raspberry Pi |===============| Call. |
     |              |               o---o---o
     +--------------+                
```

## Testing
To select and run tests execute `./testing.sh`. To get an overview of availbable tests see [Testcases.md](testcases/testcases.md).

## Software Installation
### Operating system
The project is intended to be used with the ESA [Astro Pi IR](https://astro-pi.org/) onboard the International Space Station ISS. The Astro Pi SBCs are running a dedicated flavour of Raspberry Pi OS not available for the public. But this software can also be installed on the publicly available [Raspberry Pi OS](https://www.raspberrypi.org/software/) running on any Raspberry Pi. For CalliopEO, the "OS Lite" version without desktop is sufficient. Follow the standard installation procedure.

### Installation using the setup.sh script
To set up the environment for the `CalliopEO.py` script a setup script `setup.sh` was established. It automatically performs all necesaary steps to provide a fully functional environment.

Before executing the setup script, connect the Calliope Mini to the Astro Pi/Raspberry Pi. Execute the setup script with the following command:
```
$ sudo setup.sh [username]
```
`username` is the name of the user to be created (enter without any brackets!). If this optional parameter is not given, the default `calliope` is used.

The setup script performs the following actions:
* Create a dedicated user (default: `calliope`)
* Add user to group `dialout` to be able to perform communication with the Calliope Mini over the serial port
* Set password for the new user
* Determine UUIDs for Calliope Mini's block devices and creating mount points in `/etc/fstab`. Later, this is used to mount the block devices for flashing the Calliope Mini. The mount points are defined in `/etc/fstab` in order to allow unprivileged users like `calliope` to mount/unmount the block devices.
* Copying necessary files to the user's home directory into a subfolder `~/calliopEO`, in particular `CalliopEO.py`
* Installing necessary Python modules from local wheel files. This is necessary due to security requirements set by ESA.

## De-Installation
The system can be cleaned up from the files necessary and the system settings done exclusively for CalliopEO using the procedure described as below. For this procedure it is assumed, that the username for the CalliopEO user is `calliope`. Replace `calliope` by the appropriate username if an alternative username was selected during the setup process (see above).

1. Login to the Raspberry Pi using a user different from `calliope`.
2. Remove the user `calliope` with the command
```
$ sudo userdel -r calliope
```
3. Verify that the above command successfully removed the home directory `/home/calliope` with the command `ls /home/`. If the home directory still exists, remove it using the command `$ sudo rm -rf /home/calliope`.
4. Create a backup of the file `/etc/fstab` using the command
```
$ sudo cp /etc/fstab /etc/fstab.backup
```
5. Open the file `/etc/fstab` in a text editor. Remove (or uncomment) all mount point definitions in `/etc/fstab` for CalliopEO. The section with the moint point definitions should look like the following:
```
# Mount points for Calliope Mini
/dev/disk/by-uuid/0123-4567 /home/calliope/mnt/mini vfat noauto,users 0 0
/dev/disk/by-uuid/089A-BCDE /home/calliope/mnt/flash vfat noauto,users 0 0
```
6. Restart the operating system. The de-installation is complete.
