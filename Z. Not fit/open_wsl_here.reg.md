
|   |
|---|
|Windows Registry Editor Version 5.00|
||
|[HKEY_CLASSES_ROOT\Directory\Background\shell\WSL]|
|@="Open WSL Here"|
|"Icon"="C:\\Windows\\System32\\wsl.exe"|
|"NoWorkingDirectory"=""|
||
|[HKEY_CLASSES_ROOT\Directory\Background\shell\WSL\command]|
|@="wsl.exe --cd \"%V\""|
||
|[HKEY_CLASSES_ROOT\Directory\shell\WSL]|
|@="Open WSL Here"|
|"Icon"="C:\\Windows\\System32\\wsl.exe"|
|"NoWorkingDirectory"=""|
||
|[HKEY_CLASSES_ROOT\Directory\shell\WSL\command]|
|@="wsl.exe --cd \"%1\""|