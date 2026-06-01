@echo off
set "installerPath=\\<server>\<share>\<path-to-installer>.exe"
set "logFile=C:\Temp\Install.log"

if exist "%installerPath%" (
    echo Running installer from network path...
    "%installerPath%" /s /v"/qn /l*vx %logFile%" /quiet /norestart
) else (
    echo Network path not accessible: %installerPath%
    exit /b 1
)