# [146. LRU 缓存机制](https://leetcode-cn.com/problems/lru-cache)

[English Version](/solution/0100-0199/0146.LRU%20Cache/README_EN.md)

## 题目描述

<!-- 这里写题目描述 -->

<div class="title__3Vvk">运用你所掌握的数据结构，设计和实现一个  <a href="https://baike.baidu.com/item/LRU" target="_blank">LRU (最近最少使用) 缓存机制</a> 。</div>

<div class="original__bRMd">
<div>
<p>实现 <code>LRUCache</code> 类：</p>

<ul>
	<li><code>LRUCache(int capacity)</code> 以正整数作为容量 <code>capacity</code> 初始化 LRU 缓存</li>
	<li><code>int get(int key)</code> 如果关键字 <code>key</code> 存在于缓存中，则返回关键字的值，否则返回 <code>-1</code> 。</li>
	<li><code>void put(int key, int value)</code> 如果关键字已经存在，则变更其数据值；如果关键字不存在，则插入该组「关键字-值」。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。</li>
</ul>

<p> </p>
</div>
</div>

<p><strong>进阶</strong>：你是否可以在 <code>O(1)</code> 时间复杂度内完成这两种操作？</p>

<p> </p>

<p><strong>示例：</strong></p>

<pre>
<strong>输入</strong>
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]
<strong>输出</strong>
[null, null, null, 1, null, -1, null, -1, 3, 4]

<strong>解释</strong>
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // 缓存是 {1=1}
lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}
lRUCache.get(1);    // 返回 1
lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}
lRUCache.get(2);    // 返回 -1 (未找到)
lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}
lRUCache.get(1);    // 返回 -1 (未找到)
lRUCache.get(3);    // 返回 3
lRUCache.get(4);    // 返回 4
</pre>

<p> </p>

<p><strong>提示：</strong></p>

<ul>
	<li><code>1 <= capacity <= 3000</code></li>
	<li><code>0 <= key <= 3000</code></li>
	<li><code>0 <= value <= 10<sup>4</sup></code></li>
	<li>最多调用 <code>3 * 10<sup>4</sup></code> 次 <code>get</code> 和 <code>put</code></li>
</ul>

## 解法

<!-- 这里可写通用的实现逻辑 -->

“哈希表 + 双向链表”实现。其中：

- 双向链表按照被使用的顺序存储 kv 键值对，靠近头部的 kv 键值对是最近使用的，而靠近尾部的键值对是最久未使用的。
- 哈希表通过缓存的 key 映射到双向链表中的位置。我们可以在 `O(1)` 时间内定位到缓存的 key 所对应的 value 在链表中的位置。

对于 `get` 操作，判断 key 是否存在哈希表中：

- 若不存在，返回 -1
- 若存在，则 key 对应的节点 node 是最近使用的节点。将该节点移动到双向链表的头部，最后返回该节点的值即可。

对于 `put` 操作，同样先判断 key 是否存在哈希表中：

- 若不存在，则创建一个新的 node 节点，放入哈希表中。然后在双向链表的头部添加该节点。接着判断双向链表节点数是否超过 capacity。若超过，则删除双向链表的尾部节点，以及在哈希表中对应的项。
- 若存在，则更新 node 节点的值，然后该节点移动到双向链表的头部。

双向链表节点（哈希表的 value）的结构如下：

```java
class Node {
	int key;
	int value;
	Node prev;
	Node next;
	Node() {

	}
	Node(int key, int value) {
		this.key = key;
		this.value = value;
	}
}
```

你可能会问，哈希表的 value 为何还要存放 key？

这是因为，双向链表有一个删除尾节点的操作。我们定位到双向链表的尾节点，在链表中删除之后，还要找到该尾节点在哈希表中的位置，因此需要根据 value 中存放的 key，定位到哈希表的数据项，然后将其删除。

<!-- tabs:start -->

### **Python3**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```python
class Node:
    def __init__(self, key=0, value=0):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None

class LRUCache:

    def __init__(self, capacity: int):
        self.size = 0
        self.capacity = capacity
        self.cache = {}
        self.head = Node()
        self.tail = Node()
        self.head.next = self.tail
        self.tail.prev = self.head

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        node = self.cache[key]
        self._move_to_head(node)
        return node.value

    def put(self, key: int, value: int) -> None:
        if key not in self.cache:
            new_node = Node(key, value)
            self.cache[key] = new_node
            self._add_to_head(new_node)
            self.size += 1
            if self.size > self.capacity:
                node = self._remove_tail()
                self.cache.pop(node.key)
                self.size -= 1
        else:
            node = self.cache[key]
            node.value = value
            self._move_to_head(node)

    def _move_to_head(self, node):
        self._remove_node(node)
        self._add_to_head(node)

    def _remove_node(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev

    def _add_to_head(self, node):
        node.next = self.head.next
        node.next.prev = node
        node.prev = self.head
        self.head.next = node

    def _remove_tail(self):
        node = self.tail.prev
        self._remove_node(node)
        return node

# Your LRUCache object will be instantiated and called as such:
# obj = LRUCache(capacity)
# param_1 = obj.get(key)
# obj.put(key,value)
```

### **Java**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```java
class LRUCache {
    class Node {
        int key;
        int value;
        Node prev;
        Node next;
        Node() {

        }
        Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }

    private int size;
    private int capacity;
    private Map<Integer, Node> cache;
    private Node head;
    private Node tail;

    public LRUCache(int capacity) {
        this.size = 0;
        this.capacity = capacity;
        cache = new HashMap<>();
        head = new Node();
        tail = new Node();
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        Node node = cache.get(key);
        if (node == null) {
            return -1;
        }
        moveToHead(node);
        return node.value;
    }

    public void put(int key, int value) {
        Node node = cache.get(key);
        if (node == null) {
            Node newNode = new Node(key, value);
            cache.put(key, newNode);
            addToHead(newNode);
            ++size;
            if (size > capacity) {
                Node tail = removeTail();
                cache.remove(tail.key);
                --size;
            }
        } else {
            node.value = value;
            moveToHead(node);
        }
    }

    private void moveToHead(Node node) {
        removeNode(node);
        addToHead(node);
    }

    private void removeNode(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void addToHead(Node node) {
        node.next = head.next;
        head.next = node;
        node.next.prev = node;
        node.prev = head;
    }

    private Node removeTail() {
        Node node = tail.prev;
        removeNode(node);
        return node;
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```

### **...**

```

```

<!-- tabs:end -->
