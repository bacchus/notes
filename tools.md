# ------------------------------------------------------------------------------
# Tools

- 01-vndk
- 02-gdbserver
- 03-debuggerd
- 04-egl
- 05-
- 06-systrace
- 07-gapid
- 08-gcov
- 09-heapchk
- 10-drm
- 11-projects
- 12-includes
- 13-ion
- 14-
- 15-perf
- 16-eklitzke
- 17-tombstone
- 18-ftrace
- 19-gdb

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






