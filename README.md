# crey's-defli-tools
Just a bunch of DeFli tools and helper scripts.

## 1. "The SOCAT command"...

**WARNING:** before proceeding you have to have readsb installed (just proceed with the standard DeFli installation, stop on the socat command)

![install_defli_feeder](https://github.com/cr3you/creys-defli-tools/assets/73391409/0250ba8e-3726-49d0-af0d-a8edc2c103e2)

TL:DR

Copy all from the field below (you have nice "copy" icon in the right top corner)

Paste all to the ssh console (if you are using Putty -> right click in the terminal window) and press ENTER. There you go!

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
Description=Timer for the readsb service.

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
### 1.1 Updating previous defli_feeder service.
If you have set up my defli_service before DeFli said the socat command is mandatory then you have to update it because the syntax is a bit different:
```
sudo nano /usr/lib/systemd/system/defli_feeder.service
```
edit the line starting with "ExecStart"
to
```
ExecStart=/bin/sh -c 'socat tcp-l:30003,fork,reuseaddr tcp:77.68.73.29:30001'
```
Press CTRL+o then ENTER (confirm the filename) and then CTRL+x

Then restart the service
```
sudo systemctl stop defli_feeder
sudo systemctl daemon-reload
sudo systemctl start defli_feeder
```

**But then there is one problem...**

The changed command requires the readsb.service to stop before running the socat command.

You can do it by just stopping the service manually and starting again (assumming you have the defli_feeder service enabled and started).
```
sudo systemctl stop readsb.service
```
wait 6+ seconds and then 
```
sudo systemctl start readsb.service
```
**But you have to do it after every station reboot.**

Solution to that is delaying the start of readsb.service by enabling timer for it and start the service 1 minute after reboot (or a bit later).

To do it post this into your putty window and press ENTER :)

```
sudo tee /usr/lib/systemd/system/readsb.timer <<"EOF"
[Unit]
Description=Timer for the readsb service.

[Timer]
OnBootSec=1min

[Install]
WantedBy=timers.target
EOF
sudo systemctl daemon-reload
sudo systemctl stop readsb.service
sudo systemctl disable readsb.service
sudo systemctl enable readsb.timer
sudo systemctl start readsb.service
```


### 1.2 Debugging and checking if it works
And now some debugging and explamnations maybe :)

If you want to check if your defli_feeder service is running and has no errors use this command:

```
sudo systemctl status defli_feeder.service
```
If it's running you should see something similar:
![image](https://github.com/cr3you/creys-defli-tools/assets/73391409/7c1fa242-9988-4530-9ddb-69165cb94d88)


