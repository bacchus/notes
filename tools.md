# ------------------------------------------------------------------------------
# Tools

- 01-vndk
- 02-gdbserver
- 03-debuggerd
- 06-systrace
- 07-gapid
- 08-gcov
- 09-heapchk
- 11-projects
- 13-ion
- 15-perf
- 16-eklitzke
- 17-tombstone
- 18-ftrace
- 19-gdb
- 20-strace

# ------------------------------------------------------------------------------
# 01-vndk
    usage: vndk_definition_tool.py [-h]
    vndk            --system --vendor
                    --aosp-system --load-extra-deps
    check-dep       --tag-file --module-info
    deps            --revert --symbol
    deps-insight
                    --full

    elfdump                 <file>
    create-generic-ref
    deps-closure
    deps-unresolved
    check-eligible-list
    dep-graph

## use 1: vndk
    development/vndk/tools/definition-tool/vndk_definition_tool.py vndk \
    --system ${ANDROID_PRODUCT_OUT}/system \
    --vendor ${ANDROID_PRODUCT_OUT}/vendor \
    --load-extra-deps development/vndk/tools/definition-tool/datasets/minimum_dlopen_deps.txt

    > error: SP-HAL <lib> depends on non vndk-sp library /system/lib64/vndk-sp/libz.so

## use 2: check-dep
    development/vndk/tools/definition-tool/vndk_definition_tool.py check-dep \
    --system ${ANDROID_PRODUCT_OUT}/system \
    --vendor ${ANDROID_PRODUCT_OUT}/vendor \
    --tag-file development/vndk/tools/definition-tool/datasets/eligible-list-28.csv \
    --module-info ${ANDROID_PRODUCT_OUT}/module-info.json

    file:///development/vndk/tools/definition-tool/datasets/eligible-list-custom-release.csv:
    +/system/${LIB}/vndk/<lib>,VNDK,

    > error: vendor lib <lib> depends on non-eligible lib <lib>

## use 3: deps-insight
    ../development/vndk/tools/definition-tool/vndk_definition_tool.py deps-insight \
    --system ${ANDROID_PRODUCT_OUT}/system \
    --vendor ${ANDROID_PRODUCT_OUT}/vendor \
    --tag-file ../development/vndk/tools/definition-tool/datasets/eligible-list-custom-release.csv \
    --module-info ${ANDROID_PRODUCT_OUT}/module-info.json \
    --revert --symbol --full

## use 4: deps-unresolved
    development/vndk/tools/definition-tool/vndk_definition_tool.py deps-unresolved \
    --system ${ANDROID_PRODUCT_OUT}/system \
    --vendor ${ANDROID_PRODUCT_OUT}/vendor

    --path-filter /system/lib64/

    out:
    /system/lib64/ld-android.so
        UNRESOLVED_SYMBOL: __cxa_finalize

    /system/lib64/libdl.so
        UNRESOLVED_SYMBOL: __cxa_finalize
        UNRESOLVED_SYMBOL: __loader_dlopen

## use 4: elfdump
    ../development/vndk/tools/definition-tool/vndk_definition_tool.py elfdump \
    ${ANDROID_PRODUCT_OUT}/system/lib64/libdl.so

``` txt
In a nutshell, the dynamic linker doesn't know anything about your application
(e.g. where its libraries live), it only knows about the LD_LIBRARY_PATH value
that was set when the process was created. When you start an Android
application, you really fork the Zygote process, you don't create a new one,
so the library search path is the initial one and doesn't include your app's
/data/data/<pkgname>/lib/ directory, where your native libraries live.
This means that dlopen("libfoo.so") will not work, because only
/system/lib/libfoo.so will be searched. When you call System.loadLibrary("foo")
from Java, the VM framework knows the application's directory,
so it can translate "foo" into "/data/data/<pkgname>/lib/libfoo.so",
then call dlopen() with this full path, which will work. It libfoo.so references
"libbar.so", then the dynamic linker will not be able to find the latter.
Add to this that even if you update LD_LIBRARY_PATH from native code,
the dynamic linker will not see the new value. For various low-level reasons,
the dynamic linker contains its own copy of the program's environment as it was
when the process was created (not forked). And there is simply no way to update
it from native code. This is by design, and changing this would have drastic
security constraints. For the record, this is also how the Linux dynamic linker
works, this forces any program that needs a custom library search path to use
a wrapper script to launch its executable (e.g. Firefox, Chrome and many others)
```

## audit2allow
    adb shell dmesg | grep denied | audit2allow -p out/target/product/$board/root/sepolicy

## selinux
    device/vendor_foo/device_bar/sepolicy/appdomain.te:
    # allow <scontext> <tcontext>:<tclass> { <denied> };
    # allow appdomain display_prop:file { getattr map open read };
    avc: denied { read } for pid=2182 comm="android.hardware" name="u:object_r:display_prop:s0" dev="tmpfs" ino=8458
    scontext=u:r:appdomain:s0
    tcontext=u:object_r:display_prop:s0
    tclass=file


# ------------------------------------------------------------------------------
# 02-gdbserver
    sudo apt install gdb-multiarch

    # Debuggers: add /usr/bin/gdb-multiarch
    # Kits: add gdb-multiarch
    # libDebugger.so on /usr/lib/x86_64-linux-gnu/qtcreator/plugins
    # Debugger -> GDB -> Additional Startup Commands:
        set serial baud 115200
        directory $workspace

    su 0 gdbserver64 --attach tcp:5039 `pidof <procs>`
    adb forward tcp:5039 tcp:5039

    # Start Debugging -> Attach to Running Debug Server
        Port: 5039
        Local executable: $workspace/out/target/product/$board/symbols/vendor/bin/hw/<exec>
        Local directory: $workspace


# ------------------------------------------------------------------------------
# 03-debuggerd
    debuggerd [-b] PID
    -b      just a backtrace rather than a full tombstone

    lsblk
    cat /proc/meminfo
    dumpsys meminfo

## CPU Perf analysis
    perf – free sampling profiler
    vtune – Intel’s tool works on Linux, too!
    Telemetry – You’re using this already, right?


# ------------------------------------------------------------------------------
# 06-systrace

on Android 9 system-level app called System Tracing
also you might submit an on-device bug report Quick Settings tile

    #define ATRACE_TAG ATRACE_TAG_GRAPHICS
    ATRACE_CALL();

    char tag[16];
    snprintf(tag, sizeof(tag), "HW_VSYNC_%1u", disp);
    ATRACE_INT(tag, ++mVSyncCounts[disp] & 1);

    python ~/Android/Sdk/platform-tools/systrace/systrace.py \
    -o ~/logs/systr/gfx.html \
    gfx sm sync hal \
    -t 20 -b 96000

    python ~/Android/Sdk/platform-tools/systrace/systrace.py -o ~/logs/systr/disp2.html \
    am binder_driver dalvik freq gfx hal idle input res sched view wm power \
    -t 10 -b 96000

    # try def: am binder_driver camera dalvik freq gfx hal idle input res sched view wm

    python ~/Android/Sdk/platform-tools/systrace/systrace.py -o ~/logs/systr/disp2.html \
    am binder_driver camera dalvik freq gfx hal idle input res sched view wm

    python systrace.py --list-categories

## tags
- adb - ADB
- aidl - AIDL calls
- am - Activity Manager
- audio - Audio
- binder_driver - Binder Kernel driver
- binder_lock - Binder global lock trace
- bionic - Bionic C Library
- camera - Camera
- dalvik - Dalvik VM
- database - Database
- disk - Disk I/O
- freq - CPU Frequency
- gfx - Graphics
- hal - Hardware Modules
- i2c - I2C Events
- idle - CPU Idle
- input - Input
- irq - IRQ Events
- memreclaim - Kernel Memory Reclaim
- mmc - eMMC commands
- network - Network
- pagecache - Page cache
- pdx - PDX services
- pm - Package Manager
- power - Power Management
- regulators - Voltage and Current Regulators
- res - Resource Loading
- rs - RenderScript
- sched - CPU Scheduling
- sm - Sync Manager
- ss - System Server
- sync - Synchronization
- vibrator - Vibrator
- video - Video
- view - View System
- webview - WebView
- wm - Window Manager
- workq - Kernel Workqueues



# ------------------------------------------------------------------------------
# 07-gapid
[https://docs.bazel.build/versions/master/install-compile-source.html]
[https://ninja-build.org/]

    export ANDROID_HOME=$tools/Android/Sdk
    export ANDROID_NDK_HOME=$ANDROID_HOME/ndk-bundle
    bazel build pkg


# ------------------------------------------------------------------------------
# 08-gcov

    LOCAL_CFLAGS += -g -O0 --coverage \
        -fprofile-instr-generate -fcoverage-mapping \
        -Xclang -coverage-cfg-checksum \
        -Xclang -coverage-no-function-names-in-data \
        -Xclang -coverage-version='409*'
    LOCAL_LDFLAGS += --coverage

    extern "C" {
    void __llvm_profile_initialize_file(void);
    void __llvm_profile_set_filename(const char*);
    int __llvm_profile_write_file(void);
    }
    __llvm_profile_set_filename("/data/misc/llgcov");
    int res = __llvm_profile_write_file();
    ALOGD("BCC: __llvm_profile_write_file: %d", res);


# ------------------------------------------------------------------------------
# 09-heapchk
    adb root
    adb shell ps | grep <prcs>
    adb shell am dumpheap -n 2316 /data/local/tmp/dump1
    adb pull /data/local/tmp/dump1


# ------------------------------------------------------------------------------
# 11-projects
    $workspace/
    external/drm_hwcomposer/drmencoder.h                           google hwcomposer
    external/libdrm/tests/planetest                                        drm tests
    external/libdrm/xf86drm.h                                                 libdrm
    external/libdrm/xf86drmMode.h                                             libdrm
    hardware/interfaces/graphics/composer/2.1/default/Hwc.h         hwcomposer iface
    hardware/interfaces/graphics/allocator/2.0/default/Gralloc1Allocator.cpp:275
    hardware/libhardware/include/hardware/hwcomposer2.h                  hwcomposer2
    frameworks/native/services/surfaceflinger                         surfaceflinger
    frameworks/native/services/surfaceflinger/DisplayHardware                   HWC2
    frameworks/native/libs/gui                                           bufferqueue
    frameworks/native/libs/ui/include/ui/GraphicBuffer.h               GraphicBuffer
    frameworks/native/include/gui                                        bufferqueue
    system/core/libion                                                        libion
    system/core/libsync                                                      libysnc
    system/libhidl/base/HidlSupport.cpp                                         hidl
    system/core/include/utils/RefBase.h                                   sp<>, wp<>
    system/core/libsystem/include/system/graphics-base.h              graloc formats

    out/soong/.intermediates/hardware/interfaces/graphics/composer/2.1/ \
    android.hardware.graphics.composer@2.1_genc++_headers/gen/ \
    android/hardware/graphics/composer/2.1/IComposer.h                     IComposer

    $kernel/
    drivers/staging/android/ion                                                  ion
    drivers/android/binder.c                                                  binder
    drivers/dma-buf/fence.c                                          sw_sync, fences
    drivers/gpu/drm/drm_ioctl.c              BEWARE THE DRAGONS! MIND THE TRAPDOORS!
    drivers/gpu/drm/bridge/dw-hdmi.h                                         dw hdmi
    drivers/clk/clk-5p49x.c                                                5p49x src
    drivers/media/platform/vsp1/vsp1.h                                           VSP
    include/drm                                                                  DRM
    include/drm/bridge/dw_hdmi.h                                             dw hdmi
    include/media/vsp1.h                                                         VSP
    include/uapi/drm/*.h                                                    user app
    include/uapi/linux/android/binder.h                                       binder


# ------------------------------------------------------------------------------
# 13-ion
    ion, dma-buf, gralloc, ashmem, gem

    ion: kern, sys/core
    ION     is a generalized memory manager
    DMA     Direct Memory Access
    CMA     contiguous mem alloc
    GEM     Graphics Execution Manager (mem alloc and share as ION)
    DMABUF  DMA buffer sharing framework from Linaro
    Gralloc dev-spec user-sp alloc
    dma-buf shared to user-sp using file desc
    PRIME   GEM-spec mech for sharing buff btw dev
    DRM PRIME allow drv to share GEM buff via dma-buf
    V4L     subsys used for camera with integrated dma-buf

    ion -> dma-buf
    heaps
    -- linux,ion-heap-system
    -- linux,ion-heap-system-contig
    -- linux,ion-heap-carveout
    -- linux,ion-heap-chunk
    -- linux,ion-heap-dma
    -- linux,ion-heap-custom

    system/core/libion: open/close, alloc/free, map/share/import/sync
        ion_alloc -> ion_ioctl(fd, ION_IOC_ALLOC, &data);

    kern/drv/staging/android/ion


# ------------------------------------------------------------------------------
# 15-perf
    sudo perf top
    record -p <pid> sleep 5
    -a    all
    -g    trace stack
    list # show event types
    record -e syscalls:sys_enter_connect -ag
    report     # simple report
    annotate   # asm
    script     # as text

- broken stack traces: stack walking: -fno-omit-frame-pointer
- stack depth 127 limit: sysctl -w kernel.perf_event_max_stack=512 [1024]
- fix native symb: -dbgsym

- tools stack trace:

    debuggerd -b <pid>
    stack < /data/tombstones/tombstone_05

- tools symbols:

    nm -D --demangle
    nm -D | c++filt
    objdump
    readelf -a
    file

## droid: simpleperf
    linux: linux-tool-$(uname -a)
    gh: google/perf_data_converter
    gh: google/pprof
    external/linux-tools-perf: See system/extras/simpleperf/ instead
    perf_to_profile
    perfetto

## simpleperf help
    debug-unwind        Debug/test offline unwinding.
    dump                dump perf record file
    help                print help information for simpleperf
    kmem                collect kernel memory allocation information
    list                list available event types
    record              record sampling info in perf.data
    report              report sampling information in perf.data
    report-sample       report raw sample information in perf.data
    stat                gather performance counter information

    -e page-faults; context-switches

## use: top, ps -a
    simpleperf record -p `pidof <prcs>` -g sleep 5
    simpleperf report > rep.txt

    export GOPATH=$TOOLS/go
    go get -u github.com/google/pprof
    export PPROF_BINARY_PATH=$workspace/out/target/product/$board
    pprof -web perf.data

## FlameGraph
- gh: brendangregg/FlameGraph
- gh: iovisor/bcc
- gh: corpaul/flamegraphdiff

    stackcollapse-perf.pl perf.data > out.folded

    perf script -F comm,pid,tid,cpu,time,event,ip,sym,dso,trace

    perf record -F 99 -p <pid> -g -- sleep 60
    perf script | stackcollapse-perf.pl | flamegraph.pl > perf.svg


    bcc/tools/profile.py -dF 99 30 | flamegraph.pl > perf.svg


## final
    export PATH=$PATH:~/tools/FlameGraph
    export PATH=$PATH:$workspace/system/extras/simpleperf/scripts/bin/linux/x86_64
    export PATH=$PATH:$workspace/system/extras/simpleperf/scripts

    rm /sdcard/perf/perf.data; mkdir -p /sdcard/perf; cd /sdcard/perf

    # run test
    simpleperf record -p `pidof <pack>` -g sleep 60

    adb pull /sdcard/perf/perf.data && inferno.sh -sc

    report_sample.py | stackcollapse-perf.pl | flamegraph.pl > a.svg
    report_sample.py | stackcollapse-perf.pl | flamegraph.pl --inverted > inv.svg

## diff
    report_sample.py | stackcollapse-perf.pl > fold1
    difffolded.pl fold1 fold2 | flamegraph.pl > diff1.svg
    difffolded.pl fold2 fold1 | flamegraph.pl --negate > diff2.svg

## extra
[http://jamie-wong.com/post/speedscope/]
[https://github.com/Netflix/flamescope]


# ------------------------------------------------------------------------------
# 16-eklitzke
nice <cmd> # run nicely
type <cmd> # check alias
[https://www.shellcheck.net]
[https://github.com/koalaman/shellcheck/wiki/Checks]


# ------------------------------------------------------------------------------
# 17-tombstone
    /data/tombstones/
    debuggerd64
    crash_dump64

    development/scripts/stack
    # or just
    stack

    stack < tombstone_01
    debuggerd <pid>

    mm system/core/debuggerd/ # builds crasher


# ------------------------------------------------------------------------------
# 18-ftrace
[https://www.youtube.com/watch?v=2ff-7UTg5rE]
    cat /d/tracing/README

    sysctl -w kernel.sysrq=1
    echo l > /proc/sysrq-trigger
    dmesg or Call trace in minicom

    echo workqueue:workqueue_queue_work > /d/tracing/set_event
    cat /d/tracing/trace_pipe

## need:
    ftrace
    mount -t debugfs nodev /sys/kernel/debug
    cat /proc/THE_OFFENDING_KWORKER/stack

    echo 1 > tracing_on
    
    echo 1 >/proc/sys/kernel/stack_tracer_enabled
    cat /d/tracing/stack_trace

    cat available_tracers
    > function function_graph nop
    cat available_filter_functions

    echo 'function_graph' > current_tracer
    cat trace
    
    set_ftrace_filter
    set_ftrace_notrace
    set_ftrace_pid
    set_graph_function
    
    echo schedule > set_ftrace_filter
    echo function > current_tracer
    cat trace
    
    sh -c 'echo $$ > set_ftrace_pid; echo 1 > tracing_on; exec echo hello'
    cat trace

    echo nop > current_tracer
    echo schedule > set_ftrace_filter
    echo 1 > options/func_stack_trace
    echo function > current_tracer
    sleep 1
    echo 0 > tracing_on
    echo 0 > options/func_stack_trace
    cat trace
    
    cat trace_optinos
    echo 1 > options/sym-offset
    echo 1 > options/sym-addr
    echo 0 > options/funcgraph-irqs
    echo 1 > max_graph_depth
    cat trace
    
    lsmod # kern modules
    echo :mod:mac80211 > set_ftrace_filter

## triggers
    echo '!ieee80211_rx_napi:stacktrace' >> set_ftrace_filter
    tail -5 set_ftrace_filter

## events
    sched irq timer exceptions
    event-fork function-fork


# ------------------------------------------------------------------------------
# 19-gdb
[https://www.youtube.com/watch?v=PorfLSr3DDI]

    gcc -g hello.c
    gdb a.out
    > start
    > list
    ctrl+x+a
    > next (n)
    ctrl+l
    > disas
    ctrl+x+2
    > tui reg float
    ctrl+p/n
    > b main
    > b _exit.c:32
    > b 9
    py print(gdb.breakpoints())
    py gdb.Breakpoint('7')
    
    > command 3
    > run
    > command 2
    > record
    > set pagination off
    > run
    > p $pc # prog cnt
    > p $sp # stac ptr
    > x 0x4545464
    > bt
    > reverse-stepi
    > watch (long*) 0x25454
    > reverse-continue



## further
    tbreak                  temporary breakpoint
    rbreak                  reg-ex breakpoint
    break xxx if yyy        conditionally break at xxx if condition yyy holds
    commands                list of commands to be executed when a breakpoint is hit
    silent                  special command to suppress output on breakpoint hit
    save breakpoints        save a list of breakpoints to a script
    save history            save history of executed gdb commands
    call                    call a function in the inferior
    watch -l                watchpoint based on address (location)
    rwatch                  read watchpoint
    info line foo.c:42      show PC for line
    info line * $pc         show line begin/end for current program counter
    thread apply all bt     backtrace for every thread
    dprintf                 dynamic printf
    python: define custom commands by inheriting from gdb.Command class
    python: hook events to invoke python functions using gdb.events.stop.connect
    gcc’s -g and -O are orthogonal


# ------------------------------------------------------------------------------
# strace
    strace ls
    strace -f -e open <cmd> 
    # -e - events: write read connect sendto recvfrom execve
    # -f - subprcs
    # -p <pid>
    # -s 100 - strings strip to 100
    # -o <file> - write to file
    # -y - show file names instead of id

