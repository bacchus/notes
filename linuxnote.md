## Ubuntu tricks
#### Nautilus
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
alt-f2		    run

#### Open from terminal
    gnome-open <filename>

#### Unmount tatra
    udisks --unmount /dev/sdb1
    udisks --detach /dev/sdb

#### Dropbox encfs
    sudo apt-get install encfs
    #sudo addgroup <username> fuse
    encfs ~/Dropbox/.encrypted ~/private
    sudo install ~/gnome-encfs /usr/local/bin
    gnome-encfs -a ~/Dropbox/.encrypted ~/private
    fusermount -u ~/private

#### Googling
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

#### Tweeter unfolow all
Open "Following" page on Twitter https://twitter.com/following
scroll down till all following accounts showed
open developer console(CTRL+SHIFT+C)
$('.button-text.unfollow-text').trigger('click');

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

#### Aliases
    alias sag="sudo apt-get install -y"
    alias gr='git rebase -i upstream/master --autosquash'
    alias gfu='git fetch upstream develop'
    alias gup='git checkout develop && git fetch upstream develop && git pull --ff-only upstream develop && git push'
    alias fresh='git checkout develop && git fetch upstream develop && git pull --ff-only upstream develop && git push && grit fresh '
    alias rtags="find src/ripple -name \*.h -or -name \*.hpp -or -name \*.cpp | xargs etags"
    alias gdiff="git diff > /tmp/git.diff"
    alias sb='source ~/.bashrc'

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

#### Enable WebGL
    chrome://flags - Override software rendering list - Enable

#### Disable touchpad (in 14.04 from settings)
    xinput list | grep Touch
    xinput set-prop 13 "Device Enabled" 0

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
###### Install
    sudo dpkg -i <pkgname>
###### Remove
    sudo dpkg -r <pkgname>
###### Purge
    sudo dpkg -P <pkgname>

#### Hardware info
    lspci - VGA
    lsusb
    demidecode
    cat /proc/meminfo
    cat /proc/cpuinfo
    cat /proc/partitions
    ls -l /dev/disk/by-uuid/

#### System info
    date - cur date and time
    cal - calendar on cur month
    uptime - time from boot
    w - users online
    whoami - login name
    uname -a - kernel info
    df - discs usage
    du - weight of cur dir
    du -sh <dir> - weight of <dir> in human readable
    free - memory usage and swap
    whereis app - location of app
    which app - which app will be run

#### Commands
    passwd          change pasword
    locate file     find file
    dpkg -i pkg.deb install package
    gksu            sudo with graphical ui
    crontab -e      edit cron tasks

#### Install

    nautilus-open-terminal
    compizconfig-settings-manager
    ubuntu-restricted-extras
    chromium-codecs-ffmpeg-extra

    sudo apt-get clean
    sudo apt-get update
    sudo apt-get upgrade

    necessitas qt gnu-octave meld git
    skype gimp chromium stellarium vlc cheese-webcam guvcview guake

#### Desktop message
    notify-send "Title" "Text"

#### System Program Problem Detected
1. disable apport enabled=1 to enabled=0
    sudo gedit /etc/default/apport
    sudo restart apport
or just reboot
2. remove old crash reports
    sudo rm /var/crash/*

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
## GIT
#### Save direct commits history
    git pull --rebase
    git fetch origin
    git status
    git fsck
    git reflog
    git rebase -i HEAD~3

git stash -> git pull -> git stash apply -> fix conflicts
or just
    git rebase --autostash
or in config
rebase.autostash

#### Push to remote branch
    git push [remotename] [localbranch]:[remotebranch]

#### Delete remote branch
    git push [remotename] :[remotebranch]

#### resolve conflicts
    git checkout --theirs -- path/to/conflicted-file.txt
    git checkout --ours -- path/to/conflicted-file.txt

#### Stash
    git stash
    git stash list
    git stash apply stash@{2}
    git stash drop stash@{0}
    git stash pop # = apply + drop
    git stash branch [name] # create branch from stash
###### stash options
    --keep-index - not to stash staged
    --include-untracked or -u - +untracked

#### Clean
###### ☠ Use with caution: remove not tracked
    git clean
###### safer: remove everything but save it in a stash
    git stash --all
###### remove all the untracked files in your working directory
    git clean -d
-x - remove ignored
-n - dry run
-f - force ☠
-i - interactive

#### List git-ignored files
    git ls-files . --ignored --exclude-standard --others

#### List untracked files
    git ls-files . --exclude-standard --others
    git diff --name-status

#### Tags
    git tag -a v1.4 -m 'my version 1.4'
    git fetch --tags

#### Difftool: meld folders
    git difftool -d master..devel
    git difftool -d [branch]

#### Merge
    git merge master feature

#### Rebase
rebase the current branch onto <base>
    git rebase <base>

    git checkout feature
    git checkout -b temporary-branch
    git rebase -i master
clean up the history
    git checkout master
    git merge temporary-branch

rebase of only the last 3 commits
    git rebase -i HEAD~3

returns the commit ID of the original base
    git merge-base test_dev perforce

rebase merge conflict
    git mergetool
    git rebase --continue

reset: --soft --mixed --hard
    git checkout hotfix
    git reset HEAD~2 //foo.cpp
checkout
    git checkout HEAD~2 //foo.cpp
revert
    git checkout hotfix
    git revert HEAD~2

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
## Patching
#### Make patch
    diff -ruN ../boost_1_52_0 . > ../boost-droid-bcpd.diff
#### Apply patch
    patch -Np0 --dry-run < boost-droid-bcpd.diff

//------------------------------------------------------------------------------
#### Show number of CPU cores
    grep -c ^processor /proc/cpuinfo

#### Show first 5 errors
    <make> 2>&1|grep error|head -5|tee log.txt

#### Display information about the contents of ELF format files
    readelf <option(s)> elf-file(s)
-a --all               Equivalent to: -h -l -S -s -r -d -V -A -I

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

#### Qt setting custom makefile
-o qMakefile
-f qMakefile

    pkg-config --cflags --libs opencv

    locate libQt5Script.so

    LD_LIBRARY_PATH=\`pwd\` ./<executable>

    gcc -Wall -fno-stack-protector stacksmash.c -o stacksmash

    std::cout << "\n         (__)\n         (oo)\n   /------\\/\n  / |    ||\n *  /\\---/\\\n    ~~   ~~\n\n";

    struct T{U u;T(U u):u(std::move(u)){}};

//------------------------------------------------------------------------------
## Bash
#### Scroll up and down the list. 'q' to quit
    ls | less
    more
    head <filename>

#### Rename
    rename 'y/A-Z/a-z/' *
    rename 's/.df/.txt/g' *.df
    rename 's/aa/12/g' *
    sed -i 's/old-word/new-word/g' *.txt

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

#### Find in files
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
#### Finds "word" in files in subdirs
    grep -rl ‘text’ directory/*
    grep -r word *

###### -r recursive, -n line number, -w whole word
###### -i ignore-case, -v non-matching,  -l print files matches
###### -B before, -A after, -C context
###### --include-dir=dir0
###### --exclude-dir={dir1,dir2,*.dst}
###### --include=\\*.{c,h}
###### --exclude=*.o
    grep -rnw 'directory' -e "pattern"
###### -iwrvn -locb

#### EXPRESSIONS
^ - begin line
$ - end line
grep -c "^$" - count empty lines

'1*' matches zero or more '1'
. any symb
\\+ one or more of prev symb
\\? zero or one
\\b words boundary

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
###### -i5 increase cnt, -s. add string '.' after cnt, -w2     column for cnt
###### -b style: a all lines,  t nonempty lines, n no lines, pREG lines for regexp (-bpA - only with A)
###### -n format: ln left justified no leading zeros, rn right -same-,  rz right j lead z
    nl file.txt

#### wc - coun lines, words, bytes
###### -w words, -L length of longest line, -l new lines, -c bytes
    wc file.txt
#### lines words bytes
5     5     41     sort.txt

#### sed
###### a\ append, i\ insert, c\ replace, = print line number
###### ADDRESS line number, PATTERN reg, $ end of file

    sed 'ADDRESS a\
        Line which you want to append' filename

    sed '/PATTERN/ a\
        Line which you want to append' filename

    sed '$ a\
        Line which you want to append to end of file' filename

    sed '/PATTERN/=' filename

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

#### Generate and set key
    cd ~/.ssh
    ssh-keygen -t rsa
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
## Setup WIFI
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
###### Quickly rename a file
    mv filename.{old,new}
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
###### Display the top ten running processes. (Sorted by memory usage)
    ps aux | sort -nk +4 | tail
###### 32 bits or 64 bits
    getconf LONG_BIT
###### Displays a calendar
    cal 02 1988
###### Quick access to the ascii table
    man ascii


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



