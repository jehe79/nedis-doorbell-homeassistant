# Hacking your Nedis video doorbell for chime events in Home Assistant
A brief guide on how to hack your Nedis video doorbell to integrate the chime into Home Assistant.

## Support
If you appreciate this project, consider [buying me a coffee](https://www.buymeacoffee.com/cci5pkj1kx)!

## What you need
* Nedis SmartLife video doorbell (or any doorbell compatible with Nedis door chime) [link](https://nedis.com/en-us/product/safety-and-security/doorbells/video-doorbells/550702007/smartlife-video-doorbell-wi-fi-transformer-full-hd-1080p-cloud-storage-optional-microsd-not-included-ip54-with-motion-sensor-night-vision-black-grey) 
* Nedis SmartLife door chime [link](https://nedis.com/en-us/product/safety-and-security/doorbells/wireless-doorbells/550702011/smartlife-chime-wi-fi-accessory-for-wificdp10gy-wificdp30wt-wificdp40cwt-usb-powered-4-sounds-5-v-dc-adjustable-volume-black) [amazon SE](https://www.amazon.se/-/en/Nedis-Ficdpc10Bk-Wireless-Doorbell-Accessories/dp/B07NDVKS6Z)
* Any ESPHome compatible device like ESP32 D1 Mini [link](https://www.amazon.se/AZDelivery-WiFi-Modul-Bluetooth-Development-kompatibel/dp/B08BTRQNB3?th=1)
* A few wires, soldering skills and/or connectors

## Onvif events - doesn't seem to work
The doorbell has onvif support but it seems incomplete and I didn't get it to work.

![Onvif support](https://github.com/user-attachments/assets/2ed89fab-8752-48cb-aa5e-f0fba7bf518b)

## Hardware hack
I've got a few ESPHome devices and got to think about the motion detection sensor - it's being triggered by a GPIO pin going high (or low).
If only I could find a pin inside the chime that would trigger when the door bell is pressed. The chime is pretty cheap and it wouldn't be the world if it somehow ended up broken.

### Opening the chime
Opening up the chime is pretty straight forward - a small flat head screwdriver to pop off the rear plate.

<img src="https://github.com/jehe79/nedis-doorbell-homeassistant/blob/main/images/chime-inside-dig.jpg?raw=true" alt="Inside of Nedis chime" width="400"/>

Inside the chime there are 3 main IC's where #1 seems to handle RF, #2 the main controller and #3 runs the speaker. Without knowing - my best guess is that the main controller activates the speaker with something that also could trigger the ESPHome GPIO.

### Wire
After some trouble with GPIO's not responding - even on manual triggering I finally got things working. 
It seemed like the the 5th leg of IC #3 (speaker chip) was connected to the main controller (#2) via some filter components - this is where I connected my ESP.

Thanks to my interest in FPV drones I had a lot of spare connectors where I could make a split cable for the USB power. There are no solder points for the power connector so this was the best solution until I knew it worked. 

<img src="https://github.com/jehe79/nedis-doorbell-homeassistant/blob/main/images/chime_pin_solder.jpg?raw=true" alt="Pin to solder inside chime" width="400"/>

<img src="https://github.com/jehe79/nedis-doorbell-homeassistant/blob/main/images/esp32_wire.jpg?raw=true" alt="ESP32 wire" width="400"/>

<img src="https://github.com/jehe79/nedis-doorbell-homeassistant/blob/main/images/chime_connected.jpg?raw=true" alt="Chime and esp connected" width="400"/>

### ESPHome
If you havn't looked into [ESPHome](https://esphome.io/index.html) yet - you better do!

### Sensor code
In ESPHome this is all the code needed to add the sensor.

```yaml
binary_sensor:
  - platform: gpio
    pin:
      number: GPIO12
    name: "Doorbell Sensor"
    device_class: sound
    filters:
      - delayed_on_off:
          time_on: 10ms
          time_off: 5s
``` 

### Final result
At last - why not 3D print a case for the ESP, in my case [this](https://www.thingiverse.com/thing:4871082) (but I had to increase the box by 3% and kept the lid at original scale).

<img src="https://github.com/jehe79/nedis-doorbell-homeassistant/blob/main/images/chime_done.jpg?raw=true" alt="Nedis chime with esp done" width="400"/>
