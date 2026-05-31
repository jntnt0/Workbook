# Disable keyboards by hardware ID blocklist
# Companion script to Windows_Block_HID_Device_Registry.md
# Add hardware IDs to $blockList to prevent matched keyboards from functioning

$blockList = @(
    "HID\<hardware-id-1>",
    "HID\<hardware-id-2>",
    "HID\<hardware-id-3>"
)

Get-PnpDevice -Class Keyboard | ForEach-Object {
    $ids = (Get-PnpDeviceProperty -InstanceId $_.InstanceId -KeyName 'DEVPKEY_Device_HardwareIds' -ErrorAction SilentlyContinue).Data
    if ($null -ne $ids -and ($ids | ForEach-Object { $blockList -contains $_ } | Where-Object { $_ })) {
        Disable-PnpDevice -InstanceId $_.InstanceId -Confirm:$false -ErrorAction SilentlyContinue
        Write-Host "Disabled: $($_.FriendlyName) [$($_.InstanceId)]"
    } else {
        Write-Host "Kept enabled: $($_.FriendlyName) [$($_.InstanceId)]"
    }
}

# To remove registry policy blocks applied by the companion .reg file:
# reg delete "HKLM\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions\DenyDeviceIDs" /f
# reg delete "HKLM\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions" /f