# Enumerate all keyboard hardware IDs and write to a temp file
# Use the output to identify hardware IDs for Disable_HID_Keyboards_By_ID.ps1

Get-PnpDevice -Class Keyboard |
    Get-PnpDeviceProperty DEVPKEY_Device_HardwareIds |
    ForEach-Object {
        "$($_.InstanceId) | $($_.KeyName) | $($_.Type) | $($_.Data -join ', ')"
    } | Set-Content "$env:TEMP\kbd.txt"

notepad "$env:TEMP\kbd.txt"

# To remove registry policy blocks applied by the companion .reg file:
# reg delete "HKLM\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions\DenyDeviceIDs" /f
# reg delete "HKLM\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions" /f