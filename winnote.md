## Windows tricks

#### Settings shortcuts
    win x
    win i
    win pause

#### Win features
Control Panel\Programs\Programs and Features

#### Edit environment variables
    rundll32 sysdm.cpl,EditEnvironmentVariables

#### change permissions
    takeown /f folder_name /r /d y
    icacls folder_name /grant username_or_usergroup:F /t /q

#### change permissions from popup menu

###### InstallTakeOwnership.reg
    Windows Registry Editor Version 5.00
    
    [HKEY_CLASSES_ROOT\*\shell\runas]
    @="Take Ownership"
    "NoWorkingDirectory"=""
    
    [HKEY_CLASSES_ROOT\*\shell\runas\command]
    @="cmd.exe /c takeown /f \"%1\" && icacls \"%1\" /grant administrators:F"
    "IsolatedCommand"="cmd.exe /c takeown /f \"%1\" && icacls \"%1\" /grant administrators:F"
    
    [HKEY_CLASSES_ROOT\Directory\shell\runas]
    @="Take Ownership"
    "NoWorkingDirectory"=""
    
    [HKEY_CLASSES_ROOT\Directory\shell\runas\command]
    @="cmd.exe /c takeown /f \"%1\" /r /d y && icacls \"%1\" /grant administrators:F /t"
    "IsolatedCommand"="cmd.exe /c takeown /f \"%1\" /r /d y && icacls \"%1\" /grant administrators:F /t"

###### RemoveTakeOwnership.reg
    Windows Registry Editor Version 5.00

    [-HKEY_CLASSES_ROOT\*\shell\runas]
    
    [-HKEY_CLASSES_ROOT\Directory\shell\runas]


#### Wifi
    netsh wlan set hostednetwork mode=allow ssid=wbcchome key=qazwsxedc
    netsh wlan start hostednetwork
    netsh wlan stop hostednetwork
    netsh wlan show hostednetwork
    netsh wlan show drivers
    
#### Avast blocks wifi
    settings -> Active protection -> customize -> Policies -> Internet sharing mode -> OK

#### Win console kill
    query process
    tskill [id|name]

#### Disable driver signature enforcement
shift key while you click Restart  
Troubleshoot -> Advanced -> Startup Settings -> restart  
- Disable driver signature enforcement  
or
- Safe mode


#### REISUB Windows (not worked for me)
    regedit
    HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\i8042prt\Parameters
    HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\kbdhid\Parameters
    DWORD CrashOnCtrlScroll 1

Control Panel > System > Advanced >  Startup and Recovery > Settings > System failure > Automatically restart - uncheck  
MANUALLY_INITIATED_CRASH  
The computer has rebooted from a bugcheck: 0x000000e2  
A dump was saved in: C:\Windows\MEMORY.DMP

#### Diff-ext
http://diff-ext.sourceforge.net/downloads.shtml

#### Skype Commands
    /alertsoff
    /alertson
    /alertson [text]
    /me [text]
    wikimarkup: _italic_ *bold* ~crossed~
    (ladyvamp)

## Visual studio
ctrl k f - auto identity  
ctrl k d - all auto identity  

ctrl k c - comment  
ctrl k u - uncomment  

ctrl k s - suround with  

ctrl u - lower case  
ctrl sift u - upper case  

ctr '-'/ctrl sift '-' - go back/force  
f12 - goto definition  
ctrl alt f12 - goto declaration  
ctrl ; - find in solution  
ctrl alt space - show function parameters  

f5 - debug  
ctrl f5 - run  
sift f5 - stop debug  
f10 - step over  
f11 - step into  
sift f11 - step out  

#### Refactoring
ctrl r m - select method  
ctrl r e - incapsulate method  
ctrl r i - select interface  
ctrl r v - delete parameter  
ctrl r o - change parameter order

#### Errors
###### Error: MSVCRTD.lib(crtexew.obj) : error LNK2019: unresolved external symbol _WinMain  
    Properties -> Linker -> System -> SubSystem:  Console (/SUBSYSTEM:CONSOLE)

###### Missing: msvcp120.dll, msvcr120.dll
    Download from microsoft: Visual C++ Redistributable Packages for Visual Studio 2013

#### Additionally, you can try to map long base paths to an own drive letter. From a command line run
    subst P:\ C:\Your\Project\Folder\Is\Quite\Long
    
#### Apps
WinHEX  
Ninite
Spacesniffer  
Windirstat  
