
# interesting load library path things

Saw this post: https://stoppels.ch/2022/08/04/stop-searching-for-shared-libraries.html

You should read it. I am just repeating a lot here.

Basically there is a thing called RPATH that at least in linux gives you some interesting powers.

You can end up hard coding the library paths into your final binary! And it can be good really.

Normally when you use shared libraries when you run the load time linking has to know where the shared libraries are located on disk.

If you do not you get the ever present
```
error while loading shared libraries: XXXXX: cannot open shared object file: No such file or directory
```

Normally I would fix this by looking at the main file using ```ldd``` to figure out what it was dynamically linking.

It never hit me that you can change the names inside of the binary. Not sure why... it seems obvious now.

## the first way

In the directory way1

There is a single shared object libf that has a single function that is shared. There is a test file that I compile to two different binaries: junk and junkr.

I compile libf like this in the way1 directory

```makefile
libf.so.4.2.1 : justf.c
	# tell compiler to make an so and put the so name into the binary
	gcc -o libf.so.4.2.1 -shared justf.c -Wl,-soname,libf.so.1
	# create some other links
	ln -s libf.so.4.2.1 libf.so
	ln -s libf.so.4.2.1 libf.so.1
```

The only oddness is setting the soname in the shared object. Which is used when you link against it.

So when I run this command to make the binary

```makefile
junk : testf.c 
	gcc -o junk testf.c -lf -L.
```

You will get a binary that during the build is looking for libf.so and it is using the current directory tacked onto the search path.

But the binary ```junk``` will still fail when you try to run it! Because in order to run ```junk``` needs a shared object file named libf.so.1 and you can verify that with this command

```bash
agcc@DESKTOP-REHHCAH:~/rpath/way1$ ldd ./junk
        linux-vdso.so.1 (0x00007ffeb81f1000)
        libf.so.1 => not found
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fdac7db0000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fdac7fb4000)
```

So in this case you need to give linux a way to find that shared object.
```
LD_LIBRARY_PATH=. && ./junk
```

Then just to make sure I am back to my base state I reset LD_LIBRARY_PATH again explicitly.
```
export LD_LIBRARY_PATH=
```

If you look at the next binary we see a different story

## the 2nd way

Now we are going to build the same binary with a little more information.
```makefile
junkr : testf.c 
	gcc -o junkr testf.c -lf -L. -Wl,-rpath,${PWD}
```

This time we are going to include rpath and hard code it to the current directory. When you do that you can run run ```./junkr``` directly and you do not need to set the LD_LIBRARY_PATH at all!

Lets look at ldd and see what it shows
```bash
agcc@DESKTOP-REHHCAH:~/rpath/way1$ ldd junkr
        linux-vdso.so.1 (0x00007fffd09eb000)
        libf.so.1 => /home/agcc/rpath/way1/libf.so.1 (0x00007f5c5129c000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f5c5109f000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f5c512a8000)
```

This time libf is now hardcoded all the way to my directory. So linux knows exactly what to do here to find the shared object files.

### hardcoding ... is this ok?
I think it can be but you have to understand what you have done. If these binaries are destined to live in a certain spot for their entire life then I think a hardcode is great.

This also means that you could move the final binary (junkr in this case) anywhere on the file system as long as the hardcode is still available at an operating system level.

### another subtle point or it was for me
The name that ldd is using is the SAME soname we gave it when we made the shared object.
```
	gcc -o libf.so.4.2.1 -shared justf.c -Wl,-soname,libf.so.1
```
So in this case the soname is libf.so.1 and that is what shows in the ldd lines for the linked binary junk and junkr.

Just to drive this point home I will change the makefile and then run clean and try it again
```makefile
libf.so.4.2.1 : justf.c
	# tell compiler to make an so and put the so name into the binary
	gcc -o libf.so.4.2.1 -shared justf.c -Wl,-soname,punkmonkey
	# create some other links
	ln -s libf.so.4.2.1 libf.so
	ln -s libf.so.4.2.1 libf.so.1
```
Notice the changed line our soname is now punkmonkey

I did NOT change the compiles for the two binaries but when we look at ldd it shows the point (hopefully).
```
agcc@DESKTOP-REHHCAH:~/rpath/way1$ ldd junk
        linux-vdso.so.1 (0x00007fff525d6000)
        punkmonkey => not found
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f1bd6011000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f1bd6215000)
agcc@DESKTOP-REHHCAH:~/rpath/way1$ ldd junkr
        linux-vdso.so.1 (0x00007ffd00da8000)
        punkmonkey => not found
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fad4f1a5000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fad4f3a9000)
```
In both cases the command now shows that the binaries are looking for something called punkmonkey. That name comes from the soname that is hard coded at build/link time for that shared object.




