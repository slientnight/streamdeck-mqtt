# Native Installation (Docker-Free) — Raspberry Pi Zero W

This guide documents how to run [streamdeck-mqtt](https://github.com/LukasOchmann/streamdeck-mqtt) **without Docker** on a Raspberry Pi Zero W (original, ARMv6).

> **Credit:** This project was created by [Lukas Ochmann](https://github.com/LukasOchmann/streamdeck-mqtt). This fork adds native installation support and fixes for the hidraw HIDAPI backend.

## What this fork changes

| Area | Original | This fork |
|------|----------|-----------|
| `main.py` | Queries deck before `deck.open()` | Calls `deck.open()` immediately after enumeration |
| `StreamDeckMQTT.py` | Calls `self.deck.open()` again inside `init()` | Removed duplicate open (deck is already open) |

## Setup

### 1. Install system dependencies

    sudo apt update
    sudo apt install -y git python3 python3-venv python3-dev build-essential pkg-config \
      libcairo2-dev libhidapi-hidraw0 libhidapi-libusb0 libhidapi-dev \
      libusb-1.0-0-dev libffi-dev libjpeg-dev zlib1g-dev \
      libfreetype6-dev liblcms2-dev libopenjp2-7-dev libtiff-dev

### 2. Fix HIDAPI backend

    sudo ln -sf libhidapi-hidraw.so.0.0.0 /usr/lib/arm-linux-gnueabihf/libhidapi-libusb.so.0
    sudo ln -sf libhidapi-hidraw.so.0.0.0 /usr/lib/arm-linux-gnueabihf/libhidapi-libusb.so
    sudo ldconfig

### 3. Udev rules

Create `/etc/udev/rules.d/99-streamdeck.rules` with:

    SUBSYSTEM=="usb", ATTRS{idVendor}=="0fd9", MODE="0666"
    KERNEL=="hidraw*", ATTRS{idVendor}=="0fd9", MODE="0666"

Then reload:

    sudo udevadm control --reload-rules
    sudo udevadm trigger

Unplug and replug the Stream Deck after this step.

### 4. Clone and install

    cd ~
    git clone https://github.com/slientnight/streamdeck-mqtt.git
    cd streamdeck-mqtt
    python3 -m venv .venv
    ./.venv/bin/python -m pip install --upgrade pip setuptools wheel
    ./.venv/bin/python -m pip install --prefer-binary -r requirements.txt

### 5. Configure

Create `.env`:

    MQTT_HOST=192.168.1.50
    MQTT_PORT=1883
    MQTT_USER=your-mqtt-user
    MQTT_PASS=your-mqtt-password

Create `data.json`:

    {"brightness":60,"keys":[]}

### 6. Test

    ./.venv/bin/python src/main.py

### 7. Systemd service (optional)

Create `/etc/systemd/system/streamdeck-mqtt.service`:

    [Unit]
    Description=Stream Deck MQTT bridge
    Wants=network-online.target
    After=network-online.target

    [Service]
    Type=simple
    User=<your-user>
    WorkingDirectory=/home/<your-user>/streamdeck-mqtt
    ExecStart=/home/<your-user>/streamdeck-mqtt/.venv/bin/python src/main.py
    Environment=PYTHONUNBUFFERED=1
    Restart=on-failure
    RestartSec=10

    [Install]
    WantedBy=multi-user.target

Then enable:

    sudo systemctl daemon-reload
    sudo systemctl enable --now streamdeck-mqtt

## License

MIT — Original work by [Lukas Ochmann](https://github.com/LukasOchmann).
