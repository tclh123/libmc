# libmc

[![build status](https://travis-ci.org/douban/libmc.png)](https://travis-ci.org/douban/libmc)

libmc is a memcached client library for Python without any other dependencies in runtime.
It's mainly written in C++ and Cython. libmc can be considered as a drop in replacement for
**libmemcached and [python-libmemcached](https://github.com/douban/python-libmemcached)**.

libmc is developing and maintaining by Douban Inc. Currently, It is working in production environment, 
powering all web traffics in douban.com.
Realtime [benchmark result](https://travis-ci.org/douban/libmc/builds/57124335#L1611) is available on travis.


## Build and Installation

For users:

    pip install libmc

Usage:

```python
import libmc

mc = libmc.Client(['localhost:11211', 'localhost:11212'])
mc.set('foo', 'bar')
assert mc.get('foo') == 'bar'
```

## Under the hood

Under the hood, libmc consists of 2 parts: an internal fully-functional memcached client
implementation in C++ and a Cython wrapper around that implementation.
Dynamic memory allocation and memory-copy are slow, so we tried our best to avoid them.
The `set_multi` command is not natively supported by the [memcached protocol](https://github.com/memcached/memcached/blob/master/doc/protocol.txt).
Some techniques are applied to make `set_multi` command extremely fast in libmc
(compared to some other similiar libraries).


## Configuration

```python
import libmc
from libmc import (
    MC_HASH_MD5, MC_POLL_TIMEOUT, MC_CONNECT_TIMEOUT, MC_RETRY_TIMEOUT
)

mc = libmc.Client(
    ['localhost:11211', 'localhost:11212'], do_split=True,
    comp_threshold=0, noreply=False, prefix=None,
    hash_fn=MC_HASH_MD5, failover=False
)

mc.config(MC_POLL_TIMEOUT, 100)  # 100 ms
mc.config(MC_CONNECT_TIMEOUT, 300)  # 300 ms
mc.config(MC_RETRY_TIMEOUT, 5)  # 5 s
```

- `do_split`: Memcached server will refuse to store value if size >= 1MB,
   if `do_split` is enabled, large value (< 10 MB) will be splitted into
   several blocks. If the value is too large (>= 10 MB), it will not be stored.
   default: `True`
- `comp_threshold`: All kinds of values will be encoded into string buffer.
   If `buffer length > comp_threshold > 0`, it will be compressed using zlib.
   If `comp_threshold = 0`, string buffer will never be compressed using zlib.
   default: `0`
- `noreply`: Whether to enable memcached's `noreply` behaviour.
   default: `False`
- `prefix`: The key prefix. default: `''`
- `hash_fn`: hashing function for keys. possible values:

    + `MC_HASH_MD5`
    + `MC_HASH_FNV1_32`
    + `MC_HASH_FNV1A_32`
    + `MC_HASH_CRC_32`

    default: `MC_HASH_MD5`

    **NOTE:** fnv1_32, fnv1a_32, crc_32 implementations in libmc are per each spec,
    but they're not compatible with corresponding implementions in libmemcached.

- `failover`: Whether to failover to next server when current server is not available.
  default: `False`

- `MC_POLL_TIMEOUT` Timeout parameter used during set/get procedure. (default: `300` ms)
- `MC_CONNECT_TIMEOUT` Timeout parameter used when connecting to memcached server on initial phase. (default: `100` ms)
- `MC_RETRY_TIMEOUT` When a server is not available dur to server-end error. libmc will try to
  establish the broken connection in every `MC_RETRY_TIMEOUT` s until the connection is back to live.(default: `5` s)

**NOTE:** The hashing algorithm for host mapping on continuum is always md5.

## Contributing to libmc

Feel free to send a **Pull Request**. For feature requests or any questions, please open an **Issue**.

For **SECURITY DISCLOSURE**, please disclose the information responsibly by sending an email to security@douban.com directly instead of creating a GitHub issue.


## FAQ

#### Does libmc support PHP?

No. But if you like, you can write a wrapper for PHP based on the C++ implementation.

#### Is Memcached binary protocol supported ?

No. Only Memcached ASCII protocol is supported currently.

#### Why reinventing the wheel?

Before libmc, we're using [python-libmemcached](https://github.com/douban/python-libmemcached),
which is a python extention for [libmemcached](http://libmemcached.org/libMemcached.html).
libmemcached is quite weird and buggy. After nearly one decade, there're still some unsolved bugs.

#### Is libmc thread-safe ?

libmc is a single-threaded memcached client. If you initialize a libmc client in one thread but reuse that in another thread,
a Python Exception `ThreadUnsafe` will raise in Python.


#### Is libmc compatible with gevent?
Yes, with the help of [greenify](https://github.com/douban/greenify), libmc is friendly to gevent.
Read `tests/shabby/gevent_issue.py` for details.


## Acknowledgments

- Thanks to [@fahrenheit2539](https://github.com/fahrenheit2539) and the llvm project for the standalone.
  [SmallVector](http://fahrenheit2539.blogspot.com/2012/06/introduction-in-depths-look-at.html) implementation.
- Thanks to [@miloyip](https://github.com/miloyip) for the high performance [i64toa](https://github.com/miloyip/itoa-benchmark) implementation.
- Thanks to [Ivan Novikov](https://twitter.com/d0znpp) for the research in [THE NEW PAGE OF INJECTIONS BOOK: MEMCACHED INJECTIONS](https://www.blackhat.com/us-14/briefings.html#the-new-page-of-injections-book-memcached-injections).
- Thanks to the PolarSSL project for the md5 implementation.
- Thanks to [@lericson](https://github.com/lericson) for the [benchmark script in pylibmc](https://github.com/lericson/pylibmc/blob/master/bin/runbench.py).
- Thanks to the libmemcached project and some other projects possibly not mentioned here.


## Contributors

- [@mckelvin](https://github.com/mckelvin)
- [@zzl0](https://github.com/zzl0)
- [@windreamer](https://github.com/windreamer)
- [@lembacon](https://github.com/lembacon)
- [@seansay](https://github.com/seansay)


## LICENSE

Copyright (c) 2014-2015, Douban Inc. All rights reserved.

Licensed under a BSD license: https://github.com/douban/libmc/blob/master/LICENSE.txt
