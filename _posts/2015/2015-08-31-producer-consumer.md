---
layout: post
title: iOS下的生产者消费者经典模型
categories:
- Programming
tags:
- iOS
- 生产者消费者

---

一. 明确定义

要理解生产消费者问题，首先应弄清PV操作的含义：PV操作是由P操作原语和V操作原语组成（原语是不可中断的过程），对信号量进行操作，具体定义如下：

    P（S）：①将信号量S的值减1，即S=S-1；
           ②如果S>0，则该进程继续执行；否则该进程置为等待状态，排入等待队列。
    V（S）：①将信号量S的值加1，即S=S+1；
           ②如果S>0，则该进程继续执行；否则释放队列中第一个等待信号量的进程。
           
这只是书本的定义，对于这部分内容，先不要急于解释上面的程序流程，而是应该首先知道P操作与V操作到底有什么作用。
P操作相当于申请资源，而V操作相当于释放资源。所以要记住以下几个关键字：

	P操作----->申请资源
	V操作----->释放资源

二. 形象启发

为此举两个生活中的例子：

例一：在公共电话厅打电话

公共电话厅里有多个电话，如某人要打电话，首先要进行申请，看是否有电话空闲，若有，则可以使用电话，如果电话亭里所有电话都有人正在使用，那后来的人只有排队等候。当某人用完电话后，则有空电话腾出，正在排队的第一个人就可以使用电话。这就相当于PV操作：

某人要打电话，首先要进行申请，相当于执行一次P操作，申请一个可用资源（电话）；

某人用完电话，则有空电话腾出，相当于执行一次V操作，释放一个可用资源（电话）。

三. 分层解剖

在理解了PV操作的的含义后，就必须理解利用PV操作可以实现进程的两种情况：互斥和同步。根据互斥和同步不同的特点，***就有利用PV操作实现互斥与同步相对固定的结构模式***。这里就不详细讲解了。但生产者-消费者问题是一个有代表性的进程同步问题，要透彻理解并不容易。但是如果我们将问题细分成三种情况进行讲解，理解难度将大大降低。

1）一个生产者，一个消费者，公用一个缓冲区。

可以作以下比喻：将一个生产者比喻为一个生产厂家，如伊利牛奶厂家，而一个消费者，比喻是学生小明，而一个缓冲区则比喻成一间好又多。第一种情况，可以理解成伊利牛奶生产厂家生产一盒牛奶，把它放在好又多一分店进行销售，而小明则可以从那里买到这盒牛奶。只有当厂家把牛奶放在商店里面后，小明才可以从商店里买到牛奶。所以很明显这是最简单的同步问题。

解题如下：

	定义两个同步信号量：
	empty——表示缓冲区为空的数量，初值为1。
	full——表示缓冲区中为满的数量，初值为0。
	生产者进程
	while(TRUE){
	生产一个产品;
	     P(empty);
	     产品送往Buffer;
	     V(full);
	}
	消费者进程
	while(TRUE){
	P(full);
	   从Buffer取出一个产品;
	   V(empty);
	   消费该产品;
	}
	   
2）一个生产者，一个消费者，公用n个环形缓冲区。

第二种情况可以理解为伊利牛奶生产厂家可以生产好多牛奶，并将它们放在多个好又多分店进行销售，而小明可以从任一间好又多分店中购买到牛奶。同样，只有当厂家把牛奶放在某一分店里，小明才可以从这间分店中买到牛奶。不同于第一种情况的是，第二种情况有N个分店（即N个缓冲区形成一个环形缓冲区），所以要利用指针，要求厂家必须按一定的顺序将商品依次放到每一个分店中。***缓冲区的指向则通过模运算得到。***
 
解题如下：

	定义两个同步信号量： 
	empty——表示缓冲区为空的数量，初值为n。
	full——表示缓冲区中为满的数量，初值为0。
	    设缓冲区的编号为1～n-1，定义两个指针in和out，分别是生产者进程和消费者进程使用的指针，指向下一个可用的缓冲区。
	 
	生产者进程
	while(TRUE){
	     生产一个产品;
	     P(empty);
	     产品送往buffer（in）；
	     in=(in+1)mod n；
	     V(full);
	}
	消费者进程
	while(TRUE){
	 P(full);
	   从buffer（out）中取出产品；
	   out=(out+1)mod n；
	   V(empty);
	   消费该产品;
	}
	   
3）一组生产者，一组消费者，公用n个环形缓冲区

第三种情况，可以理解成有多间牛奶生产厂家，如蒙牛，达能，光明等，消费者也不只小明一人，有许许多多消费者。不同的牛奶生产厂家生产的商品可以放在不同的好又多分店中销售，而不同的消费者可以去不同的分店中购买。当某一分店已放满某个厂家的商品时，下一个厂家只能把商品放在下一间分店。***所以在这种情况中，生产者与消费者存在同步关系，而且各个生产者之间、各个消费者之间存在互斥关系,他们必须互斥地访问缓冲区。***

解题如下：

	定义四个信号量：
	empty——表示缓冲区为空的数量，初值为n。
	full——表示缓冲区为满的数量，初值为0。
	mutex1——生产者之间的互斥信号量，初值为1。
	mutex2——消费者之间的互斥信号量，初值为1。
	设缓冲区的编号为1～n-1，定义两个指针in和out，分别是生产者进程和消费者进程使用的指针，指向下一个可用的缓冲区。
	 
	生产者进程
	while(TRUE){
	     生产一个产品;
	     P(empty);
	     P(mutex1);
	     产品送往buffer（in）;
	     in=(in+1)mod n;
	     V(mutex1);
	     V(full);
	}
	消费者进程
	while(TRUE){
	 P(full);
	   P(mutex2);
	   从buffer（out）中取出产品;
	   out=(out+1)mod n;
	   V（mutex2）;
	   V(empty);
	}

四. iOS中实现

{% highlight objc %}
// 定义常量与变量
#define BUFFER_SIZE  (10)
dispatch_semaphore_t _empty;
dispatch_semaphore_t _full;
NSMutableArray* _buffer;
int _inIndex;
int _outIndex;
NSLock* _produceLock;
NSLock* _consumerLock;

- (void)setupAndRunProduceConsume {
    _empty = dispatch_semaphore_create(BUFFER_SIZE);
    _full = dispatch_semaphore_create(0);
    _buffer = [[NSMutableArray alloc] init];
    for (int i = 0; i < BUFFER_SIZE; ++i) {
        [_buffer addObject:[NSNull null]];
    }
    _inIndex = 0;
    _outIndex = 0;
    _produceLock = [[NSLock alloc] init];
    _consumerLock = [[NSLock alloc] init];
    for (int i = 0; i < 2; ++i) {
        NSThread* produceThread = [[NSThread alloc] initWithTarget:self selector:@selector(produceMain) object:nil];
        [produceThread start];
    }
    for (int i = 0; i < 5; ++i) {
        NSThread* consumerThread = [[NSThread alloc] initWithTarget:self selector:@selector(consumeMain) object:nil];
        [consumerThread start];
    }
}

- (void)produceMain {
    while (true) {
        dispatch_semaphore_wait(_empty, DISPATCH_TIME_FOREVER);
        [_produceLock lock];
        _buffer[_inIndex] = [NSString stringWithFormat:@"%d", _inIndex];
        NSLog(@"produceMain: %d", _inIndex);
        _inIndex = (_inIndex + 1) % BUFFER_SIZE;
        [_produceLock unlock];
        dispatch_semaphore_signal(_full);
    }
}

- (void)consumeMain {
    while (true) {
        dispatch_semaphore_wait(_full, DISPATCH_TIME_FOREVER);
        [_consumerLock lock];
        _buffer[_outIndex] = [NSNull null];
        _outIndex = (_outIndex + 1) % BUFFER_SIZE;
        NSLog(@"consumerMain: %@", _buffer);
        [_consumerLock unlock];
        dispatch_semaphore_signal(_empty);
        sleep(1); // do sth
    }
}
{% endhighlight %}
