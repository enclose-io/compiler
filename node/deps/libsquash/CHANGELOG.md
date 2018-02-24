# Libsquash Changelog

## v0.8.0

- allow creating files inside an existing folder of memfs
  - removes the temporary directory and files at exit
- produce an executable `squash_sample` when `BUILD_SAMPLE` in cmake
  - let CI discover linking errors earlier about the sample
- intercept `CreateProcessW`
  - unsets `lpCurrentDirectory` when it was set to `__enclose_io_memfs__` paths
- intercept `SetCurrentDirectoryW`, `GetCurrentDirectoryW`
- implement `enclose_io_mkdir` for Windows
  - intercept `_wmkdir`
- intercept `CreateFileW()` with writing
  - redirect `CreateFileW()` with writing inside the memfs to a temporary directory
  - removes the temporary directory and files at exit

Translations in Chinese:
- 允许在虚拟的内存文件系统中创建文件夹
  - 此种请求会被转发到磁盘的临时文件夹，并在退出时删除运行时所创建的文件夹及其内容
- 让 cmake 在 `BUILD_SAMPLE` 的时候生成一个 `squash_sample` 可执行文件
  - 这可以让 CI 今早发现 sample 代码链接阶段的问题
- 劫持 `CreateProcessW` 系统调用
  - 当第八个参数 `lpCurrentDirectory` 被设定为 `__enclose_io_memfs__` 路径时清空这个参数，以防出错
- 劫持 `SetCurrentDirectoryW`, `GetCurrentDirectoryW` 系统调用
- 为 Windows 实现 `enclose_io_mkdir` 函数
  - 为 Windows 劫持 `_wmkdir` 系统调用
- 劫持 `CreateFileW()` 系统调用的写的情况
  - 即允许在虚拟的内存文件系统中写文件，或在虚拟的内存文件系统中创建的文件夹内写文件
  - 此种请求会被转发到磁盘的临时文件夹，并在退出时删除运行时所写的文件

## v0.7.0

- test ifndef __USE_XOPEN_EXTENDED

## v0.6.0

- add `enclose_io_ifextract(const char* path, const char* ext_name)`
- add `enclose_io_if(const char* path)`
- fix a NULL-dereferencing in `EncloseIOFindFirstFileHelper`
- convert macro `ENCLOSE_IO_DOS_RETURN` to a function
- rename `squash_global_fdtable_mutex` to `squash_global_mutex`
- intercept `mkdir()`
  - redirect `mkdir()` inside the memfs to a temporary directory
  - removes the temporary directory and files at exit
- intercept `open()` with `O_CREAT`
  - redirect `open()` with `O_CREAT` inside the memfs to a temporary directory
  - removes the temporary directory and files at exit

Translations in Chinese:
- 添加新 API `enclose_io_ifextract(const char* path, const char* ext_name)`
- 添加新 API `enclose_io_if(const char* path)`
- 修复 `EncloseIOFindFirstFileHelper` 中的一个空指针解引用
- 将宏 `ENCLOSE_IO_DOS_RETURN` 改写为函数调用
- 将全局锁 `squash_global_fdtable_mutex` 重命名为 `squash_global_mutex`
- 劫持 `mkdir()` 系统调用
  - 对于那些试图在只读的内存文件系统中进行 `mkdir()` 的系统调用转发到临时文件夹
  - 让程序退出时删除此临时文件夹和其中的所有文件
- 劫持带有 `O_CREAT` flag 的 `open()` 系统调用
  - 将那些试图在只读的内存文件系统中进行带有 `O_CREAT` flag 的 `open()` 系统调用转发到临时文件夹
  - 让程序退出时删除此临时文件夹和其中的所有文件

## v0.5.0

- deprecate use of swprintf() to ease compiling on Windows
- interpret system calls to access()

## v0.4.0

- Fix dot-dot in opening `/__enclose_io_memfs__/lib/../package.json`
  - https://github.com/pmq20/node-compiler/issues/24

## v0.3.0

- Implemented the following dirent APIs.
  - squash_opendir(fs, filename)
  - squash_closedir(dirp)
  - squash_readdir(dirp)
  - squash_telldir(dirp)
  - squash_seekdir(dirp, loc)
  - squash_rewinddir(dirp)
  - squash_dirfd(dirp)
  - squash_lstat(fs, path, buf)
  - squash_readlink(fs, path, buf, bufsize)
  - squash_scandir(fs, dirname, namelist, select, compar)
- Added better support for symbolic links
- Added locks to handle concurrent situations (locking four caches and one global vfd table)
- Intercept more API's like pread and readv on Unix
- Intercept more API's like IODeviceIoControl and CreateIoCompletionPort on Win32
- Added better support for DOS errno and errno on Windows
- Support symbolic paths like /a/b/../../c/d and /a/b/c/d/..
- Enlarge buffer to support longer path names
- Fix a buffer overflow found in wcstombs functions
- Dynamically change some absolute symbolic paths into relative ones via `root_alias`

Translations in Chinese:
- 增加了如下新 API 的实现
  - squash_opendir(fs, filename)
  - squash_closedir(dirp)
  - squash_readdir(dirp)
  - squash_telldir(dirp)
  - squash_seekdir(dirp, loc)
  - squash_rewinddir(dirp)
  - squash_dirfd(dirp)
  - squash_lstat(fs, path, buf)
  - squash_readlink(fs, path, buf, bufsize)
  - squash_scandir(fs, dirname, namelist, select, compar)
- 添加了对符号链接更好的支持
- 添加了对并发的加锁控制（对四个 SquashFS 缓存加锁、一个公共文件描述符表加锁）
- 添加了更多 API 如 pread 和 readv
- 添加了 IODeviceIoControl 和 CreateIoCompletionPort 等 Win32 API 等
- 添加了对 DOS errno 和 errno 的更完备的处理
- 跟踪链接的这个地方我增加了处理 /a/b/../../c/d 和 /a/b/c/d/.. 这种带双点的情况
- 扩大了缓冲区大小解决了路径名太长的问题
- 修复缓冲区溢出（wcstombs 等库函数的第三个参数限制了最大转换长度，然而“If max bytes are successfully translated, the resulting string stored in dest is not null-terminated.”，造成了后续对 dest 访问的缓冲区溢出）
- 通过 `root_alias` 统一对未出项目根部的路径的绝对路径链接在运行时动态改为相对路径（这个地方有点 ugly hack 了，更好的做法是在 mksquashfs 的时候就做这个操作，而不是运行时，这里以后改进）

## v0.2.0

Implemented the virutal file descriptors and the following new API's:

- squash_stat(fs, path, buf)
- squash_fstat(vfd, buf)
- squash_open(fs, path)
- squash_close(vfd)
- squash_read(vfd, buf, nbyte)
- squash_lseek(vfd, offset, whence)

Translations in Chinese:
实现了虚拟文件描述符表(vfd, virtual file descriptor)，
虚拟文件描述符与正常磁盘文件的文件描述符 (fd) 可以和平共处，
这使得上层运行时可以无缝接入对内存文件系统的访问，而不需要原有代码的调用风格。
新开发了以上 API 实现，实现与 open、lseek、read 等系统调用的风格相同的调用接口。
为基础 API、stat 系列 API、虚拟文件描述符表系列 API 全部编写了测试。

## v0.1.0

Added Tests for basic APIs from squashfuse; tagged the initial release.
