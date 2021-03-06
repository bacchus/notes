# New stuff


## repo-help
see: https://source.android.com/setup/using-repo

    curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
    chmod a+x ~/bin/repo
    sudo mv ~/bin/repo /usr/local/bin/repo
    
    repo init -u <path-to-manifest> -b <branch> --depth=1
    repo sync -j8 -c -d --no-tags --force-sync
    repo status -j8
    
|  commands                 | description                                       |
| -:                        | -                                                 |
|  abandon                  | Permanently abandon a development branch          |
|  | same:                      git branch -D <branch>                          |
|  branch                   | View current topic branches                       |
|  branches                 | View current topic branches                       |
|  checkout                 | Checkout a branch for development                 |
|  | same: repo                 forall [<project>] -c git checkout <branch>     |
|  cherry-pick              | Cherry-pick a change by <sha1>                    |
|  diff                     | Show changes between commit and working tree      |
|  download                 | Download and checkout a change                    |
|  forall                   | Run a shell command in each project               |
|  | -c command             |   (and arguments) to execute                      |
|  grep                     | Print lines matching a pattern                    |
|  help <command>           |  Display detailed help on a command               |
|  init                     | Initialize repo in the current directory          |
|  | -u URL                     manifest repository location                    |
|  | -b REVISION                manifest branch or revision                     |
|  list                     | List projects and their associated directories    |
|  manifest                 | Manifest inspection utility                       |
|  prune                    | Prune (delete) already merged topics              |
|  rebase                   | Rebase local branches on upstream branch          |
|  selfupdate               | Update repo to the latest version                 |
|  smartsync                | Update to the latest known good revision          |
|  stage                    | Stage file(s) for commit                          |
|  start                    | Start a new branch for development                |
|  status                   | Show the working tree status                      |
|  sync                     | Update working tree to the latest revision        |
|  | -c                         only current branch                             |
|  | -d, --detach               detach HEAD to manifest                         |
|  | -f, --force-broken         continue sync even if fails to sync             |
|  | -l, --local-only           only update working tree, don't fetch           |
|  | -n, --network-only         fetch only, don't update working tree           |
|  | --force-sync               ??? force                                       |
|  upload                   |    Upload changes for code review                 |
|  version                  |    Display the version of repo                    |


## adb, minicom permissions
    who # get user
    sudo adduser <user> plugdev
    adb devices
    lsusb
    ls -l /dev/bus/usb/<bus>/<dev>
    gedit /etc/udev/rules.d/51-android.rules
        SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="d002", MODE="0660", GROUP="plugdev", SYMLINK+="android%n"
    sudo adduser <user> dialout
    sudo chmod a+rw /dev/ttyUSB0

## package name from apk
    aapt list -a /path/to/file.apk
    aapt dump badging <apk> | grep package
    pm list packages -f
    package manifest $<com-package-name>

    package=$(aapt dump badging "$*" | awk '/package/{gsub("name=|'"'"'","");  print $2}')
    activity=$(aapt dump badging "$*" | awk '/activity/{gsub("name=|'"'"'","");  print $2}')
    echo "   file : $1"
    echo "package : $package"
    echo "activity: $activity"

## start/stop apk
    adb shell am force-stop com.my.package
    adb shell am start -n com.my.package/com.my.package.activity

## remount, start, stop
    # adb shell "su -c mount -o remount,rw /dev/block/platform/msm_sdcc.1/by-name/system /system"
    adb remount
    
    # k=`adb shell "ps surfaceflinger" | awk '{print $2}' | grep -v PID` && adb shell "su -c kill $k"
    adb shell stop
    adb shell start

## mirror android screen (ok)
    adb exec-out screenrecord --bit-rate=16m --output-format=h264 --size 800x600 - | \
    ffplay -framerate 60 -framedrop -bufsize 16M -
    # test flags: -fast -infbuf
    
    screenrecord --bugreport --verbose --time-limit 10 /sdcard/test.mp4
    am broadcast -a android.intent.action.MEDIA_SCANNER_SCAN_FILE -d file:///sdcard/test.mp4

## decompile
    decompileandroid.com
    dex2jar, JAD, apktool

--------------------------------------------------------------------------------
# Old stuff

## Android Studio

#### TODO: make table

alt+enter   show issue solutions
f2          next error
shift+f2    prev error

shift+f6    rename

alt+c r     auto indent

alt+1|2|3   show side panes
shift+esc   hide side pane

ctrl+i      implement method
ctrl+o      override method

alt+insert  create new instance on one of project folders

ctrl+d      duplicate line or selected code
ctrl+y      delete line
ctrl+q      quick help
ctrl+p      show parameters for method

ctrl+alt+shift+n    search symbol

f4          jump to source

alt+arrows  navigate tabs

ctrl+f9     build
shift+f10   build + run

ctrl+w          select block
ctrl+shift+w 
ctrl+shift+up/down  move block

ctrl+shift+enter    complete statement

ctrl+e      recent files

ctrl+j      templates code

ctrl+shift+j    join lines

Alt+F1+8        open file in files  - conflict: go to main menu
ctrl+alt+t      surround with       - conflict: open terminal
ctrl+shift+del




###### android/frameworks/base/libs/<name>

thorsmspl  
developer.android.com/guide/topics/graphics/hardware-accel.html  
slideshare.net/jserv/accel2drendering  
0xlab.org  
Learning about Android Graphics Subsystem, Bhanu Chetlapalli  
https://www.youtube.com/watch?v=v9S5EO7CLjo  
https://www.youtube.com/watch?v=duefsFTJXzc  
https://thenewcircle.com/s/post/570/video_creating_sticky_gui_in_android_with_romain_guy_chet_haase_of_google  
http://www.curious-creature.org/2010/12/02/android-graphics-animations-and-tips-tricks/  
https://github.com/romainguy?tab=repositories  
http://www.curious-creature.org/category/android/  

#### biuld
	source ./build/envsetup.sh
	lunch <target>
	make -j4 <libname>
	make -j4 libandroid_runtime
	cd frameworks/base & mma

#### Deploy <libname>
	adb push out/target/product/obj/lib/<libname>.so /data/local/<libname>.so.custom && 
	adb shell "su -c mount -o remount,rw /dev/block/platform/msm_sdcc.1/by-name/system /system" &&
	adb shell "su -c cp /data/local/<libname>.so.custom /system/lib/<libname>.so" &&  
	k=`adb shell "ps surfaceflinger" | awk '{print $2}' | grep -v PID` && adb shell "su -c kill $k"

#### Deploy java framework
	adb shell "su -c cp -v /data/local/framework.jar.custom /system/framework/framework.jar"

#### Rebuild
	mmm ./frameworks/base/libs/<name>
	mmm ./frameworks/base/core/jni/
	mmm ./frameworks/base/

#### showfreq
	adb shell su -c setenforce 0
	adb remount
	adb push showfreq data/
	adb shell chmod 777 data/showfreq
	adb shell data/showfreq 500 ##This is time period to report information

#### Set Property
	# in adb shell
	getprop "debug.pr.propname"
	setprop "debug.pr.propname" 1

	// in code
	#include "Properties.h"
	char property[PROPERTY_VALUE_MAX];
	property_get("debug.pr.propname", property, "0");
	if (atoi(property) != 0) {

#### Adreno Profiler
	adb shell setprop debug.egl.profiler 1
	<uses-permission android:name="android.permission.INTERNET" />
	<uses-permission android:name="com.qti.permission.PROFILER" />
	graphs: fps clock busy shad busy frag busy frag inst

#### Set CPU max freq
	adb shell su -c
	adb shell "echo 2457600 > /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq"
	adb shell "echo 2457600 > /sys/devices/system/cpu/cpu0/cpufreq/scaling_min_freq"
	adb shell "echo performance > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor"

	echo max_freq
	adb shell "cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq"
	echo min_freq
	adb shell "cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_min_freq"
	echo governor
	adb shell "cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor"

	cd /sys/devices/system/cpu/cpu0/cpufreq
	echo 2457600 > scaling_max_freq
	echo 2457600 > scaling_min_freq
	echo "performance" > scaling_governor

	adb shell gfxinfo <PID>
	adb shell vmstat
	addr2line
	*#9900#
	wssyncmlnps - process

#### Logcat by tag
	adb logcat ActivityManager:I MyApp:D *:S
	-v tag
	-b radio|events|main
	-f <filename>
    redirect stderr to stdout
	adb shell stop
	adb shell setprop log.redirect-stdio true
	adb shell start

#### Build Kernel
	adb pull /proc/config.gz
	kernel/arch/arm/configs

	cd android/kernel
	export ANDROID_KERNEL_DIR=`pwd`
	export ARCH=arm
	export CROSS_COMPILE=../prebuilts/gcc/linux-x86/arm/arm-eabi-4.8/bin/arm-eabi-
	export VARIANT_DEFCONFIG=msm8974pro_sec_klte_eur_defconfig
	export DEBUG_DEFCONFIG=msm8974_sec_eng_defconfig
	export SELINUX_DEFCONFIG=selinux_defconfig
	export SELINUX_LOG_DEFCONFIG=selinux_log_defconfig
	export TIMA_DEFCONFIG=
	make msm8974_sec_defconfig

#### Build daemon
	cd daemon-src
	mv gator-daemon jni
	cd jni
	ndk-build -j
	cp ../libs/armeabi/gatord ../../

#### Build driver
	cd driver-src/gator-driver
	make -C $work_dir M=`pwd` modules
	cp gator.ko ../../

#### Copying & starting gator
	adb forward tcp:8080 tcp:8080
	adb push gator.ko /sdcard/gator.ko
	adb push gatord /sdcard/gatord
	adb shell
	su
	cp /sdcard/gator* /data/
	chmod 777 /data/gatord
	/data/gatord&

	./mkbootimg --kernel kernel --ramdisk ramdisk.img --output boot.img --cmdline "console=null androidboot.hardware=qcom user_debug=23 msm_rtb.filter=0x37 ehci-hcd.park=3" --base 0x00000000 --pagesize 2048 --ramdisk_offset 0x02000000 --tags_offset 0x01E00000 --dt dt.img

	tools/dtbTool -o dt.img -s 2048 -p scripts/dtc/ arch/arm/boot/


#### Net android
OkHttp
Retrofit
Volley

SQLiteOpenHelper
SQLiteDatabase

ORMLite GreenDao SQLCipher  
ES File Explorer  
Android Terminal Emulator  
Orai lietuvo  
SimpleScreenRecorder  

Wi-Fi analyzer, Fing, ConnectBot, BitTorrentSync, Owncloud, KeyPass

Cities in motion, Tycoon
