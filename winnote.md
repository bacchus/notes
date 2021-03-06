## Windows tricks

#### Settings shortcuts
    win x
    win i
    win pause
    win g - video record

#### Win features
Control Panel\Programs\Programs and Features

#### Admin tools
    gpedit.msc
    optionalfeatures.exe
    check the %APPDATA% folder
    
#### Start/stop services
    sc stop "AppXSvc"
    sc config "AppXSvc" start=disabled
    sc delete "AppXSvc"

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


#### REISUB Windows
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

//==============================================================================
#### Errors
###### Error: MSVCRTD.lib(crtexew.obj) : error LNK2019: unresolved external symbol _WinMain  
    Properties -> Linker -> System -> SubSystem:  Console (/SUBSYSTEM:CONSOLE)

###### Missing: msvcp120.dll, msvcr120.dll
    Download from microsoft: Visual C++ Redistributable Packages for Visual Studio 2013

#### Additionally, you can try to map long base paths to an own drive letter. From a command line run
    subst P:\ C:\Your\Project\Folder\Is\Quite\Long

#### side-by-side configuration is incorrect
    sxstrace trace -logfile:sxs.log
    sxstrace parse -logfile:sxs.log -outfile:sxstrace.txt

#### incompatibles in Debug/Release libs check this:
    _ITERATOR_DEBUG_LEVEL

//==============================================================================
#### virtual desktop (VD)
    win+ctrl+D: new VD
    win+ctrl+F4: close current VD
    win+ctrl+left/right: switch between VDs

#### Apps
WinHEX  
Ninite
Spacesniffer  
Windirstat  

//==============================================================================
## visual-studio-hang

#### 1. disable the  "Hosting Process"
- Open an executable project in Visual Studio. Projects that do not
produce executables (for example, class library or service projects) do not have this option.
- On the Project menu, click Properties.
- Click the Debug tab.
- Clear the Enable the Visual Studio hosting process check box.


#### 2. suspended ReSharper
Options -> ReSharper -> General ->Suspend Now


#### 3. Disable "Allow this precompiled site to be updatable"
- Open Site/Solution
- Right click and view Property Pages
- Go to MSBuild Options
- Uncheck "Allow this precompiled site to be updatable"


#### 4. Debug symbols

DB files are generated if using /Zi or /ZI (Produce PDB Information) compiler  
and /DEBUG (Generate Debug Info) linker

###### PDB files contain the following information:
- Public symbols (typically all functions, static and global variables)
- A list of object files that are responsible for sections of code in the executable
- Frame pointer optimization information (FPO)
- Name and type information for local variables and data structures
- Source file and line number information

###### Developer Links
    https://www.microsoft.com/whdc/devtools/debugging/default.mspx
    https://developer.microsoft.com/en-us/windows/downloads/windows-10-sdk
    https://developer.microsoft.com/en-us/windows/hardware/windows-driver-kit
    https://developer.microsoft.com/en-us/windows/hardware/download-symbolsw


###### Check if a given DLL or .exe file and PDB in the same folder match
/s option tells symchk to look for symbols only in the current folder, and not to look in any symbol servers

    "c:\Program Files\Debugging Tools for Windows\symchk" testing.dll /s
    -> SYMCHK: FAILED files = 0
    -> SYMCHK: PASSED + IGNORED files = 1

###### Check if all the DLLs and executable files in a set of folders have matching PDBs
/r - recursively

    "c:\Program Files\Debugging Tools for Windows\symchk" *.* /r

###### show the symbol paths that are searched
    DUMPBIN /PDBPATH:VERBOSE filename.exe

###### Microsoft Symbol Server: _NT_SYMBOL_PATH
Options -> Debugging -> Symbols -> 
    
    http://msdl.microsoft.com/download/symbols
    srv*c:\symbols*https://msdl.microsoft.com/download/symbols

local file share on \\mainserver\symbols:

    Srv*c:\symbols*\\mainserver\symbols*https://msdl.microsoft.com/download/symbols

###### Getting Symbols Manually
for lib: d3dx9_30.dll:

    "c:\Program Files\Debugging Tools for Windows\symchk" c:\Windows\System32\d3dx9_30.dll /oc \.

//==============================================================================
###### Kill EPP&AMP
Windows killed services: MSClickToRun, Superfetch

//==============================================================================
## Skype block ads
[forum](https://community.skype.com/t5/Windows-desktop-client/Block-Skype-Ads-Quick-and-Easy/td-p/3222434/page/7)  
[github](https://gist.github.com/eyecatchup/ba7dc7a50d90cbf6377d)  
add it to host file: system32/drivers/etc/host

## 32 vs 64
64: %ProgramFiles%
64: %ProgramW6432%
32: %ProgramFiles(x86)%

64: %SystemRoot%\System32
32: %SystemRoot%\SysWOW64
16: %SystemRoot%\System

64: HKLM\Software
32: HKLM\Software\Wow6432Node

PowerShell: Set-ExecutionPolicy RemoteSigned
