## Ubuntu tricks
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

#### Alt mouse move window
    sudo apt-get install dconf-tools
dconf-editor → org → gnome → desktop → wm → preferences → mouse-button-modifier

#### Change pdf encoding
    gs -sDEVICE=pdfwrite -dNOPAUSE -dBATCH -dSAFER  -sOutputFile=output.pdf input.pdf  
int pdf viewer: file\save as\format\settings\UTF-8\save

#### Restart ui
    sudo service lightdm restart
#### Restart network
    sudo service network-manager restart

#### Enable WebGL
chrome://flags - Override software rendering list - Enable

#### Disable touchpad
    xinput list | grep Touch
    xinput set-prop 13 "Device Enabled" 0

#### Unzip
    tar -zxvf <file.tar.gz>
    tar -jxvf <file.tar.bz2>

#### See progress bar: md5sum filename
    pv filename | md5sum

#### Hardware info
    lspci - VGA
    lsusb
    demidecode
    cat /proc/meminfo
    cat /proc/cpuinfo
    cat /proc/partitions

#### Install
nautilus-open-terminal  
CompizConfig  
evolution  
ubuntu-restricted-extras  
chromium-codecs-ffmpeg-extra  
libalut-dev  
libopenal-dev

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

#### sticky-bit
    chmod +t sticky-bit

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

#### List git-ignored files
    git ls-files . --ignored --exclude-standard --others

#### List untracked files
    git ls-files . --exclude-standard --others
    git diff --name-status

//------------------------------------------------------------------------------
## Certificates
    sudo apt-get install libnss3-tools
    sudo cp <cer-folder>\ *.cer /usr/share/ca-certificates/
    sudo dpkg-reconfigure ca-certificates
    certutil -d sql:$HOME/.pki/nssdb -A -t "C,," -n "<name>" -i SRK\ Interseption.cer

//------------------------------------------------------------------------------
## Killing process
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

#### eclipse crash: add following lines to eclipse.ini
-Dorg.eclipse.swt.browser.DefaultType=mozilla  
-Dorg.eclipse.swt.browser.XULRunnerPath=path_to_xullrunner

#### Physical Source Lines of Code (SLOC)
    sloccount directoryname

#### Building android project on linux project root:
    <PathToSDK>/tools/android update project --name <ProjectName> --target "android-15" --path .
    ndk-build
    ant debug

#### Qt setting custom makefile
-o qMakefile  
-f qMakefile

    LD_LIBRARY_PATH=\`pwd\` ./<executable>

    gcc -Wall -fno-stack-protector stacksmash.c -o stacksmash

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
#### Generate and set key
    cd ~/.ssh
    ssh-keygen -t rsa
    sftp bacchus@106.125.32.44
    mkdir .ssh
    cd .ssh
    put id_rsa.pub
    exit
    ssh bacchus@106.125.32.44
    cd .ssh
    cat id_rsa.pub > authorized_keys
    exit

#### mount remote ssh dir: sshfs *ssh-dir* *local-dir*
    sshfs bacchus@106.125.32.44:/surc/Projects/ve/share ~/shares/ve_share

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
user@ss.com  
pop3.w1.ss.net  
995  
ssl pwd  
leave msg on srv  
smtp.w1.ss.net  
25  
no enc

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

#### file: hostapd.conf
interface=wlan0  
driver=nl80211  
ssid=sdjlh  
hw_mode=g  
channel=6  
wpa=2  
wpa_passphrase=33323345  

#### file: start.sh
\#!/bin/bash  
\# fix 14.04 troubles  
sudo nmcli nm wifi off  
sudo rfkill unblock wlan
\# Start  
\# Configure IP address for WLAN  
sudo ifconfig wlan0 192.168.150.1
\# Start DHCP/DNS server  
sudo service dnsmasq restart
\# Enable routing  
sudo sysctl net.ipv4.ip_forward=1
\# Enable NAT  
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
\# Run access point daemon  
sudo hostapd /etc/hostapd/hostapd.conf
\# Stop  
\# Disable NAT  
sudo iptables -D POSTROUTING -t nat -o eth0 -j MASQUERADE
\# Disable routing  
sudo sysctl net.ipv4.ip_forward=0
\# Disable DHCP/DNS server  
sudo service dnsmasq stop  
sudo service hostapd stop

//--------------------------------------------------------------
## SSD settings
#### tools

    swapon -s
    sudo blkid 

#### folders
/var/cache  
/media/home

#### commands
    sudo gedit /etc/fstab &
    
UUID=????????   /media/home    ext4          defaults       0       2   

    sudo mkdir /media/home
    sudo mount -a
    sudo rsync -aXS /home/. /media/home/.
    
UUID=????????   /home    ext4          defaults       0       2  

    cd / && sudo mv /home /old_home && sudo mkdir /home
    sudo mount -a
