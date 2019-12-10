通常大家都会使用redis作为应用的任务队列表，redis的List结构，在一段进行任务的插入，在另一端进行任务的提取。

任务的插入
```
$redis->lPush("key:task:list",$task);
```

任务的提取
```
$tasks = $redis->RPop("key:task:list",0,-1);
```

可是大家想，如何使用mysql来实现一个队列表呢？
映入大家脑海的一个典型的模式是一个表包含多种类型的记录：未处理记录，已处理记录，正在处理记录等。一个或者多个消费者线程在表中查询未处理的记录，然后声称正在处理这个任务，处理完成之后，再讲记录更新为已处理状态。

这个典型的模式，存在两个问题；1：随着队列表越来越大，查找未处理记录的速度会越来越慢。2：频繁的加锁会让多个消费者线程增加竞争。

首先我们来创建一个表
```
create table unsent_emails{
    id int not null primary key auto_increment,
    status enum("unsent","claimed","sent"),
    owner int unsigned not null default 0,
    ts timestamp,
    key (owner,status,ts)
};
```
该表的列owner用来存储当前正在处理这个记录的连接id，由函数 CONNECTION_ID()返回的连接id或者线程id。如果这个记录当前被没有被处理，则该值为0

我们在 owner status ts上面做了索引的处理，所以查找未处理的记录会很快。

通过我们会采用 select for update的方式来标记待处理的记录，方法如下
```
begin;
select id from unsent_emails
    where owner = 0 and status = 'unsent'
    limit 10 for update;
-- result 10,20,33
update unsent_emails
    set status = 'claimed',owner = CONNECTION_ID()
    where id in (10,20,33); 
commit;
```
select的时候，使用了两个索引，应该会很快。问题出在select 和 update两个查询之间的间隙，这里的加锁会让其他相同的查询全部阻塞。

如果我们采用update then select的方式，那么效果就会更加高效，代码如下
```
set autocommit=1;
commit;
update unsent_emails
    set statue = 'claimed',owner = CONNECTION_ID()
    where owner = 0 and status = 'unsent'
    limit 10;
set autocommit=0;
select id from unsent_emails
    where owner = CONNECTION_ID() and status = 'claimed';
```
根本无需使用select去查找哪些记录还没有处理。客户端协议会告诉你更新了几条记录，就可以知道这次需要处理多少条记录。

这样是不是解决了上面的第二个问题，select for update的模式的加锁会增加多个消费队列的竞争问题。

其实所有的select for update 都可以替换为 update then select模式。

问题还没有结束，还有一种情况需要处理，就是比如正在处理任务的进程异常退出了，那么对应的进程正在处理的任务也就变为僵尸任务了。如何避免这种情况的发生呢？

所以我们还是需要一个新的定时器或者线程来定时检测并且update，将那些僵尸任务的记录更新到原始状态，就可以了。
僵尸任务的定义必须符合两点，1：任务被搁置了很久，比如十分钟，而通常一个任务只需要10秒就可以处理完；2：任务的owner(线程id或者连接id)已经不存在，只需要执行show processlist就可以获取当前正在工作的线程id了。代码如下

```
update unsent_emails
    set owner = 0,status = 'unsent'
    where owner not in (10,20,33,44) and status = 'claimed'
    and ts < current_timestamp - interval 10 minute;
```

一个基于mysql构建的队列表就完成了。

当然，最好的办法就是将任务队列从数据库中迁移出来。redis真是一个很好的队列容器，当然也可以使用ssdb(基于leveldb，内存占用更少)。
