# tmux
Może być przydatne `sudo apt install xclip`

Żeby przeładować - `tmux source-file ~/.tmux.conf`

# Mapowanie CAPS LOCK na ESC
Chyba najlepiej specjalnym programem ktory omija DE. 

```
sudo add-apt-repository ppa:deafmute/interception
sudo apt update
sudo apt install interception-tools interception-caps2esc
```

If the PPA doesn't work for your Ubuntu, you can build from source:

```
sudo apt install cmake libudev-dev libyaml-cpp-dev libevdev-dev && \
git clone https://gitlab.com/interception/linux/tools.git /tmp/interception-tools && cd /tmp/interception-tools && cmake -B build -DCMAKE_BUILD_TYPE=Release && cmake --build build && sudo cmake --install build && \
git clone https://gitlab.com/interception/linux/plugins/caps2esc.git /tmp/caps2esc && cd /tmp/caps2esc && cmake -B build -DCMAKE_BUILD_TYPE=Release && cmake --build build && sudo cmake --install build
```
  
To zainstaluje program w bin (nie szkodzi ze zrodla sa w /tmp).

Teraz set up config i service:

```
sudo mkdir -p /etc/interception/udevmon.d && \
sudo tee /etc/systemd/system/udevmon.service << 'EOF'
[Unit]
Description=udevmon
Wants=systemd-udev-settle.service
After=systemd-udev-settle.service

[Service]
ExecStart=/usr/local/bin/udevmon -c /etc/interception/udevmon.d/caps2esc.yaml
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

Then:

```
sudo tee /etc/interception/udevmon.d/caps2esc.yaml << 'EOF'
- JOB: intercept -g $DEVNODE | caps2esc -m 1 | uinput -d $DEVNODE
  DEVICE:
    EVENTS:
      EV_KEY: [KEY_CAPSLOCK, KEY_ESC]
EOF
```

Start the service:

```
sudo systemctl daemon-reload
sudo systemctl enable --now udevmon
```

# inputrc
Żeby przeładować - `bind -f ~/.inputrc`
