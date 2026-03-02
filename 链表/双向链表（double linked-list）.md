### 第一部分：知识点大纲（通俗易懂版）

#### 1. 双向链表是为了解决什么“痛点”而诞生的？
在工程视角中，单链表有一个致命的弱点：**它没有“后视镜”**。
假设你在遍历单链表，突然发现节点 `curr` 的数据符合某种条件，你想**删除它**，或者在它**前面插入**一个新节点。你怎么办？
在单链表中，你必须要有 `curr` 的**前驱节点（prev）**才能进行改线操作。但单链表只能往前走！你不得不从头节点再遍历一次，白白浪费 $O(N)$ 的时间。
双向链表的诞生，就是为了实现空间换时间**：通过增加一根反向指针，让你在拿到任何一个节点时，都能拥有“纵览全局、瞻前顾后”的上帝视角。

#### 2. 双向链表的物理结构是什么样的？
单链表是单向牵手，而双向链表是**双向牵手**。
内存里的每一个节点，同时伸出两只手：
- 右手 `next` 牵着后面的兄弟。
- 左手 `prev` 牵着前面的大哥。

如果你想在节点 A 和节点 B 之间插入节点 X，你需要同时打断 A 和 B 之间的双手，并让 X 伸出双手，与 A 和 B 重新建立联系。**这就意味着，一次插入或删除，你需要极其精准地操作 4 根指针的连线（2 对互相指向的指针）！** 这是极易触发 Segfault 的高危操作区。

#### 3. 核心操作的时间复杂度对比
- **查找第 N 个节点**：依然是 $O(N)$，链表的物理内存不连续，无法随机访问（这点跟单链表一样，没救）。
- **已知节点，删除该节点**：
  - 单链表：$O(N)$（需要从头找前驱）。
  - 双向链表：**极速 $O(1)$**（直接 `curr->prev->next = curr->next`，丝滑无比）。
- **已知节点，在它前面插入**：单链表 $O(N)$，双向链表 **$O(1)$**。

**代价是什么？**
每个节点多了一个指针，在 64 位系统中多占 8 Bytes 的内存。同时，你的代码只要写错一行指针顺序，链表就会在内存里变成一团死结。

---

### 第二部分：C++ 新手启动包（代码骨架）

既然你已经领悟了单链表 Dummy Head 的优雅，那在双向链表中，我们将这种优雅推向极致——**双虚拟节点（Dummy Head + Dummy Tail）**！

想象一下，双向链表是一座吊桥。Dummy Head 是左岸的桥墩，Dummy Tail 是右岸的桥墩。无论中间怎么翻江倒海，首尾边界永远坚如磐石。**你彻底不需要写 `if (head == nullptr)` 或者 `if (tail == nullptr)` 这种恶心的特判代码了！**

下面是符合现代 C++ 规范的骨架，我已经帮你建好了桥墩，并写好了正反向遍历的代码：

```cpp
#include <iostream>

// 1. 双向链表节点结构体
struct DNode {
    int val;
    DNode* prev;
    DNode* next;
    
    // 构造函数：默认前后指针为空
    DNode(int v = 0) : val(v), prev(nullptr), next(nullptr) {}
};

// 2. 双向链表类
class DoublyLinkedList {
private:
    int size;
    DNode* dummyHead; // 虚拟头桥墩
    DNode* dummyTail; // 虚拟尾桥墩

public:
    // 构造函数：建立首尾桥墩，并互相牵手
    DoublyLinkedList() {
        size = 0;
        dummyHead = new DNode(-1); // 里面的值无所谓
        dummyTail = new DNode(-1);
        
        // 初始状态：头尾直接双向相连
        dummyHead->next = dummyTail;
        dummyTail->prev = dummyHead;
    }

    // 析构函数：释放所有内存（防内存泄漏，交给你稍后完善）
    ~DoublyLinkedList() {
        // TODO: 依赖你即将实现的机制来释放所有节点，暂时先留空
    }

    // 打印（正向）- 测试用
    void printForward() {
        std::cout << "Forward: DummyHead <-> ";
        DNode* curr = dummyHead->next;
        while (curr != dummyTail) {
            std::cout << curr->val << " <-> ";
            curr = curr->next;
        }
        std::cout << "DummyTail" << std::endl;
    }

    // 打印（反向）- 测试用（检验 prev 指针是否连对的试金石）
    void printBackward() {
        std::cout << "Backward: DummyTail <-> ";
        DNode* curr = dummyTail->prev;
        while (curr != dummyHead) {
            std::cout << curr->val << " <-> ";
            curr = curr->prev;
        }
        std::cout << "DummyHead" << std::endl;
    }

    // ==========================================
    // ⚔️ 你的挑战区域开始 ⚔️
    // ==========================================

    // 任务1：在链表尾部（DummyTail之前）追加节点
    void append(int val) {
        // TODO: 请实现
    }

    // 任务2：在索引 index 处插入新节点（0 <= index <= size）
    void insert(int index, int val) {
        // TODO: 请实现
    }
};

int main() {
    DoublyLinkedList dll;
    
    // 期待你的代码能让这两行跑起来：
    // dll.append(10);
    // dll.append(20);
    // dll.insert(1, 15);
    
    dll.printForward();
    dll.printBackward();
    
    return 0;
}
```

---

### 第三部分：实战过关挑战（由简到难）

接下来的几天，你需要拿下这三个高地：

#### 📌 基础过关：实现 `append` 和 `insert`
- **目标**：补充完整上面的代码，体会同时打断和连接 4 根指针的快感。
- **导师提示**：
  1. `append` 其实就是特殊的 `insert`，你总是在 `dummyTail` 的前一个节点和 `dummyTail` 之间插入！
  2. 插入新节点 `newNode` 时，永远**先绑好新节点的两只手（`prev` 和 `next`），再去打断旧节点的手**。如果你先斩断了旧节点的连接，你就会把下游数据丢进虚空，喜提 Segfault！

#### 📌 经典巩固：实现 `remove(int index)` 删除指定节点
- **目标**：在类中新增 `void remove(int index)` 函数。
- **导师提示**：
  1. 找到该节点后，让它的前驱和后继**直接越过它互相牵手**。
  2. 断开连接后，别忘了 `delete curr;`，否则这就是 Memory Leak（内存泄漏）！

#### 📌 进阶预告：工业界神级应用 —— LRU 缓存淘汰算法
- **预告**：当你熟练掌握了双向链表，我们下一阶段会挑战工业界（比如 Redis 底层）高频使用的 **LRU (Least Recently Used) 缓存淘汰机制**。
- **原理**：用 `哈希表 (Hash Map)` 配合 `双向链表`。哈希表提供 $O(1)$ 的查找，双向链表提供 $O(1)$ 的“移到最前”和“删除队尾”。如果你在这个副本把双向链表练扎实了，LRU 对你来说就是降维打击。

---

### 👨‍💻 导师寄语与 Review 要求

从现在开始，我就是你的 Code Reviewer。

当你准备好 `append` 和 `insert` 的代码并向我提交时，请注意：
**别跟我说“我觉得它是对的”，如果脑子里觉得乱，立刻去给我拿草稿纸画方块和双向箭头！画出每一步指针挪动的先后顺序！**

期待你的提交。去吧，把代码写出来，让我看看你的基本功！如果代码里敢出现空指针解引用（Segfault），准备好迎接我的无情驳回！