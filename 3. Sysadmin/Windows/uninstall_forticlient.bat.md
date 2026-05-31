@echo off
rem Uninstall FortiClient

rem Find the FortiClient product code
setlocal enabledelayedexpansion
for /f "tokens=2 delims={}" %%i in ('wmic product where "name like 'FortiClient%%'" get identifyingnumber ^| find "{"') do (
    set "guid=%%i"
    set "guid=!guid:~0,-1!"
)

if defined guid (
    echo Uninstalling FortiClient with GUID: {%guid%}
    msiexec /x {%guid%} /quiet /norestart
    echo FortiClient uninstalled successfully.
) else (
    echo FortiClient is not installed on this machine.
)

endlocal
pause
