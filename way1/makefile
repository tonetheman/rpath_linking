
# libf is listed first since that has to happen
# for the -L. to work on the later compiles
all : libf.so.4.2.1 junk junkr

# separate target for my own testing
shared : libf.so.4.2.1

# compile look in . need libf.so
# hardcode the rpath to the current directory
junkr : testf.c 
	gcc -o junkr testf.c -lf -L. -Wl,-rpath,${PWD}

junk : testf.c 
	gcc -o junk testf.c -lf -L.

# shared library here
libf.so.4.2.1 : justf.c
	# tell compiler to make an so and put the so name into the binary
	gcc -o libf.so.4.2.1 -shared justf.c -Wl,-soname,libf.so.1

	# gcc -o libf.so.4.2.1 -shared justf.c -Wl,-soname,punkmonkey
	# create some other links
	ln -s libf.so.4.2.1 libf.so
	ln -s libf.so.4.2.1 libf.so.1

clean :
	rm -f ./junk
	rm -f ./junkr
	rm -f libf.so
	rm -f libf.so.1
	rm -f libf.so.4.2.1
