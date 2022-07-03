# Qt-Input-UI

THE CORRECT GCC COMPILE LINE IS:

```

g++ /home/opensuse/libuiohook/include/test.cpp -o test_hook  -Wl,-rpath,'$ORIGIN': libuiohook.so.1.2.0 -L/usr/lib64 -lX11 -lXtst -lxkbcommon-x11 -lxkbcommon -lX11-xcb -lxcb -lxkbfile -lXt -lX11 -lXtst -lxkbcommon-x11 -lxkbcommon -lX11-xcb -lxcb -lxkbfile -lXt -lXinerama

```

And the GIMP layer position offset code is:

```
def dumpGroup(g):
    for l in g.layers:
        if isinstance(l,gimp.GroupLayer):
            dumpGroup(l)
        else:
            print l.name,l.offsets

image=gimp.image_list()[0]
dumpGroup(image)
```
