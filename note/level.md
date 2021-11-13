# level

# levelController levelHandler 
用于管理每个层的数据

## 索引

## Compaction
db Open 之后就会一直在协程中不断 compaction 直到 db close



## level_controller
初始化的时候，将 manifest 文件中的文件全部加载到内存中，然后初始化 level_handler 为每一层分配一个 level_handler。

tablemanifest 里面有这个文件的所在层

## level_handler
level_handler 对应一个层的管理，持有一个层的所有的文件


fillTables

fillMaxLevelTables

收集最大层的 table

把 table 排序，然后从小到大，找到符合条件的 table
把找到的 table 在去找 bot table



## levelCompactStatus
每一层的压缩状态，包含了这一层的所有 table 的 keyRange。

## compactStatus
包含了每一层的 levelCompactStatus，被 levelController 持有。


compaction 的时候会丢弃掉表里面的数据，通过dropPrefixes 来判断。 skip key 也会跳过

## compactDef?


### subcompact() 
compact tables 然后返回一个新的 table 
discardTs 丢弃的时间戳。
遍历之间的所有 table，把 table 里面的元素一个一个全部放在新的 table 中。
如果新的 table 的 keyrange 在下一个表中超过了 10 个，就用新的表。

### levelTargets()


compact 会从上到下保留版本，如果压缩的层被

compact 的时候保留指定数量的版本 默认只保留一个版本

### 
levelController 和 levelHandler 

levelController 是全部的层控制器，用来执行压实动作，里面有每一层的 levelHandler。
levelHandler 是每一层的管理器，持有当前层的所有 table 对象。

当数据库打开时候，会初始化 levelController 对象，然后 levelController 会调用 startCompact() 开启协程 50ms一次不断执行压实操作。每个压实的协程会带有一个id编号。特殊的 `id=0` 的压缩是用来压实第0层的，`id=2` 是用来压实最大层的。
压实操作会从所有的层选出一个优先级 pickCompactLevels()，优先级的计算方法参考：[https://github.com/facebook/rocksdb/wiki/Leveled-Compaction](https://github.com/facebook/rocksdb/wiki/Leveled-Compaction) 给每层计算好优先级之后，
每次压实只会挑选优先级最高的一层进行压实，只要压实操作进行了一次之后就会返回。然后调用 doCompact() 来进行压实。

doCompact()
会创建出来 compactDef 对象， compactDef 对象是压实动作执行的的参数，单元测试用例里面都是通过 compactDef 对象来控制压实的。

compactDef 中有当前层的 levelHandler 以及下一层的 levelHandler，然后调用 filltables 初始化 top,bot 的 table，基本上 top table 就是当前层的所有table,bot 是下一层的所有 table, 以及当前层的 keyRange 和下一层的 keyRange。
compactDef 对象中的所有参数构建完毕之后，调用 runCompactDef() 函数执行压实。在 runCompactDef() 之中调用 compactBuildTables() 然后调用 subcompact() 把所有的 bot,top 中的 table 的元素合并在一起生成新的 table。当中会根据数据库设置保留数量的历史版本。构造产生新的 table 之后，返回。

subcompact 就是将所有的 table 压实产生新的 table。其中有一些比较特殊的处理，比如要求新的 table 的 keyrange 的 table 分布不能超过 10 个。exceedsAllowedOverlap。如果元素过期了或者被丢弃了，也会被保留下来，但是会记录在 table 的 staleDataSize 中。

第一层的特殊处理
当压实的层是第一层的时候，压实的下一层为 baseLevel, baseLevel 是从顶到底不为空的一个。
只将第一层的 table 压实或者将第一层的table 压实到 baselLevel 上。
