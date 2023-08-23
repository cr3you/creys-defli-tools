# defli-tools
a bunch of DeFli tools and helper scripts


## 1. "The SOCAT command"...

**WARNING:** before proceeding you have to have readsb installed (just proceed with the standard DeFli installation, stop on the socat command)

TL:DR

Copy all from the field below (you have nice "copy" icon in the right top corner)

Paste all to the ssh console (if you are using Putty -> right click in the terminal window)

Wait a bit and press enter. There you go!

```
sudo apt -y install socat
sudo mkdir -p /usr/lib/systemd/system/
sudo tee /usr/lib/systemd/system/defli_feeder.service <<"EOF"
[Unit]
Description=defli_feeder
Requires=network.target
After=network.target

[Service]
ExecStart=/bin/sh -c 'socat tcp-l:30003,fork,reuseaddr tcp:77.68.73.29:30001'
Restart=on-failure
KillMode=control-group
RestartSec=5s

[Install]
WantedBy=multi-user.target
EOF
sudo tee /usr/lib/systemd/system/readsb.timer <<"EOF"
[Unit]
Description=Timer for the defli_feeder service

[Timer]
OnBootSec=1min

[Install]
WantedBy=timers.target
EOF
sudo systemctl daemon-reload
sudo systemctl stop readsb.service
sudo systemctl disable readsb.service
sudo systemctl enable readsb.timer
sudo systemctl enable defli_feeder
sudo systemctl start defli_feeder
sudo systemctl start readsb.service
```

### 1.1 Debugging and checking if it works
And now some debugging and explamnations maybe :)

If you want to check if your defli_feeder service is running and has no errors use this command:

```
journalctl -u defli_feeder.service -b -f
```
