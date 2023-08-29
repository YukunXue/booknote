# MSHR-详解

Miss Status Handling Register

--> 当处理器遇到 `cache miss` , 处理器后续的存储访问会被阻塞，直到前面缓存缺失的存储访问得到相应的数据

出现缓存缺失，一个 MSHR 会记录缓存缺失的状态，高速缓存可以继续响应其他存储访问的请求，当缓存缺失满足后，这个 MSHR 会被释放 （`非阻塞或者无锁高速缓存`）

对于无锁缓存，未命中的三种情况：
* 初次缺失（Primary Miss）:

* 二次缺失 (Secondary Miss) :

* 结构停顿缺失（Structural-Stall Miss）:

实现的三种形式：

* 隐式寻址 MSHR （Implicitly Addressed MSHRs）
* 显示寻址 MSHR  (Explicitly Addressed MSHRS)
* 缓存内MSHR     (In-Cache MSHRs)