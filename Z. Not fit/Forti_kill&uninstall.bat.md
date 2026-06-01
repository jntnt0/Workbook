|   |
|---|
|@echo off|
|rem Uninstall FortiClient|
||
|rem Kill the FortiClient process if it's running|
|taskkill /F /IM FortiClient.exe /T|
||
|rem Wait for a few seconds to ensure the process is terminated|
|timeout /t 5 /nobreak|
||
|rem Find the FortiClient product code|
|setlocal enabledelayedexpansion|
|for /f "tokens=2 delims={}" %%i in ('wmic product where "name like 'FortiClient%%'" get identifyingnumber ^\| find "{"') do (|
|set "guid=%%i"|
|set "guid=!guid:~0,-1!"|
|)|
||
|if defined guid (|
|echo Uninstalling FortiClient with GUID: {%guid%}|
|msiexec /x {%guid%} /quiet /norestart|
|echo FortiClient uninstalled successfully.|
|) else (|
|echo FortiClient is not installed on this machine.|
|)|
||
|endlocal|
||
|rem Exit the script|
|exit|