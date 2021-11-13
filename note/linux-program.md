# mmap direct_io

一般说到数据库程序的时候，都会提到会用 direct_io 的方式来写入文件。
但是在 bbolt 和 badger 中都看到了通过 mmap 的方式来读写文件。
以下是 mmap 和 direct_io 的不同。

from: [https://developpaper.com/question/what-is-the-difference-between-direct-io-and-memory-mapping/](https://developpaper.com/question/what-is-the-difference-between-direct-io-and-memory-mapping/)

默认情况下，打开文件的读写方式是写到操作系统的缓存中，由操作系统定时写入到磁盘中。对于数据库这种数据持久化敏感的程序，需要保证可靠的数据持久化。所以会采用 direct_io 的形式不经过操作系统的缓存，这样会导致写入的时间比 buffer_io 长。

mmap 是通过将磁盘的文件直接映射到虚拟内存中，应用程序读取文件的时候减少了磁盘的io，但是实际上虚拟内存写入的时候也是通过 bufferio 来完成的。

badger 和 bbolt 都可以显示调用 msync() 强制虚拟内存的数据更新到磁盘。
