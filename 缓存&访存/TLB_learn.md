# TLB (translation lookaside buffer)


MMU -> PA ---> VA

64位系统 --> 3 ~ 5级

## 四级页表：

PGD、PUD、 PMD、 PTE

## 本质

TLB --> 虚拟高速缓存 （ DCache缓存地址 （PA or VA）和数据， TLB缓存虚拟地址和其映射的物理地址）
