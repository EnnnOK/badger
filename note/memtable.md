# memtable

memtable 中的 logFile 持有的 Mmapfile 中的 buff 是通过 mmap 映射到内存的数据

修改数据库的动作变为 request 对象，直接追加到 file 里面

