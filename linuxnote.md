---
## Regex Cheat Sheet
Characters | - | Anchors | -
----------- | -----------                       | ----------- | -----------
.           | any character except newline      | ^abc$       | start / end of the string
\w \d \s    | word, digit, whitespace           | \b          | word boundary
\W \D \S    | not word, digit, whitespace       | **Escaped** | **-**
[abc]       | any of a, b, or c                 | \\. \\* \\\ | escaped special characters
[^abc]      | not a, b, or c                    | \t \n \r    | tab, linefeed, carriage return
[a-g]       | character between a & g           | \u00A9      | unicode escaped ©
**Groups** | **-** | **Quantifiers** | **-**
(abc)       | capture group                     | a* a+ a?    | 0 or more, 1 or more, 0 or 1
\1          | backreference to group #1         | a{5} a{2,}  | exactly five, two or more
(?:abc)     | non-capturing group               | a{1,3}      | between one & three
(?=abc)     | positive lookahead                | a+? a{2,}?  | match as few as possible
(?!abc)     | negative lookahead                | ab\|cd      | match ab or cd

---
## Googling
    ~ [synonym] (~best films -best)
    - [except] (-.ua)
    * [any] (best editor * image)
    | [or] (buy house | car)
    "[exact]"
    define: [word definition]
    site: [addres]
    links: [to site]
    filetype: [extension]
    cached: [cached pages]
    time [city]
    weather [city]
    1 kg in pounds; 1 usd in uah
    translate [word] into [lang]
    2+2=
    google.com/ncr - really google.com
    google.com/advanced_search - advanced

---
## AdBlock rules
    r-click -> menu -> inspect -> adblock -> block elements
#### rules:
    http://example.com/ads/*            // all matching
    ||example.com/banner.gif            // beginning of the domain name
    ^                                   // placeholder for a single separator character: [:/?&=]
    ! comment                           // used for scripts in downloaded filter lists, not in custom filters
    */ads/*$script,match-case           // '*/ads/*' - filter, '$script,match-case' - options
    /banner\d+/                         // regex: matches 'banner123', but not 'banners'; low performance

    ##div.textad                        // <div class="textad">
    ##div#sponsorad                     // <div id="sponsorad">
    ##*#sponsorad                       // also works
    ##textad                            // <textad>

    ##*.sponsorad                       // all sites
    example.com##*.sponsorad            // onlly on example.com
    ~example.com##*.sponsorad           // exception on example.com

    ##table[width="80%"]                // tables with width attribute set to 80%
    ##div[title*="adv"]                 // div elements with title containing "adv"
    ##div[title^="adv"][title$="ert"]   // starting "adv" and ending "ert"
    table[width="80%"][bgcolor="white"] // tables with width 80% and bgcolor white

#### exceptions
    http://example.com/advice.html      // exception rule @@advice
    @@||example.com^$document           // adblock entirely disabled on this page
    ~example.com##*.sponsorad           // exception on example.com
     example.com#@#div.textad           // exceptions

---
## tampermonkey
    https://greasyfork.org/en/scripts/22210-facebook-unsponsored
    https://greasyfork.org/en/scripts/13807-youtubedefaultspeed
    https://greasyfork.org/en/scripts/34651-disable-youtube-autoplay
    https://greasyfork.org/en/scripts/32954-automatic-material-dark-mode-for-youtube

#### facebook-unsponsored
    -- leave only en

    ++ 'en':        ['Suggested Post', 'Recommended fer ye eye', 'Recommended for you']

    +++
        }, {
            // Video
            'selector': [
                '.fbUserStory > div > div > div > div > div > div > div > div > div > h5 > span > span > a',
                'div > div > div > div > div > div > div > div > div > div > h5 > span > span > a',
                'div > div > div > div > div > div > div > div > div > div > div > h5 > span > span > a',
                'div > div > div > div > div > div > div > div > div > div > div > div > h5 > span > span > a'
            ],
            'content': {
                'en':        ['video', 'a photo and a video', 'post']
            }
    +++

#### youtubedefaultspeed
    ++ var RATE_OPTIONS = ["1", "1.25", "1.5"];

    -- var head = document.getElementById("yt-masthead-user");
    ++ var head = document.getElementById("container");

    ++ label.style.color = "red";

    -- head.appendChild(form);
    +++
        var end_elm = document.getElementById("end");
        form.style = "margin-right: 10px";
        head.insertBefore(form, end_elm);
    +++


---
## Disable ads
#### utorrent
    utorrent->options->prefs->advanced: filter
    offers.left_rail_enabled = false
    offers.sponsored_torrent_offer_enabled = false

#### skype
    %appdata%/Skype/<skype-username>/config.xml
    delete all <AdvertPlaceholder>

---
## Chrome
    chromium
    chromium-codecs-ffmpeg-extra

#### Chrome must plugins
- my tampermonkey scripts
- Theme: James White
- ~~Webcam Toy app~~
- SetupVPN
- Pinterest Save Button
- Octotree
- Google Arts & Culture; Google Mail Checker; Google Search by Image
- Chrome Apps & Extensions Developer Tool
- Adblock Plus; AdBlock+ Element Hiding Helper; ~~CatBlock~~; Ghostery; AdNauseam
- uBlock Origin; Tampermonkey; Simple URL Extender
- grammarly
- Stylish - Custom themes for any website

#### Old bookmarks man - disble Material Design
    chrome://flags/#enable-md-bookmarks

#### Enable WebGL
    chrome://flags - Override software rendering list - Enable

#### Chromium libpepflashplayer.so causing excessive disk writes
    shift-esc = chrome task manager
    iotop
    chrome://flags/Enable Offline Auto-Reload Mode
    chrome:plugins disable
    - chromoting view (no need accses other computers)
    - pdf viewer (let download pdf)
    - flash (uncheck always allowed to run)

    uninstall pepperflash
    sudo apt-get remove flashplugin-installer
    sudo update-pepperflashplugin-nonfree --uninstall
    /etc/chromium/default config to also add "--disk-cache-dir=/tmp"
    http://peter.sh/experiments/chromium-command-line-switches


## viber
settings: ~/.ViberPC/{your-phone-number}/viber.db  
edit with sqliteman  

Paste below query to query edit area  
replaces text Documents/ViberDownloads with .viberdownloads  
in PayloadPath field of messages table

    Update messages set PayloadPath = replace(PayloadPath, "Documents/ViberDownloads", ".viberdownloads") 
        where PayloadPath is not null and PayloadPath <> '';

    Click Run(F9)

## split flac
    sudo apt install cuetools shntool flac wavpack                              # install tools
    cuebreakpoints *.cue | shnsplit -o flac -f *.cue  -t "%n - %t" *.flac ;     # split
    cuetag *.cue split-track*.flac ;                                            # add tags
    shntool conv -o flac *.ape                                                  # convert ape to flac
    iconv -f Windows-1251 Windows_file.txt > UTF8_file.txt                      # convert to utf8

## ln -s args order
    cp    existing_thing new_thing
    ln -s existing_thing new_thing

---
## Find in files

### ag silversearch
    sudo apt-get install silversearcher-ag
    gedit .agignore
    ag <text-to-find>

### grep
    grep -rl ‘text’ directory/*
    grep -r word *
    grep -rnw 'directory' -e "pattern"
    -iwrvn -locb

-r recursive; -n line number; -w whole word  
-i ignore-case; -v non-matching; -l print files matches  
-B before; -A after; -C context  
--include-dir=dir0; --exclude-dir={dir1,dir2,.dst}  
--include=\.{c,h}; --exclude=*.o

### find in pdf
    find . -iname '*.pdf' -exec pdfgrep <patern> {} +
    pdfgrep -irn <text>
    
### fast rm
    rsync -a --delete empty/ <dir>/

## ubuntu 18.04 problems
+ [x] turn off display changes resoution: xset dpms force off
+ [x] set lang shortcut to caps: use gnome-tweak-tool
+ [x] guake to f1, disable system f1, add to startup: just in guake set f1, startapps
+ [x]   if not: compizconfig -> comands -> guake -t -> bind to f1
+ [x] mount tatra on boot: use Disks  
    /etc/fstab; blkid
    UUID=225cad76-baf5-49f9-b49d-e7eeb113aae8 /media/bacchus/alcor \
    ext4 errors=remount-ro,auto,user,exec,rw 0 0
+ [x] enable wifi on boot: /etc/NetworkManager/system-connections/Hotspot: autoconnect=true
+ [x] test keys: xev
+ [x] youtube freze: flags - Override software rendering list - true
+ [x] kidle_inject: find smw here
+ [x] unplug tatra: udisksctl power-off -b /dev/sdc
- [ ] when delete, don't ask to undo: I'm sorry Dave, I'm afraid I can't do that.
+ [x] (16.04) wallpaper is not changing: gsettings set org.gnome.settings-daemon.plugins.background active true

---
## TODO:
---
## Ubuntu first install
    sudo apt-get clean
    sudo apt-get update
    sudo apt-get upgrade
    
    sudo apt-get purge <pack> # remove pack
    
#### tools
- ~~nautilus-open-terminal~~ - present by default
- compizconfig-settings-manager - **dconf-editor** instead, but I still use it
- ubuntu-restricted-extras, chromium-codecs-ffmpeg-extra - extras
- meld, git, guake, qt, cppcheck, cppclean, astyle, ag(the silver searcher), ~~necessitas~~ - dev tools
- skype, gimp, gpick, chromium, ~~stellarium~~, vlc, cheese-webcam, guvcview - madia tools
- ~~krita: sudo add-apt-repository ppa:kubuntu-ppa/backports; sudo apt-get install krita~~ - image editor (gimp beter works for me)
- git-recall
- jupyter notebook vs pycharm
    
#### Fixes
- apport crash report
- use linux_settings

## Nautilus
hotkey | what does  
---- | ----
ctrl-pgup(pgdown)   | prev/next tab  
alt-1(2,3,4)        | move to 1/2,3,4 tab  
alt-home            | home dir  
alt-up              | parent dir  
alt-left(right)     | history dir move  
alt-enter           | properties  
ctrl-1(2,3)         | view list, compact  
ctrl-shift-n        | new dir  
ctrl-m              | make link

#### Hotkeys
super-n         open app in launcher  
super-shift-n   opens app in launcher if already runing  
super-t         trash  
alt-f1 	        left toolbar  
alt-f2		run
ctrl-s/ctrl-q	bash pause/resume

#### Open from terminal
~~gnome-open \<filename\>~~

    xdg-open <filename>
    gvfs-open <filename>


#### Unmount tatra
    udisks --unmount /dev/sdb1
    udisks --detach /dev/sdb

#### Backup: create file list m_Photos.txt
    du -ah > m_Photos1.txt
    sort -k 2,2 m_Photos1.txt > m_Photos.txt
    rm m_Photos1.txt

#### rsync to android
    cd /run/user/1000/gvfs/mtp:host=Xiaomi_MI_6_49fad438/Internal shared storage
    rsync --verbose --progress --omit-dir-times --no-perms --recursive --inplace --ignore-existing \
    <from>/ ./<to>/

#### Dropbox encfs
    sudo apt-get install encfs
    #sudo addgroup <username> fuse
    encfs ~/Dropbox/.encrypted ~/private
    sudo install ~/gnome-encfs /usr/local/bin
    gnome-encfs -a ~/Dropbox/.encrypted ~/private
    fusermount -u ~/private

#### Tweeter unfolow all
Open "Following" page on Twitter https://twitter.com/following  
scroll down till all following accounts showed  
open developer console(CTRL+SHIFT+C)  
$('.button-text.unfollow-text').trigger('click');

#### Youtube: search user comments
    "user" * "weeks ago" site:youtube.com/watch

#### Styling
    brew install astyle
    curl https://raw.githubusercontent.com/sept-en/JUCE-utilities/master/juce_astyle.options > ~/.juce_astyle.options
    echo 'alias jucestyle="astyle --options=\"${HOME}/.juce_astyle.options\""' >> ~/.bash_profile
    . ~/.bash_profile

#### Bash cpp
    cat >> intro.cpp
    #include <iostream>
    int main() { std::cout<<"hello kittie\n"; }
    ^D
    make intro
    ./intro 

#### Safe fixed array param
    inline mat3_t::mat3_t( float (&src)[3][3] ) {
        memcpy( mat, src, sizeof(src ) );
    }

#### Alt mouse move window
    sudo apt-get install dconf-tools
dconf-editor → org → gnome → desktop → wm → preferences → mouse-button-modifier

#### Change pdf encoding
    gs -sDEVICE=pdfwrite -dNOPAUSE -dBATCH -dSAFER  -sOutputFile=output.pdf input.pdf  
int pdf viewer: file\save as\format\settings\UTF-8\save

#### ffmpeg
######## Video to gif
    ffmpeg -y -t 10 -i SampleVideo_1080x720_10mb.mp4 \
    -vf fps=10,scale=320:-1:flags=lanczos,palettegen gifPallet.png
	
    ffmpeg -y -t 10 -i SampleVideo_1080x720_10mb.mp4 -i gifPallet.png -filter_complex \
	"fps=10,scale=320:-1:flags=lanczos[x];[x][1:v]paletteuse" output2.gif
	
#### Restart ui
    sudo service lightdm restart

#### Restart network
    sudo service network-manager restart

#### Disable touchpad (in 14.04 from settings)
    xinput list | grep Touch
    xinput set-prop 13 "Device Enabled" 0

## disable laptop keyboard
    xinput list
    > AT Translated Set 2 keyboard  id=17	[slave  keyboard (3)]
    xinput float 17         # disable
    xinput reattach 17 3    # enable

#### Tar
c - create, x - extract  
z - gzip, j - bzip2  
f - file, v - verbose

    tar -czvf <file.tar.gz> <files>
    tar -xjvf <file.tar.bz2>
    tar cf <file.tar> <files>

    gzip file
    gzip -d file.gz

###### Encrypt archive
    tar cz <file_name> | openssl enc -aes-256-cbc -e > out.tar.gz.enc
    openssl aes-256-cbc -d -in out.tar.gz | tar xz

#### See progress bar: md5sum filename
    pv filename | md5sum

#### apt fix
    sudo apt-get clean
    sudo apt-get autoremove
    sudo apt-get autoclean
    sudo apt-get -f install
    sudo dpkg --configure -a

#### ☠ Use with caution
    sudo apt-get --force-yes install <pkgname>
    sudo apt-get --force-yes remove <pkgname>

#### dpkg
    sudo dpkg -i <pkgname> # Install
    sudo dpkg -r <pkgname> # Remove
    sudo dpkg -P <pkgname> # Purge

#### Hardware info
    lspci - VGA
    lsusb
    demidecode
    cat /proc/meminfo
    cat /proc/cpuinfo
    cat /proc/partitions
    ls -l /dev/disk/by-uuid/

#### System info
    uname -r                # kernel version
    cat /etc/issue.net      # ubuntu version
    date                    # cur date and time
    uptime                  # time from boot
    w                       # users online
    whoami                  # login name
    df                      # discs usage  
    du                      # weight of cur dir  
    du -sh <dir>            # weight of <dir> in human readable  
    free                    # memory usage and swap  
    whereis <app>           # location of <app>  
    which <app>             # which <app> will be run

#### Commands
    passwd                  # change pasword  
    locate file             # find file  
    dpkg -i pkg.deb         # install package  
    gksu                    # sudo with graphical ui  
    crontab -e              # edit cron tasks

#### Desktop message
    notify-send "Title" "Text"

#### crash com.ubuntu.apport-support-gtk-root
    sudo gedit /etc/default/apport
    # enabled=0
    sudo rm /var/crash/*    
    sudo service apport restart

#### Conky
    sudo apt-get remove indicator-appmenu
    xdg-open <open-def-app-file>
    ~/.gconf/apps/gnome-settings/gedit/%gconf.xml

#### brightness gamma
    xrandr --output LVDS1 --brightness 0.8
    xgamma -gamma 0.5

#### sticky-bit setuid-setgid
    chmod +t sticky-bit
    chmod ug+s setuid-setgid

#### Network
    ping host - ping host  
    whois domain - whois info  
    dig domain - dns info  
    dig -x host - reverse search host  
    wget file - load file  
    nmap -v -A scanme.nmap.org - scan  
    /etc/resolv.conf - dns-servers list  
    /etc/services - standart well known ports

    wget -r --no-parent -k -p <web-page>

#### Other
/etc/apt/sources.list - repository list  
/var/cache/apt/archives - archive of instaled packages  
/etc/usplash.conf - usplash conf  
/etc/rc.local - your script on boot  
/boot/grub/menu.lst - grub conf  
/ets/fstab - mounted devices conf  
/var/log - logs  

#### Grub customizer
!!!not tested!!!  
    sudo add-apt-repository ppa:danielrichter2007/grub-customizer
    sudo apt-get update
    sudo apt-get install grub-customizer
    
#### Boot repair
!!!not tested!!!  
    sudo add-apt-repository ppa:yannubuntu/boot-repair
    sudo apt-get update
    sudo apt-get install boot-repair

#### Youtube
0-9         parts of video  
space       play pause  
<- ->       fwd back 5 sec  
^ v	        +- vol  
f           fullscreen  
esc         exit fullscreen  
tab         navigate  
    youtube CTRL + U. CTRL + F. og:image. the link.  

#### Android Studio & ibus-daemon
1: Force ibus in synchronous mode, Do this preferably before starting Studio  
    IBUS_ENABLE_SYNC_MODE=1 ibus-daemon -xrd

2: Disable IBus input in Studio, This will only disable input methods for Studio  
    XMODIFIERS= ./bin/studio.sh

Settings -> Language Support -> Keyboard input method system  
change from 'IBus' to 'None'  
and now Ctrl+Shift+u works again

You can increase the value of idea.cycle.buffer.size=1024  
in property file android-studio\bin\idea.properties

#### GPS
В Крыму находится станция IGS с кодом CRAO.  
igs.org/igsnetwork/network_by_site.php?site=crao  
Кто угодно может скачать RINEX-файлы и прогнать через rtklib

#### PS1
PS1="$PS1\n--> "  
PROMPT_DIRTRIM=2  
PS1='${debian_chroot:+($debian_chroot)}\[\033[00;32m\]\u\[\033[01m\]@\[\033[00;36m\]\h\[\033[01m\]:\[\033[00;35m\]\w\[\033[00m\]\[\033[01;33m\]`git branch 2>/dev/null|cut -f2 -d\* -s`\[\033[00m\]\$ '


#### Find online printers
inurl:hp/device/this.LCDispatcher?nav=hp.Print  
https://www.google.ru/search?hl=en&newwindow=1&tbo=d&site=&source=hp&q=inurl%3Ahp%2Fdevice%2Fthis.LCDispatcher%3Fnav%3Dhp.Print&oq=inurl%3Ahp%2Fdevice%2Fthis.LCDispatcher%3Fnav%3Dhp.Print&gs_l=hp.3...634.634.0.1065.1.1.0.0.0.0.43.43.1.1.0.les%3B..0.0...1c.1.z__9Aio_1J0  
http://www.google.com/search?rls=en&q=intitle:start+inurl:cgistart  
http://www.google.com/search?q=inurl:CgiStart?page=Single  
http://www.google.com/search?q=inurl:indexFrame.shtml?newstyle=Quad

//------------------------------------------------------------------------------
## Perforce

#### Integrate perforce to QtCreator

    p4 command: /Users/<user>/bin/p4  
    P4PORT=<ip>:<port>  
    P4USER=<user-name>  
    P4CLIENT=<workspace>

    /usr/local/bin/p4 rename <from_name> <to_name>
    p4 changes -u $P4USER @2016/06/08,@now -m 5

//------------------------------------------------------------------------------
## Certificates
    sudo apt-get install libnss3-tools
    sudo cp <cer-folder>\ *.cer /usr/share/ca-certificates/
    sudo dpkg-reconfigure ca-certificates
    certutil -d sql:$HOME/.pki/nssdb -A -t "C,," -n "<name>" -i <cer-folder>\ SOME.cer

//------------------------------------------------------------------------------
## Killing process
#### Processes
    ps - show all current procs  
    top - show running procs  
    bg - list stoped and bg tasks; continue stoped task  
    fg - move last task to foreground  
    fg n - move n task to foreground

#### Getting id
    ps aux | grep [name]
    pidof [name]

#### Killing
    kill [id]
    kill -9 [id]
    killall -9 [name]
    pkill [name]

#### Kill with mouse
alt+f2 xkill

    sudo gedit /etc/sysctl.d/10-magic-sysrq.conf

change 176 to 244  
if "the system is BUSIER than it should be"  
(slowly, with a few seconds between each)  
alt+SysReq(Print Screen), type REISUB  
alt+SysReq+F - Kernel will kill the mostly "expensive" process each time  
alt+SysReq+K - for console  

ctrl+alt+backspace - kill x-server

//------------------------------------------------------------------------------
# Oneliners
    yes 'c=(╱ ╲);printf ${c[RANDOM%2]}'|bash
    
    getfattr -dRhm- /home 2>/dev/null >./getfattr.log
    setfattr -hx name /path/to/comrade/major.png


#### Show number of CPU cores
    grep -c ^processor /proc/cpuinfo  

#### Show first 5 errors
    <make> 2>&1|grep error|head -5|tee log.txt

#### Display information about the contents of ELF format files
    readelf <option(s)> elf-file(s)
      -a --all               Equivalent to: -h -l -S -s -r -d -V -A -I

#### Show lib export symbols info
    nm -D <lib.a>
    
#### Decode/translate c++ symbols to readable format
    c++filt __ZNK4juce19AudioProcessorGraph21AudioGraphIOProcessor8isOutputEv
    > juce::AudioProcessorGraph::AudioGraphIOProcessor::isOutput() const

#### Check architecture
    file <lib>
    
#### eclipse crash: add following lines to eclipse.ini
    -Dorg.eclipse.swt.browser.DefaultType=mozilla  
    -Dorg.eclipse.swt.browser.XULRunnerPath=path_to_xullrunner

#### Physical Source Lines of Code (SLOC)
    sloccount directoryname

#### Building android project on linux project root:
    <PathToSDK>/tools/android update project --name <ProjectName> --target "android-15" --path .
    ndk-build
    ant debug

#### Create qt project from existing source
    qmake -project

#### Qt
instead

    pkg-config --cflags --libs opencv

in pro

    CONFIG += link_pkgconfig
    PKGCONFIG += opencv
    
custom makefile name

    -o qMakefile  
    -f qMakefile

    locate libQt5Script.so

    LD_LIBRARY_PATH=\`pwd\` ./<executable>

    gcc -Wall -fno-stack-protector stacksmash.c -o stacksmash
    
    std::cout << "\n         (__)\n         (oo)\n   /------\\/\n  / |    ||\n *  /\\---/\\\n    ~~   ~~\n\n";
    
    struct T{U u;T(U u):u(std::move(u)){}};

//------------------------------------------------------------------------------
## Bash

    # exit status
    echo $?

#### Scroll up and down the list. 'q' to quit
    ls | less
    more
    head <filename>

#### Rename
    rename 'y/A-Z/a-z/' *
    rename 's/.df/.txt/g' *.df
    rename 's/aa/12/g' *
    rename 's/(\.)(?!png)/_/g' * --dry-run # all '.' to '_' except in '.png'
    sed -i 's/foo/bar/g' *.txt
    mv filename.{df,txt}

#### delimeters - in: vmlinuz-3.13.0-40-generic; out: 3.13.0-40
    cut -d '-' -f2,3

#### mtr - traceroute & ping
    mtr [hostname]

#### jot - generates text
    jot [count] [begin with]
#### random
    jot -r [count] [min] [max]

    df - disk free  
    cal - calendar  
    tac - reverse cat  
    w - who  
    factor - factors num  
    nl - numering lines

    pushd/popd

    history, then !*num-in-hist*

//------------------------------------------------------------------------------
## FIND
#### Apply command to find files
    find . -type f -exec dos2unix {} +
    
    find *directory* -type f | xargs grep -rl '*text*'
    find *directory* -type f -exec grep -l ‘*text*’ {} +
    find /path -name "name" -type d

#### Find and replace in files foo -> bar
    find . -name "*.php" -print | xargs sed -i 's/foo/bar/g'

#### Move all the files and directories to the parent directory
    find . -maxdepth 1 -exec mv {} .. \;
    find from/ -name *.orig -exec mv {} to/ \;

#### List all files inside directories
    ls -alr *
//------------------------------------------------------------------------------
## GREP

#### EXPRESSIONS
- ^ - begin line  
- $ - end line  
- grep -c "^$" - count empty lines

- '1*' matches zero or more '1'  
- . any symb  
- \\+ one or more of prev symb  
- \\? zero or one  
- \\b words boundary

    grep "[0-9]\\+ times"  
    grep -i "^[^aeiou]" - starts with non matching aeiou  
    grep -v "^\\#\\|^\\/\\/" - do not show comments

[:digit:] [:alnum:] [:alpha:] [:blank:]

{m,n} {m} {m,} {,n}

grep  "^[0-9]\\{1,5\\}$" - [0-9] number 1-5 times

grep -e '^\\(abc\\)\\1$' - back references (\\n): ^; \\(abc\\); \\1; $;

OR

    grep 'pattern1\|pattern2'
    grep -E 'pattern1|pattern2'
    grep -e pattern1 -e pattern2

AND

    grep -E 'pattern1.*pattern2'
    grep -E 'pattern1.*pattern2|pattern2.*pattern1'
    grep 'pattern1' | grep 'pattern2'

NOT

    grep -v 'pattern1'

//------------------------------------------------------------------------------
#### nl - number lines
-i5 increase cnt, -s. add string '.' after cnt, -w2     column for cnt  
-b style: a all lines,  t nonempty lines, n no lines, pREG lines for regexp (-bpA - only with A)  
-n format: ln left justified no leading zeros, rn right -same-,  rz right j lead z  

    nl file.txt

#### wc - coun lines, words, bytes

-w words, -L length of longest line, -l new lines, -c bytes

    wc file.txt

#### lines words bytes
5     5     41     sort.txt



//------------------------------------------------------------------------------
## SED
#### a\ append, i\ insert, c\ replace, = print line number  
#### ADDRESS line number, PATTERN reg, $ end of file

    sed 'ADDRESS a\  
        Line which you want to append' filename

    sed '/PATTERN/ a\  
        Line which you want to append' filename

    sed '$ a\  
        Line which you want to append to end of file' filename

    sed '/PATTERN/=' filename

#### Find and replace in files foo -> bar
    sed -i 's/foo/bar/g' *.txt
    find . -name "*.txt" -print | xargs sed -i 's/foo/bar/g'
    

#### multiply lines
    sed -n '/PATTERN/,/PATTERN/ {
    =
    p
    }' filename

#### total lines
    sed -n '$=' thegeekstuff.txt

//------------------------------------------------------------------------------
## SSH tricks

    ssh [-p port] user@host - connect to host as user on port  
    ssh-copy-id user@host   - add your key to host  
    ssh -nNTL 9999:127.0.0.1:1234 secret_dev - 	port forwarding
    -v - verbose

#### Generate and set key
    cd ~/.ssh
    ssh-keygen -t rsa
    ssh-add ~/.ssh/id_rsa
    sftp bacchus@106.125.11.22
    mkdir .ssh
    cd .ssh
    put id_rsa.pub
    exit
    ssh bacchus@106.125.11.22
    cd .ssh
    cat id_rsa.pub > authorized_keys
    exit

#### mount remote ssh dir: sshfs *ssh-dir* *local-dir*
    sshfs bacchus@106.125.11.22:/share/dir ~/share/dir

#### reverse ssh tuneling
    # dest(192.168.20.55) <- NAT <- src(138.47.99.99) [<- Bob's server]
    # from dest
    ssh -R 19999:localhost:22 sourceuser@138.47.99.99
    # from src
    ssh localhost -p 19999
    # from Bob's server
    ssh sourceuser@138.47.99.99
    ssh localhost -p 19999

//------------------------------------------------------------------------------
## Java
    \#!/bin/sh  
    \# java install  
    set -e  
    sudo sh -c "echo 'deb http://www.duinsoft.nl/pkg debs all' >> /etc/apt/sources.list"  
    sudo apt-get update  
    sudo apt-key adv --keyserver keys.gnupg.net --recv-keys 5CB26B26  
    sudo apt-get update  
    sudo apt-get install update-sun-jre  

#### icedtea javaws jnlp
    sudo apt-get install icedtea-netx
    #sudo apt-get install icedtea-plugin
    javaws filename.jnlp

## Installing Oracle Java JDK
    http://www.oracle.com/technetwork/java/javase/downloads/index.html  
    download jdk-7u21-linux-i586.tar.gz  
    tar -xf jdk-7u21-linux-i586.tar.gz  
    sudo mv jdk1.7.0_21 /usr/lib/jvm/  
    sudo update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/jdk1.7.0_21/bin/java" 1  
    sudo update-alternatives --config java  
    java -version  
    JAVA_HOME=/usr/lib/jvm/jdk1.7.0_21/  
    export JAVA_HOME

#### choose default java version in system
    sudo update-alternatives --config java

#### if key error occures
    sudo apt-key update
#### or manualy if error received: NO_PUBKEY *key-number*
    sudo apt-key adv --recv-key --keyserver keyserver.ubuntu.com *key-number*
#### after this try
    sudo apt-get update

#### helps if notveryfied sources error doesnt allow to update
    sudo apt-get upgrade

//------------------------------------------------------------------------------
#### Evolution maill settings
    user@s.com  
    pop3.ww.s.net  
    995  
    ssl pwd  
    leave msg on srv  
    smtp.ww.s.net  
    25  
    no enc

//------------------------------------------------------------------------------
## Setup Bluetooth
    dmesg | grep -i blue
    lspci | grep -i blue
    lsusb | grep -i blue
    
    sudo apt-get install bluez-utils
    sudo /etc/init.d/bluetooth restart
    hcitool dev

//------------------------------------------------------------------------------
## Setup WIFI (since Ubuntu 16.04)

Edit Connections -> Add -> WiFi ... -> Connect to hiden

    SSID: <name>
    Mode: Hotspot
    Device: <choose>
    Security: WPA & WPA2 Personal
    IPv4: Shared to others


//------------------------------------------------------------------------------
## Setup WIFI (on pre Ubuntu 16.04)
    sudo apt-get install hostapd dnsmasq

#### file: /etc/dnsmasq.conf
    bind-interfaces  
    interface=wlan0  
    dhcp-range=192.168.150.2,192.168.150.10

#### in bash
    sudo service hostapd stop
    sudo service dnsmasq stop
    sudo update-rc.d hostapd disable
    sudo update-rc.d dnsmasq disable

#### hostapd.conf
    interface=wlan0  
    driver=nl80211  
    ssid=sdjlh  
    hw_mode=g  
    channel=6  
    wpa=2  
    wpa_passphrase=33323345  

#### start.sh
    #!/bin/bash  
    # fix 14.04 troubles  
    sudo nmcli nm wifi off  
    sudo rfkill unblock wlan
    # Start  
    # Configure IP address for WLAN  
    sudo ifconfig wlan0 192.168.150.1
    # Start DHCP/DNS server  
    sudo service dnsmasq restart
    # Enable routing  
    sudo sysctl net.ipv4.ip_forward=1
    # Enable NAT  
    sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    # Run access point daemon  
    sudo hostapd /etc/hostapd/hostapd.conf
    # Stop  
    # Disable NAT  
    sudo iptables -D POSTROUTING -t nat -o eth0 -j MASQUERADE
    # Disable routing  
    sudo sysctl net.ipv4.ip_forward=0
    # Disable DHCP/DNS server  
    sudo service dnsmasq stop  
    sudo service hostapd stop

#### Broadcom BCM43228 (not tested)
	lspci
	
03:00.0 Network controller: Broadcom Corporation BCM43228 802.11a/b/g/n

	sudo apt-get install linux-headers-generic
	sudo apt-get install --reinstall bcmwl-kernel-source

//--------------------------------------------------------------
## SSD settings
#### tools

    swapon -s
    sudo blkid 

#### folders
    /var/cache  
    /media/home

#### commands
    sudo cp /etc/fstab /etc/fstab.$(date +%Y-%m-%d)
    sudo gedit /etc/fstab &
    
UUID=????????   /media/home    ext4          defaults       0       2   

    sudo mkdir /media/home
    sudo mount -a
    sudo rsync -aXS /home/. /media/home/.
    
UUID=????????   /home    ext4          defaults       0       2  

    cd / && sudo mv /home /old_home && sudo mkdir /home
    sudo mount -a
    
/etc/fstab:  
###### swap was on /dev/sda6 during installation
UUID=xxxxx none         swap    sw          0   0  
###### home
UUID=xxxxx /home        ext4    defaults    0   2  
###### cashe
UUID=xxxxx /var/cache   ext4    defaults    0   2  
###### tmp
tmpfs      /tmp         tmpfs   defaults    0   0  
tmpfs      /var/tmp     tmpfs   defaults    0   0  


#### Comandline-one-liners
###### Create a script of the last executed command
    echo "!!" > script.sh
###### Runs previous command but replacing
    echo no typos
    ^typos^errors
###### List of commands you use most often
    history | awk '{print $2}' | sort | uniq -c | sort -rn | head
###### Execute a command without saving it in the history
    <space>command
###### Make directory including intermediate directories
    mkdir -p a/long/directory/path
###### Create simple text file from command line
    cat > file.txt
    {your text here}
    {your text here}
    <ctrl-d>
###### Show PATH in a human-readable way
    echo $PATH | tr ':' '\n'
###### Share a file between two computers
    nc -l 5566 > data-dump.sql
    nc <his-ip-address> 5566 < data-dump.sql
###### Set audible alarm when an IP address comes online
    ping -a IP_address
###### List programs with open ports and connections
    lsof -i
###### Currently mounted filesystems in nice layout
    mount | column -t
###### Display free disk space
    df -h
###### Display disk usage statistics for the current directory
    du -sh *
###### Execute a command at a given time
    echo "ls -l" | at midnight
###### Simple stopwatch
    time read
    <ctrl-d>
###### Put a console clock in top right corner
    while sleep 1;do tput sc;tput cup 0 $(($(tput cols)-29));date;tput rc;done &
    tput clear; while sleep 1; do tput sc; tput cup 0 0; <commands>; tput rc; done
###### Display the top ten running processes. (Sorted by memory usage)
    ps aux | sort -nk +4 | tail
###### 32 bits or 64 bits
    getconf LONG_BIT
###### Displays a calendar
    cal 02 1988
###### Quick access to the ascii table
    man ascii
###### Proc temperature
    cat /sys/class/thermal/thermal_zone*/temp
    cat /sys/class/thermal/thermal_zone*/type
    paste <(cat /sys/class/thermal/thermal_zone*/type) <(cat /sys/class/thermal/thermal_zone*/temp) | column -s $'\t' -t
    ${hwmon 2 temp 1}°C     # conky
    
#### viewing thread information
    ps -Lf
    ps -T
    ps -Lm
    top -H

TotalView debugger is LC's recommended debugger for parallel programs  
computing.llnl.gov/code/content/software_tools.php  
Some tools worth investigating, specifically for threaded codes, include:  
Open|SpeedShop  
TAU  
HPCToolkit  
PAPI  
Intel VTune Amplifier  
ThreadSpotter  


## ubuntu-nvidia
    sudo apt-get upgrade
    sudo apt-get install ubuntu-restricted-extras

after this updates not working with error:  
failed to open plugin /usr/lib/gs-plugins-9

###### fixed
    sudo apt-get autoremove gnome-software
    sudo apt-get install gnome-software


software & updates  
software updater

#### drivers updates
###### intel microcode
###### nvidia 361.42

    sudo apt-get install mesa-utils
    glxinfo | grep -i opengl
    
###### nvidia settings - set graphics card


#### answer 1
    sudo apt-get purge nvidia*
    
###### fixes xorg
    sudo apt-get install --reinstall xserver-xorg-video-intel  libgl1-mesa-glx libgl1-mesa-dri xserver-xorg-core
    sudo dpkg-reconfigure xserver-xorg
    sudo update-alternatives --remove gl_conf /usr/lib/nvidia-current/ld.so.conf
    
###### reinstall nVidia software
    sudo apt-add-repository ppa:xorg-edgers/ppa
    sudo apt-get update
    sudo apt-get install bumblebee-nvidia nvidia-319 nvidia-settings-319


#### answer 2
I tried manually installing the Nvidia proprietary drivers  
un-installing them with a

    sudo ./NVIDIA-Linux-x86-331.67.run --uninstall
    
###### every option in the driver manager always resulted in:

OpenGL vendor string: VMware, Inc.  
OpenGL renderer string: Gallium 0.4 on llvmpipe (LLVM 3.3, 256 bits)

#### use answer 1
    sudo apt-get install nvidia-337 nvidia-settings-337

## ubuntu-default-app

#### /usr/share/applications/defaults.list
    text/qt-project-file=DigiaQt-qtcreator-community.desktop

#### ~/.local/share/applications/DigiaQt-qtcreator-community.desktop
    [Desktop Entry]
    Type=Application
    Exec=/media/bacchus/dubhe/tools/qt/Tools/QtCreator/bin/qtcreator
    Name=Qt Creator (Community)
    GenericName=The IDE of choice for Qt development.
    Icon=QtProject-qtcreator
    Terminal=false
    Categories=Development;IDE;Qt;
    MimeType=text/x-c++src;text/x-c++hdr;text/x-xsrc;application/x-designer;application/vnd.qt.qmakeprofile;application/vnd.qt.xml.resource;text/x-qml;text/x-qt.qml;text/x-qt.qbs;


#### qt-pro.xml
    <?xml version="1.0"?>
    <mime-info xmlns='http://www.freedesktop.org/standards/shared-mime-info'>
      <mime-type type="text/qt-project-file" subtype="text/plain">
        <comment>QtCreator project</comment>
        <glob pattern="*.pro"/>
      </mime-type>
    </mime-info>

#### install mime type
    sudo xdg-mime install qt-pro.xml

#### copy your custom icon to ~/.icons
    cp QtProject-qtcreator.png ~/.icons/

## slow ubuntu 16.04
[askubuntu:](https://askubuntu.com/questions/584636/kidle-inject-causing-very-high-load)
top shows kidle_inject/x 50%+ cpu load

    sudo rmmod intel_powerclamp
    echo "blacklist intel_powerclamp" > /etc/modprobe.d/disable-powerclamp.conf

## ERROR: The driver descriptor says the physical block size is 2048 bytes, but Linux says it is 512 bytes.
    sudo dd if=/dev/zero of=/dev/<disk-like:sdc> bs=2048 count=32

# ------------------------------------------------------------------------------
## GCC
    sudo update-alternatives --remove-all gcc 
    sudo update-alternatives --remove-all g++

    sudo apt install gcc-5 g++-5

    sudo update-alternatives --config gcc

    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 10
    sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-5 10

    sudo update-alternatives --install /usr/bin/cc cc /usr/bin/gcc 30
    sudo update-alternatives --set cc /usr/bin/gcc

    sudo update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++ 30
    sudo update-alternatives --set c++ /usr/bin/g++

# ------------------------------------------------------------------------------
## CPU temp
    sudo apt install lm-sensors hddtemp
    sudo sensors-detect
    sensors

# ------------------------------------------------------------------------------
## Add/del/change user
    adduser <newuser>
    groups <newuser>
    usermod -aG sudo <newuser>
    
    deluser --remove-home <olduser>
    
    # from start screen Ctrl+Alt+F1, login
    sudo passwd root # unlock the root
    ...
    exit
    # login as root
    usermod -l <newname> -d /home/<newname> -m <oldname>
    groupmod -n <newgroup> <oldgroup>
    passwd -l root # lock the root
    # if using ecryptfs:
    # - mount: ecryptfs-recover-private
    # - edit: <mountpoint>/.ecryptfs/Private.mnt
    exit
    # go to ui Ctrl+Alt+F7
    

