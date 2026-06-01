#!/bin/bash
# =============================================================================
# disable_suspend_ubuntu.sh
# Disable sleep, suspend, hibernation, and screen lock on Ubuntu systems.
#
# Overview:
#   Ubuntu workstations and servers configured with a desktop environment will
#   suspend or lock the screen after a period of inactivity by default. This
#   behavior is appropriate for personal workstations but is problematic for
#   machines serving as RDP hosts, lab servers, kiosk endpoints, or any system
#   that must remain available without physical interaction.
#
#   This script disables suspend and hibernation at the systemd level, configures
#   logind to ignore lid-close and power key events, and disables the GNOME idle
#   timer and screen lock for the current user session. All changes persist across
#   reboots.
#
# Usage:
#   Run as a user with sudo privileges. The GNOME settings commands apply to the
#   currently logged-in user session and do not require elevation.
#
#       sudo bash disable_suspend_ubuntu.sh
#
# Tested on:
#   Ubuntu 22.04 LTS, Ubuntu 24.04 LTS
#
# Reverting:
#   To restore default suspend behavior:
#       sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target
#       sudo sed -i 's/HandleLidSwitch=ignore/#HandleLidSwitch=suspend/' /etc/systemd/logind.conf
#       sudo systemctl restart systemd-logind
#       gsettings reset org.gnome.desktop.session idle-delay
#       gsettings reset org.gnome.desktop.screensaver lock-enabled
# =============================================================================

set -euo pipefail

# ---------------------------------------------------------------------------
# Helper: print a timestamped status message
# ---------------------------------------------------------------------------
log() {
    echo "[$(date '+%H:%M:%S')] $1"
}

# ---------------------------------------------------------------------------
# Confirm the script is running with the necessary privileges
# ---------------------------------------------------------------------------
if [[ $EUID -ne 0 ]]; then
    echo "Error: This script must be run with sudo or as root." >&2
    exit 1
fi

# ---------------------------------------------------------------------------
# Step 1 — Mask systemd sleep and suspend targets
# Masking prevents these targets from being activated by any means,
# including manual systemctl calls and ACPI events.
# ---------------------------------------------------------------------------
log "Masking systemd sleep, suspend, hibernate, and hybrid-sleep targets..."
systemctl mask \
    sleep.target \
    suspend.target \
    hibernate.target \
    hybrid-sleep.target

log "Systemd targets masked."

# ---------------------------------------------------------------------------
# Step 2 — Configure systemd-logind to ignore hardware power events
# Updates /etc/systemd/logind.conf to suppress lid-close, suspend key,
# hibernate key, and idle action events.
# A backup of the original file is created before modification.
# ---------------------------------------------------------------------------
LOGIND_CONF="/etc/systemd/logind.conf"
LOGIND_BACKUP="${LOGIND_CONF}.bak.$(date '+%Y%m%d%H%M%S')"

log "Backing up $LOGIND_CONF to $LOGIND_BACKUP..."
cp "$LOGIND_CONF" "$LOGIND_BACKUP"

log "Configuring logind to ignore power events..."
sed -i 's/^#\?HandleLidSwitch=.*/HandleLidSwitch=ignore/'       "$LOGIND_CONF"
sed -i 's/^#\?HandleSuspendKey=.*/HandleSuspendKey=ignore/'     "$LOGIND_CONF"
sed -i 's/^#\?HandleHibernateKey=.*/HandleHibernateKey=ignore/' "$LOGIND_CONF"
sed -i 's/^#\?IdleAction=.*/IdleAction=ignore/'                 "$LOGIND_CONF"

log "Restarting systemd-logind to apply changes..."
systemctl restart systemd-logind

log "logind configuration applied."

# ---------------------------------------------------------------------------
# Step 3 — Disable GNOME idle timer and screen lock for the current user
# These settings apply to the user session running the script. If this script
# is executed via sudo from a desktop session, $SUDO_USER is used to identify
# the target user. If run as root directly, the settings are applied to root's
# session only.
# ---------------------------------------------------------------------------
TARGET_USER="${SUDO_USER:-$USER}"

log "Disabling GNOME idle and screen lock for user: $TARGET_USER..."

# Run gsettings as the target user, not as root
sudo -u "$TARGET_USER" gsettings set org.gnome.desktop.session    idle-delay  0
sudo -u "$TARGET_USER" gsettings set org.gnome.desktop.screensaver lock-enabled false

log "GNOME session settings applied."

# ---------------------------------------------------------------------------
# Summary
# ---------------------------------------------------------------------------
echo ""
log "All changes applied successfully."
log "Systemd targets masked    : sleep, suspend, hibernate, hybrid-sleep"
log "logind power events       : lid-close, suspend key, hibernate key, idle — all ignored"
log "GNOME idle delay          : disabled (0 seconds)"
log "GNOME screen lock         : disabled"
log "logind backup             : $LOGIND_BACKUP"
echo ""
log "No reboot is required. Changes take effect immediately."
