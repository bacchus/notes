## MacOS tricks

⌘ = Command  
⇧ = Shift  
⌥ = Option/Alt  
⌃ = Control  

#### Shortcuts
###### Screenshot
    cmd sift 3 - screenshot
    cmd sift 4 - area screenshot
    cmd sift 4 space - window screenshot

###### Safari
    cmd sift L - side bar
    cmd y - history

###### Finder
    cmd o - open
    cmd , - preferences
    space - quick look
    cmd i - get info window
    cmd backspaсe - delete to trash
    cmd sift n - new folder
    cmd d - duplicate file
    cmd ctrl up - open in new win

###### Folders
    cmd sift c - my computer
    cmd sift d - desktop
    cmd sift h - home
    cmd sift o - documents
    cmd sift l - downloads
    cmd sift u - utilities
    cmd sift g - go to folder

###### Navigating
    cmd [ - prev folder
    cmd ] - next folder
    cmd up - parent folder
    cmd down - child folder

###### Editor
    cmd ctrl d - lookup in dictionary
    cmd ; - spelling

###### Find
    cmd f - find
    cmd g - find next
    cmd sift g - find prev

#### XCODE
###### Panels
    cmd 1 - pro nav
    cmd sift j - show file in pro nav
    cmd 0 - nav panel
    cmd alt 0 - util panel
    ctrl 1 - show related
    alt cliсk - open paralel
    cmd sift 2 - show devices
    ctrl i - indent

###### Search
    cmd sift f - pro search
    cmd alt f - file find/replace
    cmd alt sift f - project find/replace

    ctrl 6 - jump bar
    cmd sift o - quick open
    cmd l - goto line
    cmd ctrl j - jump definition
    ctrl cmd up/down - switch cpp/header
    ctrl cmd left/right - forward/backward
    ctrl cmd e - rename variable
    cmd / - comment
    cmd alt [ ] - move line

###### Build
    cmd b - build
    cmd r - run
    cmd sift k - clean
    cmd . - stop

###### Debug
    f6 - step over
    f7 - step into
    cmd \ - breakpoint
    cmd y - enable/disable all breakpoints
    ctrl cmd y - play/pause debugger

## Git Meld
#### [yousseb](https://yousseb.github.io/meld/)
    [diff]
      tool = meld
    [difftool]
      prompt = false
    [difftool "meld"]
      trustExitCode = true
      cmd = open -W -a Meld --args \"$LOCAL\" \"$PWD/$REMOTE\"
    [merge]
      tool = meld
    [mergetool]
      prompt = false
    [mergetool "meld"]
      trustExitCode = true
      cmd = open -W -a Meld --args --auto-merge \"$PWD/$LOCAL\" \"$PWD/$BASE\" \"$PWD/$REMOTE\" --output=\"$PWD/$MERGED\"


## Admin
#### http://brew.sh
    https://git-scm.com/book/en/v1/Git-Basics-Tips-and-Tricks
    brew install bash-completion
    brew tap homebrew/completions
    https://github.com/Homebrew/homebrew-completions

#### Lock screen
Enable the fast user switching menu from the Users & Groups preference pane and then select Login Window… from the menu.  
Applications -> Automator -> Service -> Service receives no input in any application -> Run Shell Script:

    /System/Library/CoreServices/Menu\ Extras/User.menu/Contents/Resources/CGSession -suspend
-> Cmd-S to save -> Name "Lock screen" -> saved in ~/Library/Services  
System Preferences -> Keyboard -> Shortcuts -> Services -> General -> Shortcut "Ctrl+Cmd+L"

#### Change language by CapsLock
(previous was Sail)

    https://pqrs.org/latest/karabiner-elements-latest.dmg
add modifier -> CapsLock -> f19  
System Preferences -> Keyboard -> Shortcuts -> Input Sources -> Shortcut "CapsLock" (replaces by f19)

#### Finder
###### Disable autocompleteng
    Control Panel\Clock, Language, and Region\Language\Advanced settings
###### See Library folder in Users
    chflags nohidden ~/Library/
###### quicklookplugins.com 
    To install the plug-ins after downloading them, place them in ~/Library/QuickLook
    qlmanage -r
    

## DEV

### xcode disable double click
    prefs -> navigation -> double click navigation -> same as click

###### Slow xcode
    defaults write com.apple.dt.XCode IDEIndexDisable 1
    ./Applications/Xcode.app/Contents/PlugIns/
    Preferences Locations DerivedData -> DELETE
###### Fix CoreAudio missing
    xcode-select --install

    remove all references to CoreAudioKit  

    sudo xcode-select -s /path/to/xcode/Contents/Developer  
    fast fix  
    cd /usr/local/lib sudo ln -s ../../lib/libSystem.B.dylib libgcc_s.10.5.dylib  

###### Debug/Release
    Product -> Scheme -> Edit Scheme. Change the Build Configuration under the Info tab  

#### XCode Needs
- install ubuntu fonts and qt colouring
- Creating method stubs automatically
- jump def - decl
- find usages
- not to write non ascii symbols

#### XCode settings
    defaults ACTION DOMAIN KEY VALUE
    defaults write com.apple.Xcode PBXNumberOfParallelBuildSubtasks 4

#### That should restore Xcode to the state of its first launch
    defaults delete com.apple.dt.Xcode
    rm -rf $HOME/Library/Application Support/Developer/Shared/Xcode
    rm -rf $HOME/Library/Saved\ Application\ State/com.apple.dt.Xcode.savedState
    rm -rf $HOME/Library/Preferences/com.apple.dt.Xcode.*

#### XCode bash build (xcrun / xcodebuild / xcode-select)
    xcodebuild -list -project <pro-name>.xcodeproj
    xcodebuild -scheme "<scheme>" build
    xcodebuild -jobs 4

###### options
-list   lists the targets in a project  
-jobs NUMBER  
-showsdks   show system sdks  
-showBuildSettings  show project settings  
-version        of xcode  
-sdk SDK, -toolchain NAME   use for build  
-find-executable NAME, -find-library NAME   in system sdk  
The active developer directory can be set using `xcode-select`  
, or via the DEVELOPER_DIR


#### Qt with installed Xcode 7.2 
    QMAKE_MAC_SDK = macosx10.11

    wget --password=<pass> --user=<user> -c '<link>'

    find . -name ".DS_Store" -exec rm {} \;
    find . -name ".DS_Store" -delete
    find . -name ".DS_Store" -print0 | xargs -0 rm
    find . -name "*.bak" -print0 | xargs -0 -I file mv file ~/old.files
    find . -type f -exec sed -i -e 's/foo/bar/g' {} \;

#### nouchg Means the file can be changed (immutable bit cleared)
    chflags nouchg [files]


#### LLDB not work
    breakpoint set -M std::ostream::operator<<

#### run app from terminal
    ./target.app/Contents/MacOS/target [args]

#### arch binary info
    lipo -info <binary>
    https://gist.github.com/sponno/7228256
    
## Sardi
#### Ardis Icon Theme
#### http://erikdubois.be/sardi-flexible/
#### http://erikdubois.be/what-are-my-personal-settings-for-mac-osx-el-capitan/
    #1793D1  to #choosecolour
    find -name "*.svg" -type f -exec sed -i '/fill="#ffffff"/!s/fill="#1793D1"/fill="#choosecolour"/g' {}  \;
    find -name "*.svg" -type f -exec sed -i '/fill:#ffffff/!s/fill:#1793D1/fill:#choosecolour/g' {} \;

#### Preferences in Finder 
- Icons in the navigation in the Finder
- pressing the ALT button gives you an hidden folder Library 
- Order the favorites in the finder
- Finder toolbar – completely changed
- Security – allow all programs to install – every random source
- How to share a map between computers
- Log yourself in automatically
- autoinstall osx updates
- display the day in your time

#### Kill EPP&AMP

###### Tools
mdfind, which, locate, kextunload, kextstat  

    #try 
    sudo launchctl unload /Library/LaunchDaemons/...
    #else
    sudo rm /Library/LaunchDaemons/com.sourcefire.amp.daemon.plist
    
    sudo mv /usr/local/libexec/sourcefire /usr/local/libexec/sourcefire_
    sudo mv /Library/CoSoSys/EndpointProtector /Library/CoSoSys/EndpointProtector_

    sudo kextunload -b com.cososys.kext.EPPUsbHelper
    sudo kextunload -b com.cososys.driver.EPPDeviceController
    sudo kextunload -b com.cososys.eppclient.eppkauth
    sudo kextunload -b com.sourcefire.amp.fileop
    sudo kextunload -b com.sourcefire.amp.nke
    
#### Bugs
**Heisenbug** - disappear or alter its behaviour when one attempts to study it  
**Bohrbug** - is a "good, solid bug". do not change their behaviour and are relatively easily detected.  
**Mandelbug** - is a bug whose causes are so complex it defies repair, or makes its behaviour appear chaotic or even non-deterministic or bug that exhibits fractal behaviour by revealing more bugs  
**Schrödinbug** - manifests itself in running software after a programmer notices that the code should never have worked in the first place  
**Hindenbug** - is a bug with catastrophic behaviour  


    gnome, unity, gtk, enlightenment, kde
