# Using Android to run Klipper, Moonraker, Mainsail/Fuidd, and KlipperScreen
## Alternative Version: https://github.com/gaifeng8864/klipper-on-android (uses fluidd and simplifies the script creation)

### Disclaimer: this is an ongoing work, still some changes pending. will try to update it when i can.
#### Original work by @RyanEwen (https://gist.github.com/RyanEwen/ae81fc48ad00397f1026915f0e6beed9)
#### Current setup i own: Artillery Genius Pro with Klipper Firmware + Lenovo Tab M8 running Klipper+Moonraker+Mainsail+klipperscreen
## Requirements
- A rooted Android device with the following installed:
  - Linux Deploy app: https://play.google.com/store/apps/details?id=ru.meefik.linuxdeploy
  - XServer app: https://play.google.com/store/apps/details?id=x.org.server
  - Octo4a app: https://github.com/feelfreelinux/octo4a
- An OTG+Charge cable up and running for the same device ( please check this video for reference: https://www.youtube.com/watch?v=8afFKyIbky0)
- An already flashed printer using Klipper firmware. 
  - For reference : https://3dprintbeginner.com/how-to-install-klipper-on-sidewinder-x2/

- Init scripts for Klipper and Moonraker (scripts folder).
- XTerm script for KlipperScreen (scripts folder).
 
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
  - **INIT**:
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
  
  If you cannot find your printer in `/dev/`, then you can check Octo4a app which includes a custom implementation of the CH34x driver. IMPORTANT: You don't need to run OctoPrint within it so once in the main screen of the app just stop it if it's running. To do this:   
    - Install Octo4a from https://github.com/feelfreelinux/octo4a/releases
    - Run Octo4a and let it install OctoPrint (optionally tap the Stop button once it's done installing).
    - Make sure Octo4a sees your printer (it will be listed with a checked-box next to it).
    - Now you need to go back to Linux Deploy and edit the container settings:
      - **MOUNTS**:
          - **Enable**: `yes`
          - **Mount points**: press on the "+" button
            - Source: `/data/data/com.octo4a/files`
            - Target: `/home/android/octo4a`
    - `/home/android/octo4a/serialpipe` is the serial port you need to use in your `printer.cfg`
- Make the serial device accessible to Klipper:
    ```bash
    sudo chmod 777 /dev/ttyACM0
    # or 
    sudo chmod 777 /dev/ttyUSB0
    # or 
    sudo chmod 777 /home/android/octo4a/serialpipe
    ```
- Install the init and xterm scripts from this gist:  
  ```bash
  sudo wget -O /etc/default/klipper https://raw.githubusercontent.com/d4rk50ul1/klipper-on-android/main/scripts/etc_default_klipper
  sudo wget -O /etc/init.d/klipper https://raw.githubusercontent.com/d4rk50ul1/klipper-on-android/main/scripts/etc_init.d_klipper
  sudo wget -O /etc/default/moonraker https://raw.githubusercontent.com/d4rk50ul1/klipper-on-android/main/scripts/etc_default_moonraker
  sudo wget -O /etc/init.d/moonraker https://raw.githubusercontent.com/d4rk50ul1/klipper-on-android/main/scripts/etc_init.d_moonraker
  sudo wget -O /usr/local/bin/xterm https://raw.githubusercontent.com/d4rk50ul1/klipper-on-android/main/scripts/usr_local_bin_xterm
  
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

## Telegram Bot
You can find the instructions how to setup the Telegram Bot [here](https://github.com/d4rk50ul1/klipper-on-android/blob/main/telegram_instructions.md)

## Troubleshooting (ongoing section based on comments)
- There might be the case that when accessing Mainsail through Browser, you get an error message and no connection to moonraker: mainsail Permission denied while connecting to upstream in `klipper_logs/mainsail_error.log`. To fix this you must change the file `/etc/nginx/nginx.conf`, change `user www-data;` to `user android;` 
- If anyone is having network issues in the container as a non root user after a few minutes, you need to disable deep sleep/idle. You can do that by using this command in a shell (termux or adb doesn't matter): `dumpsys deviceidle disable`. You may also need this app: [Wake Lock - CPU Awake] (https://play.google.com/store/apps/details?id=com.dambara.wakelocker)
- As per [ZerGo0](https://gist.github.com/ZerGo0) comments - The latest moonraker update seems to break a few things. There are some changes about the directory and file locations, but you can just sym link the new directories to the old ones using the included script: 
  ```bash
  sudo /etc/init.d/moonraker stop
  cd ~/moonraker
  scripts/data-path-fix.sh
  sudo /etc/init.d/moonraker start
  ```
  You can obviously also do this manually.
  You also have to add the following section to your `moonraker.conf`:
  ```bash
  [machine]
  validate_service: False
  validate_config: False
  provider: none
  ```
  There are a few more deprecated settings now, check the notifications or moonraker docs to find out what you need to remove.
 
- When checking the OTG+Charge Cable, each phone "recognizes" a different resistor, my recommendation is that once you build your cable, try not to solder the resistor directly to the 5 pin plug. Instead, use a breadboard temporary and test with different resistors. My approach was the following:
    - Without connecting anything to the microusb port (not tested with type-c) Open Octo4a app in your mobile
    - connect the printer to the usb modified cable (assuming that you have already builded one ;) ) 
    - connect the charger to the microusb male port
    - connect the modified cable to the mobile phone.
        - If the phone detects charge and Octo4a shows a popup requesting access to a serial device, you're done! the resistor is working. 
        - If the phone detects only charge and Octo4a doesn't show a popup requesting serial access, you must remove the resistor from the breadboard and try with a different one. 
        - Repeat this process until you figure it out.
        - My case i bought a resistor kit with values from 1k Ohm to 1M (about 20 possibilities). I changed them 1 by 1 until it worked. 
        - Just as a reference in case anyone has the same device. I used a Lenovo tab M8 (TB 8505F) and it worked with a 10k Ohm resistor!
         
