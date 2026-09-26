# tmux
Może być przydatne `sudo apt install xclip`

Żeby przeładować - `tmux source-file ~/.tmux.conf`

# Mapowanie CAPS LOCK na ESC

Chyba najlepiej specjalnym programem ktory omija DE (interception-tools + caps2esc).
Od Ubuntu 24 oba są w oficjalnych repo — nie trzeba PPA ani budowania ze źródeł:

```bash
sudo apt install interception-tools interception-caps2esc
```

Uwaga: w paczce Debiana/Ubuntu `intercept` nazywa się `interception` (konflikt nazw),
a binarki są w `/usr/bin`, nie w `/usr/local/bin`.

Config i service:

```bash
sudo mkdir -p /etc/interception/udevmon.d && \
sudo tee /etc/systemd/system/udevmon.service << 'EOF'
[Unit]
Description=udevmon
Wants=systemd-udev-settle.service
After=systemd-udev-settle.service

[Service]
ExecStart=/usr/bin/udevmon -c /etc/interception/udevmon.d/caps2esc.yaml
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

```bash
sudo tee /etc/interception/udevmon.d/caps2esc.yaml << 'EOF'
- JOB: interception -g $DEVNODE | caps2esc -m 1 | uinput -d $DEVNODE
DEVICE:
  EVENTS:
    EV_KEY: [KEY_CAPSLOCK, KEY_ESC]
EOF
```

Start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now udevmon
```

Debug: `journalctl -u udevmon -b --no-pager | tail`

### Migracja z Ubuntu 22 (stara wersja budowana ze źródeł)

Stare binarki w `/usr/local/bin` są zlinkowane z `libyaml-cpp.so.0.7`, której nie ma w Ubuntu 24.
Trzeba je usunąć (inaczej mają pierwszeństwo w PATH przed `/usr/bin`):

```bash
sudo rm /usr/local/bin/{udevmon,intercept,uinput,mux,caps2esc}
```

# inputrc
Żeby przeładować - `bind -f ~/.inputrc`
