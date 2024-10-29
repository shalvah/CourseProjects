# [Introduction to the Internet of Things] Temperature reader
Goal: create a smart temperature reader that captures the temperature in a room and allows me to view the data in realtime from anywhere.

Chosen solution: A microcontroller/SBC with a temperature sensor, which reads the temperature at intervals and sends to a web server I set up. The web app stores the temperature readings and I can view at any time in its UI.

## Components used
- Microcontroller: [Raspberry Pi Pico WH](https://www.conrad.de/de/p/raspberry-pi-pico-wh-mikrocontroller-pico-wh-2825613.html). I went with the Pico model because I didn't need the full computing capabilities of the Pi line. I chose the WH variant for two reasons:
  - Wi-Fi connectivity [W] - It needs to send the temperature data over the network, and Wi-Fi is the simplest way to do that.
  - pre-soldered headers [H] - The headers are what you use to connect the chip to other devices, and I didn't want to do any soldering.
- Temperature sensor: DHT11 [from Joy-IT](https://www.conrad.de/de/p/joy-it-sen-ky015tf-temperatur-feuchtigkeitssensor-1-st-1695379.html)
- Breadboard and jumper cables for connecting the components
- RGB LED. Not required, but I added this later because it's nice to have a visual indicator, and it helps with observability.

## Circuit diagram



## Procedure
1. As this was my first time with a microcontroller, I first needed to set up my environment and get familiar with the chip. I followed [the official Getting Started tutorial](https://projects.raspberrypi.org/en/projects/get-started-pico-w), which guided me in loading it with the correct firmware (MicroPython), configuring my environment (installing the Thonny IDE) and running some test programs. 
2. Next, I connected the chip to the temperature sensor via the breadboard and jumper cables. This was fairly straightforward, as I already had previous knowledge of breadboards. The important step was to follow the directions provided by the sensor manufacturer (the manual detailed how to connect it to a Pico). I also double-checked the Pico's pinout diagram (a diagram showing the names of each pin, as they aren't always visible on the Pico).
3. Then I wrote a basic program on the microcontroller (connected to my PC via USB), which focused on reading from the sensor and connecting to my home Wi-Fi. The sensor manual had a sample program, and I found the `picozero` library documentation helpful too. I tested it by running it on the microcontroller from my machine; this helped me get familiar with what to expect.
4. After that, I built and deployed the API to receive temperature readings, store and render them, a simple Node.js server ([source code](https://github.com/shalvah/mercury)).
5. I went back to finish the program by having it send the temperature readings to the API server.
6. When all was working, I saved it to the Pico as `main.py`, so it runs whenever the chip is plugged in. After testing some more, I disconnected it from my PC and plugged it into a persistent power supply, and now it runs by itself. 

## Software
Raspberry Pico program:

```python

```

## Results
After a day of recording:

![Temperature and humidity line chart](img/iot_temperature_reader_1.png)


## Challenges
- **Powering the Pico**: In the initial phases, the chip was plugged in to my PC, but I needed an always-on solution, 
while taking note of the Pico's power requirements, as specified in its datasheet.
Eventually, I simply plugged it into an extension box with USB slots I had lying around.
Another option is to get a dedicated power supply, such as [this](https://www.amazon.de/dp/B0CF44S2HG?ref=ppx_yo2ov_dt_b_fed_asin_title).
- **Interrupting `main.py`**: Since `main.py` always runs on startup, if you have an infinite loop (read data - send - repeat),
it becomes hard to interrupt this. I was unable to edit the program or run any new programs via Thonny.
Eventually I had to upload a new firmware that deleted all files on the chip, then reupload the MicroPython firmware and start again.
My eventual solution was to add a 5-second delay before entering the loop, which gives me enough time to interrupt via Thonny.
- **Wi-Fi connection failures or other interruptions**: When I interrupted the program on my PC, 
it would sometimes fail to connect to Wi-Fi afterwards. I never totally solved this, 
but added a redundancy feature to reset the chip and retry connection.
- **Memory leaks**: A minor problem was running into an "ENOMEM" (out of memory error) after two HTTP requests. 
The solution to this was adding `response.close()` after each request.
- **Visibility**: The chip is an opaque, limited system, so it can be quite hard to know what's going on when it's not connected to Thonny.
To improve this, I used a lighting system: the onboard green LED would blink to indicate it was connecting to Wi-Fi, 
and an extra RGB LED would pulse to show it was operating normally.

## Improvements
- **Handling and indicating more error states**: For instance, too many failed network requests could turn the LED red.
A failed Wi-Fi connection could cause it to flash red.  
- **Informational lighting**: I could also use a LED to indicate when the temperature goes above or below threshold
- **Automatic control**: I could connect this to my thermostat with an actuator to regulate the temperature.