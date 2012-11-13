## Introduction

### Getting Started

In this guide we will use Ubuntu 12.04(10) for bulding and running Tarantool/Box

#### Building From Source

##### On Linux

You must have cmake(>=2.6), git, gcc(>=4.5) or clang, gobjc in order to build server and libreadline with ncurses or termcap to build client.

	sudo aptitude install git cmake
	sudo aptitude install build-essential
	sudo aptitude install gcc-4.7 g++-4.7
	sudo aptitude install gobjc-4.7 gobjc++-4.7
	sudo aptitude install libncurses5-dev libreadline6-dev

	git clone git://github.com/mailru/tarantool.git -b master-stable
	cd tarantool
	cmake . -DENABLE_CLIENT=TRUE
	make

##### Testing

After you succesfully build you first Tarantool it's a great idea to test it. But in order to test Tarantool you'll need Python(>=2.6, <3), PyYAML, python-daemon and python-expect for running tests.

	sudo aptitude install python-daemon python-yaml python-pexpect
	cd test
	./test-run.py 

##### Running

If you want to run Tarantool you must copy a number of files and initialize Tarantool:

	cp tarantool/src/box/tarantool_box vardir/
	cp tarantool/client/tarantool vardir/
	cd vardir

	./tarantool_box --init-storage

#### Installing from Binary Packages

For Ubuntu there's repo with nightly builds.

##### Comment


