% Gentle Introduction to Tarantool/Box
% Eugine Blikh

## Introduction

### Getting Started

In this guide we will use Ubuntu 12.04(10) for bulding and running Tarantool/Box

#### Building From Source

##### Compiling

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

	mkdir vardir
	cp tarantool/src/box/tarantool_box vardir
	cp tarantool/client/tarantool vardir
	cd vardir
	wget https://raw.github.com/bigbes92/gentle_introduction_to_tarantool/master/tarantool.cfg
	./tarantool_box --init-storage

And now simply:
	
	./tarantool_box

#### Installing from Binary Packages

##### Installing

There are packages in main tree repos for Ubuntu >=11.10

	sudo aptitude install tarantool tarantool-client

Or there's Ubuntu there's repo with nightly builds for Precise.

	wget http://tarantool.org/dist/public.key
	sudo apt-key add ./public.key
	release=`lsb_release -c -s`
	echo "deb http://tarantool.org/dist/ubuntu/ $release main" | sudo tee -a /etc/apt/sources.list.d/tarantool.list
	echo "deb-src http://tarantool.org/dist/ubuntu/ $release main" | sudo tee -a /etc/apt/sources.list.d/tarantool.list
	sudo apt-get update
	sudo apt-get install tarantool tarantool-client

##### Running

Create directory, run storage initialization aaaaaand it's done!

	mkdir vardir
	cd vardir
	wget https://raw.github.com/bigbes92/gentle_introduction_to_tarantool/master/tarantool.cfg
	tarantool_box --init-storage

And now simply:

	tarantool_box

#### Bugs

If you'll have some - feel free to read logs - they're full of important information.
If you still can't understand problem - you can write to #tarantool channel on irc.freenode.net or to [tarantool-ru](https://groups.google.com/forum/?fromgroups#!forum/tarantool-ru)
And if it's still not helps or you think that it's a bug - publish tickets to [launchpad](https://bugs.launchpad.net/tarantool)

### Basics

Let's start:

* Tarantool/Box don't have such thing as "Database", but instead it has Spaces. Space is simple pack of Tuples and there can be up to 255 ones per Tarantool.
* Tarantool/Box's Tuple is a simple tuple containing various number of Fields. Key Fields can be 32bit integer, 64bit integer or string now, but we'll discuss em later.
* Tarantool/Box has Schema, that's defined in tarantool.cfg file. Schema contains all spaces and indexes.
* Tarantool/Box has Snapshots, Write Ahead Logs(WAL) and Replication for better resilency. 

#### Understanding configs.

Open base file, that you've wget'ed. What'll we see? 

* `slab_alloc_arena = 0.5` - All tuples in Tarantool are stored in this area(with SLAB allocator).
* `pid_file = "box.pid"` - File `box.pid` now stores pid of Tarantool.
* `logger = "cat - >> tarantool.log"` - Command that is used for logging.
* `primary_port = 33013` - Base port for all Tarantool connectors.
* `secondary_port = 33014` - RO port.
* `admin_port = 33015` - Port for administrative connections.
* `rows_per_wal = 2000` - Number of ops stored per one WAL file.

Then we see simple defining of one space with one key index.

Let's enable this space:

	space[0].enabled = 1

Then tell Tarantool that first type of index will be tree-based:

	space[0].index[0].type = "TREE"

Next step is make keys Unique:

	space[0].index[0].unique = 1

Hereafter we must say that first(and only) part of our key is string and located in first field of Tuple:

	space[0].index[0].key_field[0].fieldno = 0
	space[0].index[0].key_field[0].type = "STR"

That's all. Nothing hard or impossible :)

#### Indexes Types

There are only two indexes types now: TREE or HASH.

When you use trees you can operate with sorted data, but with hashes you can't(e.g. multipart keys and box.select_range have undefined behavior on HASH). Hash is faster than tree, because looking and inserting in hashes has complexity is O(1), but with tree it's O(log(n)), where n is number of elements. Also trees consumes more space, than hashes.

#### Writing Schemas and alternative syntax Schemas.

When defining schemas you must remember few rules:

* One space must be configured.
* Each space must have one unique index.
* You need to restart server to change schema.
* HASH indexes may cover only one field and only unique.

Base syntax for defining schemas:

	space[0].enabled = 1
	space[0].index[0].type = "TREE"
	space[0].index[0].unique = 1
	space[0].index[0].key_field[0].fieldno = 0
	space[0].index[0].key_field[0].type = "STR"

But you also can define this space as:

	space[0] = {
		enabled = 1,
		index = [
			{
				type = "TREE"
				key_field = [
					{
						fieldno = 0,
						type = "STR"
					}
				]
			}
		]
	}

It's useful, when defining very very big spaces.

If you want to set max number of fields in one space, you may set cardinality: `space[0].cardinality = 5`

#### Using client and SQL-like syntax.

Let's start the client: `$ ./tarantool`. We'll get something like this: `localhost>`.

Type `help` to see the list of all commands.

Let me show you some basic commands:

* `show info` - shows us some basic information as version, uptime, pid and etc.

* `show configuration` - shows full configuration for server.

* `show slab` - show free/full space in SLAB.

* `show stat` - show some statistics in tarantool.

* `save snapshot` - save snapshot. 

* `reload configuration` - reload configuration.

Also some base commands are(SQL-like notation):

* `insert into (space) values (tuples)`

* `replace into (space) values (tuples)`

* `update (space) set (keys)`

* `delete from (space) where (keys)`

* `select * from (space) where (keys)`

* `call (function)(tuple)`

Let's start Tarantool with this space configuration:

	space[0].enabled = 1
	space[0].index[0].type = "TREE"
	space[0].index[0].unique = 1
	space[0].index[0].key_field[0].fieldno = 0
	space[0].index[0].key_field[0].type = "NUM"
	space[0].index[1].type = "TREE"
	space[0].index[1].unique = 0
	space[0].index[1].key_field[0].fieldno = 1
	space[0].index[1].key_field[0].type = "STR"


Examples:

##### Insert and Delete example:

	localhost> insert into t0 values (1,2,3,4)
	Insert OK, 1 rows affected
	localhost> insert into t0 values (1,2,3,4)
	Insert ERROR, Tuple already exists (ER_TUPLE_FOUND)
	localhost> delete from t0 where k0=1
	Delete OK, 1 rows affected
	localhost> insert into t0 values (1, 'hello')
	Insert OK, 1 rows affected
	localhost> insert into t0 values (2, 'i')
	Insert OK, 1 rows affected
	localhost> insert into t0 values (3, 'love')
	Insert OK, 1 rows affected
	localhost> insert into t0 values (4, 'you')
	Insert OK, 1 rows affected
	localhost> insert into t0 values (5, 'can')
	Insert OK, 1 rows affected
	localhost> insert into t0 values (6, 'you')
	Insert OK, 1 rows affected

##### Select example:

	localhost> select * from t0 where k0=1
	Select OK, 1 rows affected
	[1, 'hello']
	localhost> select * from t0 where k1='you'
	Select OK, 2 rows affected
	[4, 'you']
	[6, 'you']
	localhost> select * from t0 where k1='you' limit 1
	Select OK, 1 rows affected
	[4, 'you']
	localhost> select * from t0 where k1='you' or k1='i'
	Select OK, 3 rows affected
	[4, 'you']
	[6, 'you']
	[2, 'i']

##### Update example:

	localhost> select * from t0 where k0=2
	Select OK, 1 rows affected
	[2, 'i']
	localhost> update t0 set k1=splice(k1,1,0,' love') where k0=2
	Update OK, 1 rows affected
	localhost> select * from t0 where k0=2
	Select OK, 1 rows affected
	[2, 'i love']





### Understanding SQL-Like Syntax.

#### Insert
#### Select
#### Update
#### Delete
#### Call

