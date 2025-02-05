# 应该开启多少进程

## 如何设置进程数
进程数是由```count```属性决定的(windows系统不支持进程数设置)，例如下面代码
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// ## 启动4个进程对外提供服务 ##
$http_worker->count = 4;

...

```

## 进程数设置需要考虑以下条件
1、cpu核数

2、内存大小

3、业务偏向IO密集还是CPU密集型

## 进程数设置原则

1、每个进程占用内存之和需要小于总内存（一般来说每个业务进程占用内存大概40M左右）

2、如果是IO密集型，也就是业务中涉及到一些**阻塞式**IO，比如一般的访问Mysql、Redis等存储都是阻塞式访问的，进程数可以开大一些，如配置成CPU核数的3倍。如果业务中涉及的阻塞等待很多，可以再适当加大进程数，例如CPU核数的5倍甚至更高。注意**非阻塞式**IO属于CPU密集型，而不属于IO密集型。

3、如果是CPU密集型，也就是业务中没有**阻塞式**IO开销，例如使用异步IO读取网络资源，进程不会被业务代码阻塞的情况下，可以把进程数设置成和CPU核数一样

****

进程数设置原理有点像生产线上工人们工作的情景，有4个生产线，就相当于CPU是4核的，生产线上的产品相当于任务，工人相当于进程。

工人并非越多越好，因为只有4条生产线，生产能力有限，并且工人多了大家轮流上下生产线开销也很大，同样的进程多了进程间切换开销也很大。

如果流水线上的商品每一步操作起来比较耗时（IO密集型），就需要多一点的工人去操作。

如果流水线上的商品每一步操作都比较快（CPU密集型），只需要少数工人即可

## 进程数设置参考值
如果业务代码偏向IO密集型，也就是业务代码有IO阻塞的地方，则根据IO密集程度设置进程数，例如CPU核数的3倍。

如果业务代码偏向CPU密集型，也就是业务代码中无IO通讯或者无阻塞式IO通讯，则可以将进程数设置成cpu核数。

## 注意
WorkerMan自身的IO都是非阻塞的，例如```Connection->send```等都是非阻塞的，属于CPU密集型操作。如果不清楚自己业务偏向于哪种类型，可设置进程数为CPU核数的2倍左右即可。



