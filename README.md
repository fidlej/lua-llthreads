
# lua-llthreads fork for Torch7

For details of the project, please go to [Original lua-llthreads](https://github.com/Neopallium/lua-llthreads). This fork is only to replace the CMakeLists.txt file to make it install with torch-pkg.

About
=====

[![travis-ci status](https://secure.travis-ci.org/Neopallium/lua-llthreads.png?branch=master)](http://travis-ci.org/Neopallium/lua-llthreads/builds)

A simple Lua wrapper for pthreads & WIN32 threads.

Each thread gets it's own `lua_State` and there is no shared global state.
The parent thread can pass data to a child thread only as parameters when creating
the child thread.  The child threads can return data back to the parent thread only
when it return (i.e. ends).  The parent needs to call child:join() to get the return
values from a child thread, this call will block until the child thread ends.

The design goals of this module is only provide support for creating new `lua_State`
and running them in a different thread.  This module will not provide any
methods of thread-to-thread data passing between running threads (i.e. no locks, no shared state).

Thread to Thread communication methods
======================================

* The recommend method of passing data between threads is to use [ZeroMQ](http://github.com/Neopallium/lua-zmq).

* Another method is to use sockets library like [LuaSocket](http://w3.impa.br/~diego/software/luasocket) or [Nixio](http://neopallium.github.com/nixio/).

Installation
============

Release 1.2
-----------

lua-llthread 1.2 release:

	$ sudo luarocks install lua-llthreads

Lastest Git Revision
--------------------

With LuaRocks 2.0.4.1:

	$ sudo luarocks install https://github.com/Neopallium/lua-llthreads/raw/master/rockspecs/lua-llthreads-scm-0.rockspec

With CMake:

	$ git clone git://github.com/Neopallium/lua-llthreads.git
	$ cd lua-llthreads ; mkdir build ; cd build
	$ cmake ..
	$ make
	$ sudo make install


Example usage
=============

	local llthreads = require"llthreads"

	local thread_code = [[
		-- print thread's parameter.
		print("CHILD: received params:", ...)
		-- return all thread's parameters back to the parent thread.
		return ...
	]]
	
	-- create detached child thread.
	local thread = llthreads.new(thread_code, "number:", 1234, "nil:", nil, "bool:", true)
	-- start non-joinable detached child thread.
	assert(thread:start(true))
	-- Use a detatched child thread when you don't care when the child finishes.

	-- create child thread.
	local thread = llthreads.new(thread_code, "number:", 1234, "nil:", nil, "bool:", true)
	-- start joinable child thread.
	assert(thread:start())
	-- Warning: If you don't call thread:join() on a joinable child thread, it will be called
	-- by the garbage collector, which may cause random pauses/freeze of the parent thread.
	print("PARENT: child returned: ", thread:join())

	local socket = require"socket"
	socket.sleep(2) -- give detached thread some time to run.


