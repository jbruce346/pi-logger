# pi-logger
 
A Python service that logs a Raspberry Pi's CPU temperature to CSV every minute. It runs unattended under systemd, restarts on its own after a crash or reboot, and comes with a script that plots the data. 
## How it works
 
- **`logger.py`** reads the CPU temperature from `/sys/class/thermal/thermal_zone0/temp` (the kernel reports millidegrees, so it divides by 1000) and appends a timestamped row to `cpu_temp.csv` every 60 seconds. It writes the header only when the file is new, and flushes after every row so a crash or power cut loses at most one reading.
- **`logger.service`** is a systemd unit that starts the logger at boot and restarts it 10 seconds after any crash (`Restart=always`). It waits for `time-sync.target`, so readings aren't stamped with the wrong time before the clock has synced over the network.
- **`plot.py`** reads the CSV with pandas and plots temperature against time with matplotlib, marking the reboots.
## Hardware
 
- First built on a Raspberry Pi Zero 2 W. It now runs on a Raspberry Pi 3A+: the same SD card moved straight across.
- Raspberry Pi OS Lite (no desktop), managed headless over SSH.
- No extra hardware: the temperature sensor is built into the Pi's processor.
## Setup
 
On the Pi:
 
```bash
mkdir -p ~/logger && cd ~/logger
# copy logger.py and logger.service here
sudo cp logger.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now logger.service
systemctl status logger.service   # should say active (running)
tail -f cpu_temp.csv              # a new row every minute
```
 
The paths in `logger.py` and `logger.service` use my username (`joshbruce1107`). Change them to yours.
 
To plot, copy the CSV to a laptop and run:
 
```bash
scp <user>@<pi-hostname>.local:~/logger/cpu_temp.csv .
python plot.py
```
 
## Results
 
<"The plot shows a steady temperature throughout most the day with minor spikes when I enter the room, or dropoffs when I open the window. There are also a few major spikes caused by restarting the Pi." -->
 
 
## What came next
 
This logger was the starting point for:
- **BME280 environment logging** (temperature, humidity, pressure) on the same Pi
- **[rain-predictor](https://github.com/jbruce346/rain-predictor)**: predicts rain in the next 12 hours from barometric pressure, evaluated against baselines with the Brier score
 
## Use of AI
 
<"I used Claude as a tutor while building this: it explained systemd and SSH concepts and helped me debug. I wrote and ran the code and set up the Pi myself." -->
 
