
**ioarena** - embedded storage benchmarking
-------------------------------------------

<img src="https://travis-ci.org/pmwkaa/ioarena.svg?branch=master" />

**ioarena** is an utility designed for evaluating performance
of embedded databases.

The goal of this project is to provide a standart and simple
in use instrument for benchmarking, so any database developer or user
can reference to or repeat obtained results.

Benchmarking methods: *set*, *get*, *delete*, *iterate*, *batch*, *crud*

Sync modes: *sync*, *lazy*, *no-sync*

WAL modes: *indef* (per engine default), *wal-on*, *wal-off*

Supported databases: **rocksdb**, **leveldb**, **forestdb**, **upscaledb**, **lmdb**,
**mdbx**, **wiredtiger**, **sophia**, **sqlite3**, **iowow**, **unqlite**

*New drivers or any kind of enhancements are very welcome!*

Usage
-----

```sh
IOARENA (embedded storage benchmarking)

usage: ioarena [hDBCpnkvmlrwic]
  -D <database_driver>
     choices: sophia, leveldb, rocksdb, wiredtiger, forestdb, lmdb, mdbx, sqlite3, iowow, dummy, unqlite
  -B <benchmarks>
     choices: set, get, delete, iterate, batchset(batch_size=80000), batch, crud
  -o <driver option>                 (default: none)
  -m <sync_mode>                     (default: lazy)
     choices: sync, lazy, nosync
  -l <wal_mode>                      (default: indef)
     choices: indef, walon, waloff
  -C <name-prefix> generate csv      (default: (null))
  -p <path> for temporaries          (default: ./_ioarena)
  -n <number_of_operations>          (default: 1000000)
  -k <key_size>                      (default: 16 bytes)
  -v <value_size>                    (default: 32 bytes)
  -c continuous completing mode      (default: no)
  -r <number_of_read_threads>        (default: 0)
     `zero` to use single main/common thread
  -w <number_of_crud/write_threads>  (default: 0)
     `zero` to use single main/common thread
  -i ignore key-not-found error      (default: no)
  -h                                 help

example:
   ioarena -m sync -D sophia -B crud -n 100000000
```

Build
-----

```sh
git clone --recursive https://github.com/pmwkaa/ioarena
```

**cmake** (at least 3.8.2 required by mdbx, cmake 3.8.2 is compatible with 2.8) is required for building.

### gcc dependency (version 11)
```sh
sudo apt install gcc-11 g++-11
# 12 is also ok, but it encounters many warnings that need to be fixed.
```


### mdbx fix
```sh
# mdbx submodule url is changed to https://gitflic.ru/project/erthink/libmdbx/commit/d47eed079e71062ef5dd41a147df060ad13d42b2 (already fixed in this repo)
git submodule sync db/mdbx
cd db/mdbx
git tag v0.11.2 d47eed079e71062ef5dd41a147df060ad13d42b2
```

changes in  `db/mdbx/cmake/utils.cmake`
```
# add `--always` to db/mdbx/cmake/utils.cmake due to this tag is removed in the github url
execute_process(COMMAND ${GIT} describe --tags --always --long --dirty=-dirty
execute_process(COMMAND ${GIT} describe --tags --always --abbrev=0 "--match=v[0-9]*"
```

### rocksdb fix
changes in `cmake/BuildRocksDB.cmake`
```sh
# add zstd, tbb dependency, allready done in this repo.
set (ROCKSDB_LIBRARIES "${PROJECT_BINARY_DIR}/db/rocksdb/librocksdb${CMAKE_SHARED_LIBRARY_SUFFIX}" bz2 z lz4 snappy zstd tbb)
```

changes in `db/rocksdb/Makefile`
```sh
# turn off waning_flags (gcc-11 also need to set this)
ifndef DISABLE_WARNING_AS_ERROR
#   WARNING_FLAGS += -Werror
endif
```

changes in `db/rocksdb/CMakeLists.txt`
```sh
# turn on fail on warnings for gcc-12 (gcc-11 don't need this)
option(FAIL_ON_WARNINGS "Treat compile warnings as errors" OFF)
```

Maybe other dependencies specified in [rocksdb](build/db/rocksdb/INSTALL.md) should also be installed.

### build
To enable a specific database driver, pass -DENABLE\_**NAME**=ON to cmake.
If a specified database is not installed in system, it will be build from db/*name* directory.

```sh
mkdir build
cd build
export CXX=/usr/bin/g++-11
export CC=/usr/bin/gcc-11
cmake .. -DENABLE_ROCKSDB=ON
make
src/ioarena -h
```

Authors
-------

| Name | Contribution |
|---|---|
| Dmitry Simonenko @pmwkaa | Original author. |
| Leonid Yuriev @erthink | Multithreading and isolation from the testcases the interface of a DB drivers cardinally redesigned, it is clear and intelligible now. |
| Egor Zyryanov @er0p | Added support for SQLite, EJDB, Vedis. |
| Adamansky Anton @adamansky | Added support for IOWOW. |
| Alberto Mardegan @mardy | Added support for Upscaledb. |
