title: 数据库优化之我见
date: 2012-09-22 23:14:02
tags:
---


1、调整数据结构的设计。这一部分在开发信息系统之前完成，程序员需要考虑是否使用ORACLE数据库的分区功能，对于经常访问的数据库表是否需要建立索引等。 



2、调整应用程序结构设计。这一部分也是在开发信息系统之前完成，程序员在这一步需要考虑应用程序使用什么样的体系结构，是使用传统的Client/Server两层体系结构，还是使用Browser/Web/Database的三层体系结构。不同的应用程序体系结构要求的数据库资源是不同的。 



3、调整数据库SQL语句。应用程序的执行最终将归结为数据库中的SQL语句执行，因此SQL语句的执行效率最终决定了ORACLE数据库的性能。ORACLE公司推荐使用ORACLE语句优化器（Oracle Optimizer）和行锁管理器（row-level manager）来调整优化SQL语句。 



4、调整服务器内存分配。内存分配是在信息系统运行过程中优化配置的，数据库管理员可以根据数据库运行状况调整数据库系统全局区（SGA区）的数据缓冲区、日志缓冲区和共享池的大小；还可以调整程序全局区（PGA区）的大小。需要注意的是，SGA区不是越大越好，SGA区过大会占用操作系统使用的内存而引起虚拟内存的页面交换，这样反而会降低系统。 



5、调整硬盘I/O，这一步是在信息系统开发之前完成的。数据库管理员可以将组成同一个表空间的数据文件放在不同的硬盘上，做到硬盘之间I/O负载均衡。 



6、6、调整操作系统参数，例如：运行在UNIX操作系统上的ORACLE数据库，可以调整UNIX数据缓冲池的大小，每个进程所能使用的内存大小等参数。 



实际上，上述数据库优化措施之间是相互联系的。ORACLE数据库性能恶化表现基本上都是用户响应时间比较长，需要用户长时间的等待。但性能恶化的原因却是多种多样的，有时是多个因素共同造成了性能恶化的结果，这就需要数据库管理员有比较全面的计算机知识，能够敏感地察觉到影响数据库性能的主要原因所在。另外，良好的数据库管理工具对于优化数据库性能也是很重要的。 



ORACLE数据库性能优化工具 



常用的数据库性能优化工具有： 



1、ORACLE数据库在线数据字典，ORACLE在线数据字典能够反映出ORACLE动态运行情况，对于调整数据库性能是很有帮助的。 



2、操作系统工具，例如UNIX操作系统的vmstat，iostat等命令可以查看到系统系统级内存和硬盘I/O的使用情况，这些工具对于管理员弄清出系统瓶颈出现在什么地方有时候很有用。 



3、SQL语言跟踪工具（SQL TRACE FACILITY），SQL语言跟踪工具可以记录SQL语句的执行情况，管理员可以使用虚拟表来调整实例，使用SQL语句跟踪文件调整应用程序性能。SQL语言跟踪工具将结果输出成一个操作系统的文件，管理员可以使用TKPROF工具查看这些文件。 



4、ORACLE Enterprise Manager（OEM），这是一个图形的用户管理界面，用户可以使用它方便地进行数据库管理而不必记住复杂的ORACLE数据库管理的命令。 



5、EXPLAIN PLAN——SQL语言优化命令，使用这个命令可以帮助程序员写出高效的SQL语言。 



ORACLE数据库的系统性能评估 



信息系统的类型不同，需要关注的数据库参数也是不同的。数据库管理员需要根据自己的信息系统的类型着重考虑不同的数据库参数。 



1、在线事务处理信息系统（OLTP），这种类型的信息系统一般需要有大量的Insert、Update操作，典型的系统包括民航机票发售系统、银行储蓄系统等。OLTP系统需要保证数据库的并发性、可靠性和最终用户的速度，这类系统使用的ORACLE数据库需要主要考虑下述参数： 



l     l     数据库回滚段是否足够？ 



l     l     是否需要建立ORACLE数据库索引、聚集、散列？ 



l     l     系统全局区（SGA）大小是否足够？ 



l     l     SQL语句是否高效？ 



2、数据仓库系统（Data Warehousing），这种信息系统的主要任务是从ORACLE的海量数据中进行查询，得到数据之间的某些规律。数据库管理员需要为这种类型的ORACLE数据库着重考虑下述参数： 



l     l     是否采用B*-索引或者bitmap索引？ 



l     l     是否采用并行SQL查询以提高查询效率？ 



l     l     是否采用PL/SQL函数编写存储过程？ 



l     l     有必要的话，需要建立并行数据库提高数据库的查询效率 



SQL语句的调整原则 



SQL语言是一种灵活的语言，相同的功能可以使用不同的语句来实现，但是语句的执行效率是很不相同的。程序员可以使用EXPLAIN PLAN语句来比较各种实现方案，并选出最优的实现方案。总得来讲，程序员写SQL语句需要满足考虑如下规则： 



1、尽量使用索引。试比较下面两条SQL语句： 



语句A：SELECT dname, deptno FROM dept WHERE deptno NOT IN  



(SELECT deptno FROM emp); 



语句B：SELECT dname, deptno FROM dept WHERE NOT EXISTS 



(SELECT deptno FROM emp WHERE dept.deptno = emp.deptno); 



这两条查询语句实现的结果是相同的，但是执行语句A的时候，ORACLE会对整个emp表进行扫描，没有使用建立在emp表上的deptno索引，执行语句B的时候，由于在子查询中使用了联合查询，ORACLE只是对emp表进行的部分数据扫描，并利用了deptno列的索引，所以语句B的效率要比语句A的效率高一些。 



2、选择联合查询的联合次序。考虑下面的例子： 



SELECT stuff FROM taba a, tabb b, tabc c 



WHERE a.acol between :alow and :ahigh 



AND b.bcol between :blow and :bhigh 



AND c.ccol between :clow and :chigh 



AND a.key1 = b.key1 



AMD a.key2 = c.key2; 



这个SQL例子中，程序员首先需要选择要查询的主表，因为主表要进行整个表数据的扫描，所以主表应该数据量最小，所以例子中表A的acol列的范围应该比表B和表C相应列的范围小。 



3、在子查询中慎重使用IN或者NOT IN语句，使用where (NOT) exists的效果要好的多。 



4、慎重使用视图的联合查询，尤其是比较复杂的视图之间的联合查询。一般对视图的查询最好都分解为对数据表的直接查询效果要好一些。 



5、可以在参数文件中设置SHARED_POOL_RESERVED_SIZE参数，这个参数在SGA共享池中保留一个连续的内存空间，连续的内存空间有益于存放大的SQL程序包。 



6、ORACLE公司提供的DBMS_SHARED_POOL程序可以帮助程序员将某些经常使用的存储过程“钉”在SQL区中而不被换出内存，程序员对于经常使用并且占用内存很多的存储过程“钉”到内存中有利于提高最终用户的响应时间。 



CPU参数的调整 



CPU是服务器的一项重要资源，服务器良好的工作状态是在工作高峰时CPU的使用率在90％以上。如果空闲时间CPU使用率就在90％以上，说明服务器缺乏CPU资源，如果工作高峰时CPU使用率仍然很低，说明服务器CPU资源还比较富余。 



使用操作相同命令可以看到CPU的使用情况，一般UNIX操作系统的服务器，可以使用sar –u命令查看CPU的使用率，NT操作系统的服务器，可以使用NT的性能管理器来查看CPU的使用率。 



数据库管理员可以通过查看v$sysstat数据字典中“CPU used by this session”统计项得知ORACLE数据库使用的CPU时间，查看“OS User level CPU time”统计项得知操作系统用户态下的CPU时间，查看“OS System call CPU time”统计项得知操作系统系统态下的CPU时间，操作系统总的CPU时间就是用户态和系统态时间之和，如果ORACLE数据库使用的CPU时间占操作系统总的CPU时间90％以上，说明服务器CPU基本上被ORACLE数据库使用着，这是合理，反之，说明服务器CPU被其它程序占用过多，ORACLE数据库无法得到更多的CPU时间。 



数据库管理员还可以通过查看v$sesstat数据字典来获得当前连接ORACLE数据库各个会话占用的CPU时间，从而得知什么会话耗用服务器CPU比较多。 



出现CPU资源不足的情况是很多的：SQL语句的重解析、低效率的SQL语句、锁冲突都会引起CPU资源不足。 



1、数据库管理员可以执行下述语句来查看SQL语句的解析情况： 



SELECT * FROM V$SYSSTAT 



WHERE NAME IN 



('parse time cpu', 'parse time elapsed', 'parse count (hard)'); 



这里parse time cpu是系统服务时间，parse time elapsed是响应时间，用户等待时间 



waite time = parse time elapsed – parse time cpu 



由此可以得到用户SQL语句平均解析等待时间＝waite time / parse count。这个平均等待时间应该接近于0，如果平均解析等待时间过长，数据库管理员可以通过下述语句 



SELECT SQL_TEXT, PARSE_CALLS, EXECUTIONS FROM V$SQLAREA 



ORDER BY PARSE_CALLS; 



来发现是什么SQL语句解析效率比较低。程序员可以优化这些语句，或者增加ORACLE参数SESSION_CACHED_CURSORS的值。 



2、数据库管理员还可以通过下述语句： 



SELECT BUFFER_GETS, EXECUTIONS, SQL_TEXT FROM V$SQLAREA; 



查看低效率的SQL语句，优化这些语句也有助于提高CPU的利用率。 



3、数据库管理员可以通过v$system_event数据字典中的“latch free”统计项查看ORACLE数据库的冲突情况，如果没有冲突的话，latch free查询出来没有结果。如果冲突太大的话，数据库管理员可以降低spin_count参数值，来消除高的CPU使用率。 



内存参数的调整 



内存参数的调整主要是指ORACLE数据库的系统全局区（SGA）的调整。SGA主要由三部分构成：共享池、数据缓冲区、日志缓冲区。 



1、   共享池由两部分构成：共享SQL区和数据字典缓冲区，共享SQL区是存放用户SQL命令的区域，数据字典缓冲区存放数据库运行的动态信息。数据库管理员通过执行下述语句： 



select (sum(pins - reloads)) / sum(pins) "Lib Cache"  from v$librarycache; 



来查看共享SQL区的使用率。这个使用率应该在90％以上，否则需要增加共享池的大小。数据库管理员还可以执行下述语句： 



select (sum(gets - getmisses - usage - fixed)) / sum(gets) "Row Cache" from v$rowcache; 



查看数据字典缓冲区的使用率，这个使用率也应该在90％以上，否则需要增加共享池的大小。 



 2、   数据缓冲区。数据库管理员可以通过下述语句： 



SELECT name, value  FROM v$sysstat  WHERE name IN ('db block gets', 'consistent gets','physical reads'); 



来查看数据库数据缓冲区的使用情况。查询出来的结果可以计算出来数据缓冲区的使用命中率＝1 - ( physical reads / (db block gets + consistent gets) )。 



这个命中率应该在90％以上，否则需要增加数据缓冲区的大小。 



3、   日志缓冲区。数据库管理员可以通过执行下述语句： 



select name,value from v$sysstat where name in ('redo entries','redo log space requests');查看日志缓冲区的使用情况。查询出的结果可以计算出日志缓冲区的申请失败率： 



申请失败率＝requests/entries，申请失败率应该接近于0，否则说明日志缓冲区开设太小，需要增加ORACLE数据库的日志缓冲区。