# Data Structure

[思维导图](/mind.html?path=/technology-stack/algorithm/data-structure&initialexpandlevel=2)

[Dictionary of Algorithms and Data Structures](https://xlinux.nist.gov/dads/)

## 数组 (Array)

**定义**

数组是一种线性数据结构，用于存储具有相同类型的元素集合。
这些元素在内存中连续存储，每个元素都可以通过索引来随机访问。

**优点**

- **随机访问**：可以在 O(1) 时间内访问任何元素。
- **简单和易于使用**：数组的实现和使用都很简单。
- **高效的内存利用**：由于连续的内存位置，数组可以高效地利用内存。

**缺点**

- **固定大小**：大多数数组的大小在创建时是固定的，这可能导致内存浪费或限制了数据的存储。
- **插入和删除操作的开销**：插入或删除数组中的元素需要移动其他元素，这可能导致 O(n) 的时间复杂度。

**应用**

- **实现其他数据结构**：如堆、散列表等。
- **缓存**：数组可以用作缓存来存储临时数据。
- **多维数据**：如二维数组、三维数组等，用于存储矩阵、图像数据等。

**问题**

- 如何实现动态数组？

## 字符串 (String)

**定义**

字符串是一种线性数据结构，由一系列字符组成。
字符串的长度是其包含的字符数，可以通过索引访问字符串中的特定字符。

**优点**

- **简单和易于使用**：字符串的实现和使用都很简单。
- **高效的内存利用**：由于连续的内存位置，字符串可以高效地利用内存。

**缺点**

- **固定大小**：大多数字符串的大小在创建时是固定的，这可能导致内存浪费或限制了数据的存储。
- **可能的高内存消耗**：长字符串可能会消耗大量内存，特别是在处理大量文本数据时。
- **字符串操作可能很慢**：某些字符串操作（例如连接、比较和搜索）可能需要线性时间，尤其是对于非常长的字符串。

**应用**

- **文本处理**：字符串是文本的基本数据结构，用于存储和处理文本数据。
- **通信**：字符串用于构建和解析消息和协议，特别是在网络通信和数据交换中。

**问题**

- 如何高效地连接多个字符串？
- 如何在字符串中查找子串？
- 如何比较两个字符串？

## 链表 (Linked List)

**定义**

链表是一种线性数据结构，由一系列节点组成，每个节点包含数据和一个指向下一个节点的指针。
链表的第一个节点称为头节点，最后一个节点称为尾节点。

**优点**

- **动态大小**：与数组不同，链表的大小是动态的，可以根据需要增长或缩小。
- **插入和删除的效率**：在链表的开始或中间插入或删除节点是 O(1) 的操作，只要我们有指向该节点的指针。
- **不需要连续的内存空间**：链表的节点可以分散在内存中，不需要像数组那样占用连续的内存空间。

**缺点**

- **随机访问**：链表不支持随机访问，访问链表中的特定索引需要从头开始遍历，时间复杂度为 O(n)。
- **更多的内存开销**：由于每个节点需要额外的指针来存储下一个节点的地址，所以链表在内存使用上比数组要多。
- **不支持缓存局部性**：由于链表的节点可能分散在内存中，它们不太可能在缓存中连续存储，这可能导致较低的缓存性能。

**应用**

- **实现其他数据结构**：链表常用于实现其他数据结构，如栈、队列、哈希表的桶等。
- **动态内存分配**：链表可以用于管理动态分配的内存块，如内存管理器中的空闲块列表。
- **导航**：在某些应用中，如 web 浏览器的前进和后退按钮，可以使用双链表来导航。

**问题**

- 如何实现一个双向、循环链表？
- 跳跃表 (Skip List) 是什么？如何实现？相对于红黑树的优缺点？

## 栈 (Stack)

**定义**

栈是一种线性数据结构，它按照后进先出（LIFO，Last In First Out）的原则进行操作。
在栈中，所有的新元素都在栈顶添加，而删除操作也仅在栈顶进行。
每一个栈都有一个指向栈顶的指针来标识栈的当前位置。

**优点**

- **简单性**：栈的操作非常简单，使其易于实现和使用。
- **快速访问**：Push 和 Pop 操作通常是 O(1) 的时间复杂度。
- **局部性**：由于栈操作主要涉及栈顶，这提供了良好的缓存局部性，从而提高速度。

**缺点**

- **有限的容量**：在某些实现中，栈的大小可能是固定的，这可能导致栈溢出。
- **不支持随机访问**：要访问栈中的任何元素，你需要从栈顶开始，并移除所有其他元素，直到到达所需的元素。

**应用**

- **函数调用和递归**：每当程序执行一个函数调用时，系统都会将返回地址和局部变量推入栈中，并在函数返回时弹出。
- **表达式求值和语法分析**：栈用于解析和求值算术或逻辑表达式。
- **深度优先搜索**：在图和树的深度优先搜索算法中，栈用于跟踪待访问的顶点。

**问题**

- 如何实现一个具有最小值操作的栈？
- 如何在一个数组中实现两个栈？

## 队列 (Queue)

**定义**

队列是一种线性数据结构，它按照先进先出（FIFO，First In First Out）的原则进行操作。
在队列中，新元素添加到队列的尾部，而元素的移除发生在队列的头部。
每一个队列都有一个指向队头和队尾的指针，以便能够快速地添加和移除元素。

**变种**

1. **双端队列（Deque）**：允许在前端和末尾两端进行插入和删除操作的队列。
2. **优先队列**：每个元素都有一个优先级，元素按优先级出队，而不是它们进入队列的顺序。
3. **循环队列**：队列的末尾连接到其前端，形成一个循环。

**优点**

1. **操作简单高效**：队列的主要操作（入队、出队、查看队头）的时间复杂度为 O(1)。
2. **公平性**：由于队列遵循 FIFO 原则，它确保每个元素都按照其到达的顺序得到处理。

**缺点**

1. **随机访问困难**：要访问队列中的任意元素，需要从队头开始并遍历整个队列，时间复杂度为 O(n)。
2. **可能存在的空间浪费**：在某些实现中，例如静态数组实现的队列，可能会在队列未满时浪费一些空间。

**应用**

1. **事件处理**：在许多系统和应用程序中，队列用于管理事件和请求，确保它们按照正确的顺序得到处理。
2. **调度算法**：在操作系统中，队列用于实现各种调度算法，如先来先服务（FCFS）、短进程优先（SJF）等。
3. **广度优先搜索**：在图和树的广度优先搜索算法中，队列用于保持待访问的顶点。

**问题**

- 如何实现一个双端、循环队列？
- 如何实现一个阻塞、优先队列？

## 树 (Tree)

**定义**

树是一种非线性数据结构，由节点组成，每个节点有零个或多个子节点。
树的顶部是根节点，底部是叶节点。

**优点**

- **层次结构**：树可以表示数据的层次结构，如文件系统中的文件和目录。
- **搜索效率**：比如，在二叉搜索树中，搜索操作时间复杂度可达到 O(log n)。

**缺点**

- **难以实现**：与线性数据结构相比，树的实现和操作通常更复杂。
- **存储开销**：每个节点需要额外的存储空间来保存指针。

**应用**

- **文件系统**：目录和文件的结构可以表示为树。
- **数据库**：用于索引的数据结构，如B树和B+树。
- **路由算法**：如在计算机网络中使用的路由表。

**问题**

- 有哪些常见的树结构？它们之间有什么区别？
    - **二叉树**：每个节点最多有两个子节点。
    - **二叉搜索树**：左子节点的值小于父节点，右子节点的值大于父节点。
    - **平衡二叉树**：任意节点的两个子树的高度差不超过 1 的二叉搜索树。
    - **红黑树**：自平衡的二叉搜索树，通过颜色和旋转保持平衡。
    - **B树**：多路搜索树，每个节点可以有多个子节点。
    - **B+树**：B树的变体。非叶子节点仅包含键，数据仅存储在叶子节点；叶子节点通过指针连接，形成一个链表，方便了范围查询。
    - **完全二叉树**：除了最后一层外，每一层上的节点数均达到最大值；在最后一层上只缺少右边的若干节点。
    - **堆**：是一种特殊的完全二叉树，父节点的值总是大于（或小于）其子节点，如最大堆和最小堆。
- 如何平衡二叉树？
- 如何从数组创建堆？

## 散列表 (Hash Table)

**定义**

散列表是通过哈希函数将键映射到数组的索引，从而实现快速数据查找的数据结构。

**优点**

- **高效的查找、插入和删除**：平均情况下，这些操作的时间复杂度为 O(1)。

**缺点**

- **哈希冲突**：需要特定的策略来处理哈希冲突，这可能会影响性能。

**应用**

- **数据查找**：如数据库索引、缓存等。
- **数据去重**：利用散列表快速判断元素是否已存在。

**问题**

- 如何解决哈希冲突？
    - **链地址法**：每个散列表的槽位都存储一个链表，所有映射到同一索引的键值对都存储在这个链表中。
    - **开放地址法**：当发生冲突时，查找散列表中的下一个空槽位。这包括线性探测、二次探测和双重哈希等策略。
- 如何扩容、缩容？
    - **再哈希**：当散列表变得太满或太空时，调整其大小并重新哈希所有键值对。

## 集合 (Set)

**定义**

集合是一个无序的元素集合，不包含重复的元素。

**优点**

- **元素唯一**：自动处理重复。
- **基本操作高效**：如添加、删除、检查元素是否存在。
- **数学操作**：集合提供了一系列数学操作，如并集、交集和差集，这在某些应用中非常有用。

**缺点**

- **无序**：集合中的元素没有特定的顺序。

**应用**

- **数据去重**：快速去除重复元素。
- **集合运算**：如并集、交集、差集等。

**问题**

- 如何实现一个集合？
- 如何实现一个有序集合？
- 什么是 位集合 (BitSet)？可以用来做什么？

## 图 (Graph)

**定义**

图是由节点（顶点）和边组成的数据结构，用于表示多对多的关系。

**优点**

- **表达能力强**：能表示复杂的关系。

**缺点**

- **复杂性**：图的算法和操作通常较为复杂。

**应用**

- **网络、社交网络分析**：表示和分析复杂网络结构。
- **路由算法**：如最短路径问题。

**问题**

- 如何表示一个图？
- 如何遍历一个图？
- 如何实现一个图的深度优先搜索算法？
- 如何实现一个图的广度优先搜索算法？