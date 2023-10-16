In case your Raspberry Pi runs an OS image where python is not installed and you want to change that, here's a simple a guide to help you down the path:

* First you need to download the version of Python you want to install: 
	`wget https://www.python.org/ftp/python/3.x.x/Python-3.x.x.tar.xz`
	**Note: Make sure that the Python and the cross-compiler versions are compatible** 
* Extract the files and gt into the directory:
	`tar -xvf Python-3.x.x.tar.xz && cd Python-3.x.x.tar.xz`
* Before configuring and compiling the exec, you will need the readelf command to be linked to the equivalent binary of the cross compiler you use. Python uses this command to check the dependencies of a binary or shared library. Hence, it needs to be able to read the structure of your target machine (32-bit Raspberry Pi).
	* First, remove the already existing link to you 64-bit host device:
		`sudo rm /usr/bin/readelf`
	* Then, link the readelf binary to the one included in your cross-compiler directory:
		`sudo ln -s /opt/cross-pi-gcc-10.3.0-0/arm-linux-gnueabihf/bin/arm-linux-gnueabihf-readelf /usr/bin/readelf`
		**Note that you need to replace the above path of the cross-compiler with the one you have installed the cross compiler to**
	* You can check that the previous command is successful by running one of the commands:
		`readlink -f $(which readelf)` or `ls -l /usr/bin/readelf`
	* You can later reverse this change -after finishing with python's installation- with:
		`sudo apt-get install --reinstall binutils`
* Now you have to configure Python for cross-compilation. You may need to add some options to finish that successfully. These will depend on your system characteristics (if the directories exist in your filesystem). For instance:
	`./configure --build=x86_64-linux-gnu --host=arm-linux-gnueabihf CC=/opt/cross-pi-gcc-10.3.0-0/bin/arm-linux-gnueabihf-gcc --disable-ipv6 ac_cv_file__dev_ptmx=yes ac_cv_file__dev_ptc=no `
	**Note: It's mandatory to add the CC option to declare which compiler to use to build Python** 
* Now you can build Python:
	`make` 
	You can check that Python is built for a 32-bit system by running the command:
	`file python`
* And install:
	`make install DESTDIR=/path/to/place/the/exec/dir`
	
##### Congratulations, you are ready to copy the directory containing the executable files to your Raspberry Pi and call them to run your scripts. 

Of course, you can then link the path to a command (e.g. python) to avoid using the whole path every time:
`sudo ln -s /usr/local/bin/python3.9 python