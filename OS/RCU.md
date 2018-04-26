# RCU无锁编程


### RCU原理

实际的RCU内核实现原理比这要复杂的多，为了讲明白RCU的实现原理，我这里会尽量简化   
### 目的
RCU主要是用来保护一块内存的，而且是一块指针指向的内存.

RCU强大之处在于可以并行的读和更新数据，而且读是零开销的，非常适合多读一写的使用场景，如果  
有多个写者，写者之间必须有其他的同步机制保证(比如加锁)

所以深受Linux内核程序员的喜爱，现在内核很多老的模块代码都在逐步转向RCU，去掉性能开销大的读写锁。  

### RCU实现

**RCU是怎么实现读和写可以并发执行的呢？**

我假设gp这里是个全局指针,gp指向RCU需要保护那块内存

RCU读操作

```
 if (gp != NULL) { 
 	rp = gp;
 	do_read_something(rp)
 }
```
RCU更新操作

```
 new_gp = malloc(sizeof(*p);
 memcpy(new_gp, gp, sizeof(*p));
 modify(new_gp);
 old_gp = gp;
 gp = new_gp;

```

假设上面两个操作是并发执行的，那么对读者来讲，有两种可能，一种是在RCU更新操作 `gp = new_gp `  
语句之前去读，那么读到旧版本的gp内存，还有一种情况是在RCU更新操作 `gp = new_gp `语句之后去读，        
那么读到的就是新版本的gp内存. 不论是新旧版本的内存，对读者都是没有影响的。

看!!! 这里不就实现了读和更新操作的并发了吗？  
通过对旧版本内存做副本拷贝的方式来修改数据，再做指针替换更新

但是聪明的你会发现，旧版本的内存什么释放呢？

我们先看一个RCU释放版本

```
 new_gp = malloc(sizeof(*p);
 memcpy(new_gp, gp, sizeof(*p));
 modify(new_gp);
 old_gp = gp;
 gp = new_gp;
 free(old_gp); //增加了这一行

```
该版本在更新完内存后立马释放旧版本内存  
如果此时读者读到是旧版本的内存，但是此时RCU写者将旧版本内存释放了，那不就野指针了吗，  
所以这个方案肯定不行的。

**那什么时候去释放旧版内存才是安全的呢?**

带着这个问题去看RCU的核心机制，你就明白了

我们这里抛开内核的RCU实现机制，使用自己的方式在应用层来实现一个RCU锁，更容易明白

假设我们有两个CPU核，每个CPU都跑一个线程(多个CPU以此类推)，我们这里暂且叫做线程A，和线程B。   
线程A绑定在其中一个CPU上，线程B绑定在另一个CPU上。这两个线程会并行，而且是可以直接通信的   


```
//RCU计数器[](全局变量，每个CPU对应一个，对线程A或者线程B都可见)
RCU_counter[2]

//RCU_job链表头[](全局变量，每个CPU对应一个，对线程A或者线程B都可见)
Rcu_job_list[CPU个数]

RCU_job {
	// 生成RCU_job时,对每个计数器的快照
	snapshot[2]
	callback(void *)
}



call_rcu() {
	//对当前的RCU_counter计数器数组做快照, 分配一个job
	job = RCU_job(RCU_counter， free_rcu_mem_callback)
	//把job注册到Rcu_job_list列表
	list_add(RcuList[current_cpu], job)
}

//如果发现所有的计数器都改变了，线程A和线程B至少都循环一遍了，这样可以保证没有
//RCU读者引用那块将要释放的内存了，此时释放内存是安全的。
all_threads_has_forwarded(job){
	return job.snapshot[0] != RCU_counter[0] && job.snapshot[1] != RCU_counter[1]
}

reclaim_rcu_mem() {
	for job in RcuList {
		if (all_thread_has_forwarded(job)) {
			free(job)
		} else {
			//第一个都没有循环一周，后面就更没有了,作为一个优化trick
			break;
		}
	}
}

线程 A
for {
	event = 事件接收器
	
	switch event {
		EVENT X: RCU 读操作
	}
	counter[current_cpu]++
	reclaim_rcu_mem()
}

线程B
for {
	event = 事件接收器
	
	switch event {
		EVENT Y : RCU 更新操作
		call_rcu()
	}
	
	counter[current_cpu]++
	reclaim_rcu_mem()

}

```

上面就是RCU应用层伪码实现，我们现在对照伪码，来解释RCU什么时候释放内存是安全的.

### 应用描述

线程A和线程B的结构是一样的，都是一个事务处理器(消息处理器也一样)，这样在事务处理器中，  
对RCU包含内存进行读，是零开销的。

1. 当我们在线程B中事务处理器中完成一个RCU更新操作后，旧版的内存会放在一个job里面
2. 此时线程A中的事务处理器中的RCU读操作读取到了这个旧版的内存
3. 线程B在填充job的时候，我们还需要对Rcu所有的计数器(作用在后面讲解)进行快照，也   
一起保存在job中，再将job挂入到每个线程对应的RCU_job_list中
4. 每个事务处理器完成所有事务处理之后，都需要尝试对挂在本线程上的的RCU_job_list上的  
回收job进行扫描回收工作
5. 如果发现有job中的快照计数器改变了，说明当前情况下，有对当前job内存进行引用的读者也   
循环了一遍了，这样能保证在当前情况释放内存，已经没有任何线程引用到这块内存了，所以此时   
此时释放内存是非常安全的


### 内核实现概述
内核实现和我们这个应用实现，核心思想是一模一样的，在内存中两个线程中的事务处理器可以当做是在  
内核中所有上下文，但是区别就是什么时候去回收rcu，内核是把reclaim_rcu_mem放在定时器中断  
里面的，当所有的进程都重新调度的时候，可以安全的认为所有的进程对旧版RCU内存都解引用了，这个  
时候调用RCU回收函数非常安全

**注：上面实现都是伪码，RCU真正实现读-更新操作，还需要考虑CPU乱序问题，我们需要加编译器宏**   
**来解决CPU指令乱序**


