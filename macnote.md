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
    cmd sft g - find prev

#### XCODE
###### Panels
    cmd 1 - pro nav
    cmd sift j - show file in pro nav
    cmd 0 - nav panel
    cmd alt 0 - util panel
    ctrl 1 - show related
    alt cliсk - open paralel
    cmd sift 2 - show devices

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

## DEV
#### Finder
###### Disable autocompleteng
    Control Panel\Clock, Language, and Region\Language\Advanced settings
###### Remove the Stripes
    defaults write com.apple.finder FXListViewStripes -bool FALSE
###### Put the Path Bar on Top
    defaults write com.apple.finder _FXShowPosixPathInTitle -bool YES
###### quicklookplugins.com 
    To install the plug-ins after downloading them, place them in ~/Library/QuickLook
    qlmanage -r
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

#### http://brew.sh
    https://git-scm.com/book/en/v1/Git-Basics-Tips-and-Tricks
    brew install bash-completion
    brew tap homebrew/completions
    https://github.com/Homebrew/homebrew-completions

#### Lock screen
Enable the fast user switching menu from the Users & Groups preference pane and then select Login Window… from the menu.  
Applications -> Automator -> Service -> Service receives no input in any application -> Run Shell Script:   /System/Library/CoreServices/Menu\ Extras/User.menu/Contents/Resources/CGSession -suspend  
-> Cmd-S to save -> Name -> saved in ~/Library/Services  
System Preferences -> Keyboard -> Shortcuts -> Services -> General  

    wget --password=<pass> --user=<user> -c '<link>'

    find . -name ".DS_Store" -exec rm {} \;
    find . -name ".DS_Store" -delete
    find . -name ".DS_Store" -print0 | xargs -0 rm
    find . -name "*.bak" -print0 | xargs -0 -I file mv file ~/old.files
    find . -type f -exec sed -i -e 's/foo/bar/g' {} \;

#### run app from terminal
    ./target.app/Contents/MacOS/target [args]

￼
#### Bugs
**Heisenbug** - disappear or alter its behaviour when one attempts to study it  
**Bohrbug** - is a "good, solid bug". do not change their behaviour and are relatively easily detected.  
**Mandelbug** - is a bug whose causes are so complex it defies repair, or makes its behaviour appear chaotic or even non-deterministic or bug that exhibits fractal behaviour by revealing more bugs  
**Schrödinbug** - manifests itself in running software after a programmer notices that the code should never have worked in the first place  
**Hindenbug** - is a bug with catastrophic behaviour  


    gnome, unity, gtk, enlightenment, kde
