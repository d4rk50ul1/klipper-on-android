# Using Android to run Klipper, Moonraker, Mainsail/Fuidd, and KlipperScreen

### Disclaimer: this is an ongoing work, still some changes pending. will try to update it when i can.
#### Original work by @RyanEwen (https://gist.github.com/RyanEwen/ae81fc48ad00397f1026915f0e6beed9)
## Requirements
- A rooted Android device with the following installed:
  - Linux Deploy app: https://play.google.com/store/apps/details?id=ru.meefik.linuxdeploy
  - XServer app: https://play.google.com/store/apps/details?id=x.org.server
  - Octo4a app: https://github.com/feelfreelinux/octo4a
  - Thermux app: https://play.google.com/store/apps/details?id=com.termux&hl=en&gl=US
- An OTG+Charge cable up and running for the same device ( please check this video for reference: https://www.youtube.com/watch?v=8afFKyIbky0)
- An already flashed printer using Klipper firmware. 
  - For reference : https://3dprintbeginner.com/how-to-install-klipper-on-sidewinder-x2/

- Init scripts for Klipper and Moonraker (scripts folder).
- XTerm script for KlipperScreen (scripts folder).
- 
 
## Setup Instructions
- Create a container within Linux Deploy using the following settings:
  - **Bootstrap**:
    - **Distro**: `Debian` (buster)
    - **Installation type**: `Directory`  
    *Note: You can choose `File` but make sure it's large enough as you can't resize it later and 2 GB is not enough.*  
    - **Installation path**: `/data/local/debian`  
    *Note: You can choose a different location but if it's within `${EXTERNALDATA}` then SSH may fail to start.*  
    - **User name**: `android`  
    *Note: You can choose something else if you make sure to update the scripts in this gist accordingly.*  
  - **Init**:
    - **Enable**: `yes`
    - **Init system**: `sysv`
  - **SSH**:
    - **Enable**: `yes`
  - **GUI**:
    - **Enable**: `yes`
    - **Graphics subsystem**: `X11`
    - **Desktop environment**: `XTerm`
- SSH into the container.
- Install Git and KIAUH: 
  ```bash
  sudo apt install git
  git clone https://github.com/th33xitus/kiauh.git
  ```
- Install Klipper, Moonraker, Mainsail (or Fluidd), and KlipperScreen:
  ```bash 
  kiauh/kiauh.sh
  ```
  *Note: KlipperScreen in particular will take a very long time (tens of minutes).*  
- Find your printer's serial device for use in Klipper's `printer.cfg`:  
  It will likely be `/dev/ttyACM0` or `/dev/ttyUSB0`. Check if either of those appear/disappear under `/dev/` when plugging/unplugging your printer.  
  
  If you cannot find your printer in `/dev/`, then you can install the Octo4a app which includes a custom implementation of the CH34x driver. You don't need to run OctoPrint within it, but you will need to run the Octo4a app. To do this:   
    - Install Octo4a from https://github.com/feelfreelinux/octo4a/releases
    - Run Octo4a and let it install OctoPrint (optionally tap the Stop button once it's done installing).
    - Make sure Octo4a sees your printer (it will be listed with a checked-box next to it).
    - Install Termux from https://f-droid.org/en/packages/com.termux
    - Run Termux and find the serial device created by Octo4a: 
        ```bash
        pkg install tsu
        sudo ls -al /data/data/com.octo4a/files/serialpipe
        ```
        You should see that `/data/data/com.octo4a/files/serialpipe` is a link to `/dev/pts/0` or similar. Whatever it's linked to is the serial port you should use in `printer.cfg`. You can uninstall Termux after this as it's not needed for anything else.
- Make the serial device accessible to Klipper:
    ```bash
    sudo chmod 777 /dev/ttyACM0
    # or 
    sudo chmod 777 /dev/ttyUSB0
    # or 
    sudo chmod 777 /dev/pts/0
    ```
- Install the init and xterm scripts from this gist:  
  ```bash
  sudo wget -O /etc/default/klipper https://github.com/d4rk50ul1/klipper-on-android/blob/6727edf8b8540da102ce7ffd66fa2a0fcb8d50d3/scripts/etc_default_klipper
  sudo wget -O /etc/init.d/klipper https://github.com/d4rk50ul1/klipper-on-android/blob/6727edf8b8540da102ce7ffd66fa2a0fcb8d50d3/scripts/etc_init.d_klipper
  sudo wget -O /etc/default/moonraker https://github.com/d4rk50ul1/klipper-on-android/blob/6727edf8b8540da102ce7ffd66fa2a0fcb8d50d3/scripts/etc_default_moonraker
  sudo wget -O /etc/init.d/moonraker https://github.com/d4rk50ul1/klipper-on-android/blob/6727edf8b8540da102ce7ffd66fa2a0fcb8d50d3/scripts/etc_init.d_moonraker
  sudo wget -O /usr/local/bin/xterm https://github.com/d4rk50ul1/klipper-on-android/blob/6727edf8b8540da102ce7ffd66fa2a0fcb8d50d3/scripts/usr_local_bin_xterm
  
  sudo chmod +x /etc/init.d/klipper 
  sudo chmod +x /etc/init.d/moonraker 
  sudo chmod +x /usr/local/bin/xterm
  
  sudo update-rc.d klipper defaults
  sudo update-rc.d moonraker defaults
  ```
- Stop the Debian container.
- Start XServer XSDL.
    - One time setup: 
        - Tap 'Change Device Configuration'
        - Change Mouse Emulation Mode to Desktop, No Emulation
- Start the Debian container.
- KlipperScreen should appear in XServer XSDL and Mainsail and/or Fluidd should be accesible using your Android device's IP address in a browser.

## Misc
You can start/stop Klipper and Moonraker manually by using the `service` command (eg: `sudo service start klipper`).  
Logs can be found in `/home/android/klipper_logs`.
