# ruter_infoscreen
I made myself a small infoscreen for public transport in Oslo, based on Ruter's Dashboard service.
<img src="https://github.com/seventor/ruter_infoscreen/assets/1720848/a9d5ebf7-9b59-42de-96ea-15bfb4a2bc9c" width="450">
<img src="https://github.com/seventor/ruter_infoscreen/assets/1720848/3a28dcb5-8a7e-4341-8d09-24a3b8034825" width="450">

# Components
- Rasberry PI Zero 2
- HyperPixel 4.0" IPS Display - Square
- 32GB micro-SD card
- Power adapter for Rasberry Pi (5,1V/2A,5 micro-USB)
- GPIO Header (2x20 pin)

# Mounting
1. Solder on the GPIO Header to the Raspberry PI
2. Attach the Hyperpixel screen to the raspberry PI
3. Install Raspberry Pi OS (Legacy, 32 bit) on to the micro-SD card, put it in the Raspberry Pi, and boot up. The screen will not light up and work yet. You need to configure that after booting up.
4. Use SSH to logon to the Raspberry with the username/password you setup while installing the OS
5. Edit the `/boot/config.txt` by adding this line at the end of the file. This will make the display work on the Pi:

   ```
   # Hyperpixel Square display
   dtoverlay=vc4-kms-dpi-hyperpixel4sq
   ```
6. Go to Ruters webpage and create the dashboard you would like to show: https://mon.ruter.no/   
7. Make a bash-script that starts up `chromium-browser` with the dashboard from Ruter:
   ```
   #!/bin/bash

   # filename: ruter.sh

   # Define correct display
   export DISPLAY=:0
   
   sleep 30

   # Set the screen brightness
   gpio -g mode 19 pwm
   gpio -g pwm 19 28

   # Open chromium-browser in Kiosk-mode
   /usr/bin/chromium-browser --noerrdialogs --disable-infobars --disable-gpu --kiosk "https://mon.ruter.no/........." &

   sleep 30

   # Simulate a key refresh on the browser
   WID=$(xdotool search --onlyvisible --class chromium|head -1)
   xdotool windowactivate ${WID}
   xdotool key ctrl+F5
   ```
   The first `sleep 30` waits 30 seconds after boot before starting the browser. If I don't have that line, then very often at boot, the browser did not start correctly.
   Put in the URL for the dashboard you created at Ruters page in the script.
9. Install `xdotool` via apt-get: `sudo apt-get install xdotool`. You will need this to programatically send some keystrokes to the browser. I tried first without it, but very often the browser would just start with a blank white screen like it didn't load or rendered the dashboard correctly. Using `xdotool`, I wait 30 seconds after the browser has started, and then send keystorkes for refreshing the page to the browser.
10. Install WiringPi tool:
   ```
   wget https://project-downloads.drogon.net/wiringpi-latest.deb
   sudo dpkg -i wiringpi-latest.deb
   ```
You don't have to to this step, but I use it to adjust the screen brightness in the startup script. I set it to brightness 28 in the script. (This is the gpio lines in the script) 25 or lower seems to turn the screen off, and 127 seems to be the highest.
10. Make the script executable: `chmod +x ruter.sh`and then reboot: `sudo reboot now`

# Frame
The frame is made up by cutting a few pieces of wood in 45 dregrees angle. The dimensions I used is 21mm x 11mm, where the 11mm side will be showing beside the front of the screen. Cut in the back of the frame pieces so 4mm is left, 3mm wide. The 4 mm is the part that is covering the sides on the front of the screen. If you look at the picture the bottom and right side (seen from behind) of the frame need to be cut more than 3mm. Add 3mm, so 6mm in total. This is because the bottom of the screen has some wiring and the right side has the micro-SD that need som extra space.
Glue the frame pieces together, wait for it to dry, and then put the display inside the frame.
