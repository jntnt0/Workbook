@echo off
setlocal

set DOMAIN=<your-domain>
set USERNAME=<admin-username>

net localgroup administrators | find /i "%DOMAIN%\%USERNAME%"

if not errorlevel 1 (
    echo %DOMAIN%\%USERNAME% is already a member of the local administrators group.
) else (
    net localgroup administrators /add %DOMAIN%\%USERNAME%
    if %errorlevel% neq 0 (
        echo Failed to add %DOMAIN%\%USERNAME% to local administrators group.
    ) else (
        echo %DOMAIN%\%USERNAME% successfully added to local administrators group.
    )
)

endlocal