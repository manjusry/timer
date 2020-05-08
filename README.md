# ltimer
高精度海量定时器-链表实现

## 适用场景：超时处理，流表老化

## 设计思路：
### 定时器桶：
1.定时器桶采用预分配内存，创建时需要指定桶的容量，定时器个数不超过桶的最大上限.
2.定时器桶内的定时器用升序链表串起来，链表头部总是最近会超时的定时器.

### 超时机制.
1.每个定时器桶有一个工作线程.
2.工作线程以固定频率触发，更新当前系统时间，并检查桶内是否有已超时的定时器，有则处理.
3.工作线程监听的pipe读端有事件可读，读取出来并处理.

### 定时器.
1.超时时间用绝对时间.
2.循环定时器超时，调用超时处理函数处理，完成后摘链，重新设置超时时间，再插入到升序链表尾部合适的位置.
3.一次性定时器超时，调用超时处理函数处理，完成后摘链，放到回收链表.

### 定时器的增删改.
1.增加：从回收链表或者资源池中取出一个定时器并初始化，创建一个新增事件写入到pipe，由工作线程添加.
2.删除：创建一个删除事件写入到pipe，由工作线程进行删除.
3.修改：创建一个修改事件写入到pipe，由工作线程对定时器进行修改.

