#### Android Studio
    
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

#### Decompile
    decompileandroid.com
    dex2jar, JAD, apktool

#### droid_deploy.sh
    #!/bin/bash
    
    TARGET=$1
    LIBS=$2
    ECHO_ONLY=1
    LIBS_NUM=0
    
    function write_help {
        echo ""
        echo "Usage: droid_deploy.sh <target> <libs>"
        echo "  target:     klte k3g"
        echo "              curdir: takes libs from current dir"
        echo "              device: backup libs from device to current dir"
        echo "  libs:       hwui android_runtime framework - haf"
        echo "example: deploy haf"
        echo ""
    }
    
    function deply_system {
        if [ $# -ne 2 ]; then
            echo "deply_system wrong params num"
            exit 1
        fi
    
        local LIBNAME=$1
        local SYSNAME=$2
        
        PATH_TO_LIBS="out/target/product/$TARGET/system"
        DEVICE_TEMP_FILE=/data/local/$LIBNAME.custom
        DEVICE_DEST_FILE=/system/$SYSNAME/$LIBNAME
        HOST_LIB_FILE=./$LIBNAME
        
        if [ "$TARGET" == "device" ]; then
            if [ $ECHO_ONLY -eq 0 ]; then
                echo "\$ adb pull $DEVICE_DEST_FILE"
            else
                adb pull $DEVICE_DEST_FILE
                if [ $? -ne 0 ]; then
                    echo "pull from device failed"
                    exit 1
                fi
            fi
            return 0;
        fi
        
        if [ "$TARGET" != "curdir" ]; then
            HOST_LIB_FILE=$PATH_TO_LIBS/$SYSNAME/$LIBNAME
        fi
    
        if [ -e $HOST_LIB_FILE ]; then
            if [ $ECHO_ONLY -eq 0 ]; then
                echo "\$ adb push $HOST_LIB_FILE $DEVICE_TEMP_FILE"
                echo "\$ adb shell \"su -c cp -v $DEVICE_TEMP_FILE $DEVICE_DEST_FILE\""
            else
                adb push $HOST_LIB_FILE $DEVICE_TEMP_FILE
                adb shell "su -c cp -v $DEVICE_TEMP_FILE $DEVICE_DEST_FILE"
                if [ $? -ne 0 ]; then
                    echo "push and cp to system failed"
                    exit 1
                fi
            fi
                     
            LIBS_NUM=1
        else
            echo "$LIBNAME not found"
            exit 1
        fi
    }
    
    function deply_system_lib {
        deply_system $1 lib
    }
    
    function deply_system_framework {
        deply_system $1 framework
    }

    if [ $# -ne 2 ]; then
        write_help
        exit 1
    fi
    
    if [ "$TARGET" != "device" ]; then
        if [ $ECHO_ONLY -eq 0 ]; then
            echo "\$ adb shell \"su -c mount -o remount,rw /dev/block/platform/msm_sdcc.1/by-name/system /system\""
        else
            adb shell "su -c mount -o remount,rw /dev/block/platform/msm_sdcc.1/by-name/system /system"
            if [ $? -ne 0 ]; then
                echo "first remount rw failed, trying second"
                adb shell "su -c mount -o remount,rw /dev/block/platform/dw_mmc.0/by-name/SYSTEM /system"
                if [ $? -ne 0 ]; then
                    echo "remount rw failed"
                    adb devices
                    exit 1
                fi
            fi
            #   error: device not found
            #   error: device offline
            #   mount: Permission denied
            #   Read-only file system
        fi
    fi

    if [[ "$LIBS" == *"h"* ]]; then
        deply_system_lib libhwui.so
    fi
    
    if [[ "$LIBS" == *"a"* ]]; then
        deply_system_lib libandroid_runtime.so
    fi
    
    if [[ "$LIBS" == *"f"* ]]; then
        deply_system_framework framework.jar
    fi
    
    if [ $LIBS_NUM -eq 1 ]; then
        if [ $ECHO_ONLY -eq 0 ]; then
            echo "\$ adb shell stop"
            echo "\$ adb shell start"
        else
            # k=`adb shell "ps surfaceflinger" | awk '{print $2}' | grep -v PID` && adb shell "su -c kill $k"
            adb shell stop
            adb shell start
        fi
    else
        echo "no libs deployed exiting"
        exit 1
    fi

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
