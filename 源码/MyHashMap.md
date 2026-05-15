```cpp
#include <iostream>

#include <vector>

struct Node {
    int key;
    int value;
    Node * next;
    Node(int key_out, int val): key(key_out), value(val), next(nullptr) {}
};
class MyHashMap {
    private: std::vector < Node * > buckets;
    // 哈希函数：简单的 key % buckets.size()
    public: MyHashMap(int capacity = 8) {
        // buckets.reserve(capacity); 有问题，预订容量，里面没东西，没办法访问buckets[index]
        buckets.resize(capacity, nullptr);
    }
    void put(int key, int value) {
        // int index = key % buckets.size();这里有问题，如果是负数取模，在cpp里面是负数，应该采用正数写法
        int index = (key % buckets.size() + buckets.size()) % buckets.size();
        Node * p = buckets[index];
        while (p != nullptr) {
            if (p - > key == key) {
                p - > value = value;
                return;
            }
            p = p - > next;
        }
        Node * add = new Node(key, value);
        add - > next = buckets[index];
        buckets[index] = add;
    }
    int get(int key) {
        int index = (key % buckets.size() + buckets.size()) % buckets.size();
        Node * p = buckets[index];
        while (p != nullptr) {
            if (p - > key == key) {
                return p - > value;
            }
            p = p - > next;
        }
        std::cout << "Non-existent key" << '\n';
        return -1;
    }
};

void remove(int key) {
    // 1. 计算哈希索引
    int index = (key % buckets.size() + buckets.size()) % buckets.size();

    Node * curr = buckets[index];
    Node * prev = nullptr; // 用于记录当前节点的前一个节点

    // 2. 遍历链表寻找对应的 key
    while (curr != nullptr) {
        if (curr - > key == key) {
            // 找到了要删除的节点
            if (prev == nullptr) {
                // 情况 A：要删除的节点正好是链表的第一个节点（头节点）
                // 直接让桶里的指针指向下一个节点
                buckets[index] = curr - > next;
            } else {
                // 情况 B：要删除的节点在链表的中间或尾部
                // 跳过当前节点，将前后连接起来
                prev - > next = curr - > next;
            }

            // 3. 释放内存
            delete curr;
            return; // 删除成功，直接返回
        }
        // 指针往下走
        prev = curr;
        curr = curr - > next;
    }

    // 如果循环结束还没找到，说明 key 不存在，不需要做任何事
}
```