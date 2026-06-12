|                                                                |
| -------------------------------------------------------------- |
| @echo off                                                      |
|                                                                |
| rem Define generic paths (replace with your actual paths)      |
| set "installerPath=\\server\share\path\to\ChangeMe.exe"        |
| set "logFile=C:\Temp\InstallerLog.log"                         |
|                                                                |
| rem Check if the network path is accessible                    |
| if exist "%installerPath%" (                                   |
| echo Running installer from network path...                    |
| "%installerPath%" /s /v"/qn /l*vx %logFile%" /quiet /norestart |
| ) else (                                                       |
| echo Network path not accessible: %installerPath%              |
| exit /b 1                                                      |
| )                                                              |