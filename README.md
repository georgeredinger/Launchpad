## Launchpad Projects

This are simple Launchpad projects to get stuff going. I think it's somewhat difficult to get functional code in the internet, so I want to do my part by creating simple projects using various techniques.
These projects are tested on the mspgcc uniarch releases.

Sergio Campama 2011 
http://kaipi.me

Made for the TI MSP430 LaunchPad 
http://ti.com/launchpadwiki

Released under the Beerware License 
http://en.wikipedia.org/wiki/Beerware

----

### Dependencies

To compile the binaries, I assume that mspgcc uniarch is installed. If it's not, check out the last part of this README. It's not trivial.

To install the binaries on the Launchpad I use `mspdebug`, which is fairly easy to install, and can be found in [here](http://mspdebug.sourceforge.net/)

For the build system, I use Rake, so you'll need ruby and rake installed. After you have that, you can call `rake` and the project will compile into the final binary. The cool thing about it is that it automatically checks for all *.c files inside the 'src' folder, so if you want to create your own apps, just add the files and it'll work. Also, when compiling, all the 'src' subfolders are added to the header includes, so you can just '#include "anywhere.h"' and it'll also work. Just be sure to not conflict with system header files. Instructions on how to install the binary on the Launchpad can be found running `rake install`. I plan on fixing this to actually install it, but I still have to figure out how to do it without entering the mspdebug console.

----

### Licence

Released under the [Beerware License](http://en.wikipedia.org/wiki/Beerware). That means you can
do whatever the heck you want with it. If you find it useful and you run into me someday, you're
welcome to buy me a beer if you'd like.

----

### MSPGCC 'Uniarch'

Uniarch was an initiative to unify all the work left behind by mspgcc3 and mspgcc4, while also separating the boundaries between the compiler and TI header files.

This instructions are based on the comments inside the patch files of mspgcc.

First, we will need Git, so if you don't have it:

	sudo apt-get install git

Then, in our home, we will create a workspace to build our binaries

	cd
	mkdir -p msp430
	cd msp430

We will need the following dependencies to build mspgcc. I don't really know which ones are really needed, but with this ones it works, and I'm lazy.

	sudo apt-get install patch ncurses-dev build-essential bison flex libgmp3-dev libmpfr-dev libmpc-dev texinfo

First things first, we will need the latest release of the patch files from mspgcc. As of this writing, the latest release was 20110612

	wget http://sourceforge.net/projects/mspgcc/files/mspgcc/mspgcc-20110612.tar.bz2/download -O mspgcc-20110612.tar.bz2
	tar xjf mspgcc-20110612.tar.bz2
	cd mspgcc-20110612

Now we will need to download, patch and install the specific versions of binutils, gcc and gdb that will be used in mspgcc. This instructions are inside the patch files, but I list them here for a better reference
The bin directory will reside in /usr/local/mspgcc for this instructions. If you want to change the location, be sure to change it everywhere there is a prefix path

	mkdir -p sources
	cd sources
 
	wget ftp://ftp.gnu.org/pub/gnu/binutils/binutils-2.21.tar.gz
	tar xzf binutils-2.21.tar.gz
	( cd binutils-2.21 ; patch -p1 < ../../msp430-binutils-2.21-20110612.patch )
	mkdir -p BUILD/binutils
	cd BUILD/binutils
	../../binutils-2.21/configure --target=msp430 --prefix=/usr/local/mspgcc 2>&1 | tee co
	make 2>&1 | tee mo
	sudo make install 2>&1 | tee moi
	cd ../..

That should have the mspgcc binutils running. Now to gcc...

	wget ftp://ftp.gnu.org/pub/gnu/gcc/gcc-4.5.2/gcc-4.5.2.tar.gz
	tar xzf gcc-4.5.2.tar.gz
	( cd gcc-4.5.2 ; patch -p1 < ../../msp430-gcc-4.5.2-20110612.patch )
	mkdir -p BUILD/gcc
	cd BUILD/gcc
	../../gcc-4.5.2/configure --target=msp430 --enable-languages=c,c++ --prefix=/usr/local/mspgcc 2>&1 | tee co
	make 2>&1 | tee mo
	sudo make install 2>&1 | tee moi
	cd ../..

Done with gcc, on to gdb

	wget ftp://ftp.gnu.org/pub/gnu/gdb/gdb-7.2.tar.gz
	tar xzf gdb-7.2.tar.gz
	( cd gdb-7.2 ; patch -p1 < ../../msp430-gdb-7.2-20110103.patch )
	mkdir -p BUILD/gdb
	cd BUILD/gdb
	../../gdb-7.2/configure --target=msp430 --prefix=/usr/local/mspgcc 2>&1 | tee co
	make 2>&1 | tee mo
	sudo make install 2>&1 | tee moi
	cd ../..

Perfect, so all thats left is the microcontroller definitions that are part of msp430mcu and libc that is part of msp430-libc. This files are not downloaded yet. What we have are 2 files *.version that define what version to download. The generic link to download msp430mcu is http://sourceforge.net/projects/mspgcc/files/msp430mcu/msp430mcu-YYYYMMDD.tar.bz2 and the one for msp430-libc https://sourceforge.net/projects/mspgcc/files/msp430-libc/msp430-libc-YYYYMMDD.tar.bz2 . YYYMMDD are replaced by the contents of msp430mcu.version and msp430-libc.version

	wget http://sourceforge.net/projects/mspgcc/files/msp430mcu/msp430mcu-20110613.tar.bz2
	tar xjf msp430mcu-20110613.tar.bz2
	cd msp430mcu-20110613

Now, I don't know why, but the regular way doesn't work on my setup. So, for the sake of getting this done, edit scripts/install.sh and place this on the top, just after the comments

	MSP430MCU_ROOT=${2}

Then run

	sudo ./scripts/install.sh /usr/local/mspgcc $(pwd)
	cd ..
 
Finally we need to compile and install libc for mspgcc

	wget https://sourceforge.net/projects/mspgcc/files/msp430-libc/msp430-libc-20110612.tar.bz2
	tar xjf msp430-libc-20110612.tar.bz2
	cd msp430-libc-20110612/src

Now, this is also weird, the regular way would be to do make; make PREFIX=/usr/local/mspgcc , but for some reason, make doesn't work for me. So, edit Makefile and change lines 23, 24, and 25 to

	override CC = ${bindir}/msp430-gcc
	override AS = ${bindir}/msp430-gcc -x assembler-with-cpp
	override AR = ${bindir}/msp430-ar

Some of you may say "But you didn't setup the PATH yet, so obviously it's not gonna work". Well, the first time I installed mspgcc uniarch, I DID setup PATH to find the binaries, but it still didn't work and I had to do this same changes. So, when you're done changing Makefile, do:

	make PREFIX=/usr/local/mspgcc
	sudo make PREFIX=/usr/local/mspgcc install

Finally, we will setup the PATH in .bashrc . Add this lines to the end of ~/.bashrc

	#MSPGCC binary path
	export PATH=$PATH:/usr/local/mspgcc/bin

Now source .bashrc to be able to call `msp430-gcc` right away. 

	source ~/.bashrc

And that's it! You can compile the projects now. 

###MSPDEBUG USB permissions

Now, having mspdebug installed with the instructions from their webpage, connect your Launchpad to Ubuntu and run `mspdebug rf2500`. If it doesn't work, don't panic, follow the next steps (based on this [article](http://karuppuswamy.com/wordpress/2010/10/07/debugging-ez430-chronos-with-mspdebug-tool-in-ubuntu-linux/).).

First, as root, add `ATTRS{idVendor}=="0451", ATTRS{idProduct}=="f432", MODE="0660", GROUP="plugdev"` to `/etc/udev/rules.d/71-persistent-msp430.rules`. Then run this commands:

	sudo addgroup YOUR_USERNAME plugdev
	sudo restart udev
	
Now unplug and replug the Launchpad and try again and it should work.