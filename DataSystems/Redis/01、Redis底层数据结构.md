### 简单动态字符串（SDS）

Redis 只会使用 C字符串 作为字面量，在大多数情况下，Redis使用SDS（Simple Dynamic String，简单动态字符串）作为字符串表示

`sds.h/sdshdr`

```
struct sdshdr {
	int len 		// buf数组中已使用字节的数量，等于SDS保存字符串的长度
	int free		// buf数组中未使用字节的数量
	char buf[]	// 字节数组，用于保存字符串
}
```

优点：

- 常数复杂度获取字符串长度
- 杜绝缓冲区溢出
- 减少修改字符串时带来的内存重分配次数
- 二进制安全
- 兼容部分C字符串函数

### 链表

链表提供了高效的节点重排能力,以及顺序性的节点访问方式,并且可以通过增删节点来灵活地调整链表的长度。

`adlist.h/listNode`

```
struct listNode {
	struct listNode *prev		// 前置节点
  struct listNode *prev		// 后置节点
  void *value 						// 节点的值
}
```

`adlist.h/list`

```

```

