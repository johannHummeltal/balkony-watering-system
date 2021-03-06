# IOT balkony-watering-system

## Introduction
You would like to have a green oasis on your balkony but you are tired of carrying heavy watering cans every day? Here's your solution - simply make your flowers a part of the internet of things! I built a watering system around a ESP32 microcontroller which is now watering our plants automatically. All you need is your smartphone for setting up the system and controlling it. The rest is done by a pump, a huge water barrel, some valves, some tube and a bunch of nozzles.
<!--![This is an image](./pictures/happy_plants.JPG =250x250)-->
<img src="./pictures/happy_plants.JPG" width="500">
In the current status of the project the system can automatically water up to 4 strands independently. I created a web interface to connect to the uC via wifi and adjust your system to perfectly meet your plants needs. 
Future features which are not yet realized include a soil moisture sensor which e.g. can detect if it was raining and the plants don't need to be watered.

## Hardware
<img src="./pictures/pump_barrel_valves.png" width="500">
The central water storage of the system is an old barrel (120l) which I found on the attic (according to the label it seemed to have been used for schnapps previously;) ). The barrol is attached to the rain-gutter of the nearby roof. So if you are lucky and it is raining sufficiently you do not even need to refill the barrel manually. For the case that this does not work out, I integrated a level sensor. It indicates if the barrel is empty and the uC sends an E-mail to the user asking him to refill the barell. 

A <a href="https://www.amazon.de/gp/product/B001CV02U4/ref=ppx_yo_dt_b_asin_title_o09_s00?ie=UTF8&psc=1/" target="_blank">submersible 12V pump</a> is floating inside the barrel and drives the watering system.
The outlet of the pump is connected to a <a href="https://www.amazon.de/gp/product/B07VG6VLL6/ref=ppx_yo_dt_b_asin_title_o01_s00?ie=UTF8&psc=1" target="_blank">fourfold magnetic valve</a>. On the one hand we need the valve since some of my plants are standing on the floor and are hence below the water level of the barrel. On the other hand the fourfold valve allows us to independently water up to 4 strands of different plants (e.g. different watering durations) with only one pump. 
The outputs of the valves are connected to a <a href="https://www.amazon.de/gp/product/B07GGVJRF9/ref=ppx_yo_dt_b_asin_title_o09_s01?ie=UTF8&psc=1" target="_blank">4mm tube containing multiple nozzles</a>, at least one for each single plant (see uppermost picture). 

## Electronics

<img src="./pictures/hardware.jpg" width="500">

The system is built around an ESP32. It is a powerful uC with integrated wifi connectivity. Its big advantage is, that it can be programmed using the Arduino IDE. The system is supplied by a standard 12V power supply. A DCDC converter provides 5V for the ESP32. I integrated a fuse and some capacitors to stabilize the supply voltage.
Both the pump and the valves are operated at 12V. They are controlled by the ESP via MOSFETs.

One of the biggest challenges was to make the electronics weather proofed for outdoor usage. The electronic board is stored inside a <a href="https://www.amazon.de/gp/product/B07F937V5M/ref=ppx_yo_dt_b_asin_title_o05_s00?ie=UTF8&psc=1" target="_blank">waterproof box</a>. To further protect electronics from humidity the board is painted with a special <a href="https://www.conrad.de/de/p/kontakt-chemie-plastik-70-74309-aa-isolier-und-schutzlack-200-ml-813621.html" target="_blank">polymer</a>. For all wirings I used  <a href="https://www.conrad.de/de/p/faber-kabel-030676-litze-sihf-j-3-x-1-mm-rot-meterware-1499131.html" target="_blank">waterproof and UV resistent cables</a>.

## Software

### ESP32 Software

`esp32_code` is flahsed onto the microcontroller using the Arduino IDE. The program is written such, that the ESP starts watering directly after booting. 
The system is watering each strand one after the other. The duration can be defined separately for every strand via the web interface. Once all strands are finished, the pump and the valves are turned off and the ESP goes to deep sleep to ensure minimum power consumption. After a predefined timeout (also adjusted via web interface) the ESP wakes up and the process starts from the beginning. Like this you can for example water your plants twice a day - in the morning and in the evening. 
Furthermore the ESP reads out a fill level sensor to notice when the barrel is empty. If this is the case the it sends an e-mail notification to the user using the `esp_mail_client.h` library. For this functionality and to provide the web interface functionality the ESP is connected to my local wifi network using the `wifi.h` library. If you want to adapt the program to make it work with your wifi network you will need to create a simple `credentials.h` file which contains your SSID, your mail adress and your passwords. To save energy I setup the program such that the ESP is only connected to wifi during the watering. 

### Web Interface
<img src="./pictures/screenshot_user_interface.png" width="500">
I created a user interface to setup and to control the watering system using your smartphone or computer. The file `watering_system.html` is uploaded into the file system of the uC using the Arduino IDE. When your device is logged into the same network as your ESP you can connect by typing the ESP's local IP address into the web browser. Once the connection is established the ESP sends the html file which can than be displayed by the remote device. Control commands and data is exchanged between ESP and web device using a websocket connection. 

When the user interface is connected you can use it to setup and store the watering durations (in seconds) for each strand (two in my case) as well as the timeout between two consecutive waterings (in hours). You can also use it to manually start an additional watering session - for example on an extra hot day. Last but not least the interface is also indicating the filling level of the barrel. 
