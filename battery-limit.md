ls /sys/class/power_supply/

ls /sys/class/power_supply/BAT1/

echo 80 | sudo tee /sys/class/power_supply/BAT1/charge_control_end_threshold

sudo nano /etc/systemd/system/battery-charge-limit.service

[Unit]
Description=Set battery charge thresholds
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo 80 > /sys/class/power_supply/BAT1/charge_control_end_threshold'

[Install]
WantedBy=multi-user.target

sudo systemctl enable battery-charge-limit.service
