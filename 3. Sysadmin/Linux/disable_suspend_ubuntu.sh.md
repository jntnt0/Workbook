#!/bin/bash
# Disable hibernation, suspend, and screen lock on Ubuntu
# Useful for servers, RDP hosts, or any machine that needs to stay awake

set -e

# Disable suspend, sleep, hibernate targets
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target

# Configure login manager behavior
LOGIND_CONF="/etc/systemd/logind.conf"
sudo sed -i 's/#HandleLidSwitch=.*/HandleLidSwitch=ignore/' $LOGIND_CONF
sudo sed -i 's/#HandleSuspendKey=.*/HandleSuspendKey=ignore/' $LOGIND_CONF
sudo sed -i 's/#HandleHibernateKey=.*/HandleHibernateKey=ignore/' $LOGIND_CONF
sudo sed -i 's/#IdleAction=.*/IdleAction=ignore/' $LOGIND_CONF

# Apply logind changes
sudo systemctl restart systemd-logind

# Disable GNOME idle and screen lock (runs as current user)
gsettings set org.gnome.desktop.session idle-delay 0
gsettings set org.gnome.desktop.screensaver lock-enabled false

echo "Done. Hibernation and suspend disabled."