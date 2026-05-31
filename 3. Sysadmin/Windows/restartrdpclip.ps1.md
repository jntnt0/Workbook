|   |
|---|
|$logFilePath = "C:\Logs\startup_script.log" # Change to your desired location|
|$logDirectory = Split-Path $logFilePath -Parent|
||
|# Create the log directory if it doesn't exist|
|if (!(Test-Path -Path $logDirectory)) {|
|try {|
|New-Item -ItemType Directory -Path $logDirectory \| Out-Null|
|}|
|catch {|
|Write-Error "Failed to create log directory: $($_.Exception.Message)"|
|return # Exit the script if directory creation fails|
|}|
|}|
||
|$timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"|
|$message = "$timestamp - rdpclip restart script started"|
|Add-Content -Path $logFilePath -Value $message|
||
|while ($true) {|
|try {|
|# Check if rdpclip.exe is running|
|$rdpclipProcess = Get-Process rdpclip -ErrorAction SilentlyContinue|
||
|if ($rdpclipProcess) {|
|# Stop the process|
|Stop-Process -Name rdpclip -Force -ErrorAction Stop|
|}|
||
|# Start the process|
|Start-Process -FilePath "rdpclip.exe" -ErrorAction Stop|
||
|$timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"|
|$message = "$timestamp - rdpclip.exe restarted successfully"|
|Add-Content -Path $logFilePath -Value $message|
|}|
|catch {|
|$timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"|
|$message = "$timestamp - Error restarting rdpclip.exe: $($_.Exception.Message)"|
|Add-Content -Path $logFilePath -Value $message|
|}|
||
|# Wait for 10 seconds|
|Start-Sleep -Seconds 10|