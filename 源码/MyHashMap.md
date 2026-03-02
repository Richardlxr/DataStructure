```cpp
#include <iostream>
#include <vector>
struct Node
{
int key;
int value;
Node *next;
Node(int key_out, int val) : key(key_out), value(val), next(nullptr) {}
};
class MyHashMap
{
private:
std::vector<Node *> buckets;
// 哈希函数：简单的 key % buckets.size()
public:
MyHashMap(int capacity = 8)
{
// buckets.reserve(capacity); 有问题，预订容量，里面没东西，没办法访问buckets[index]
buckets.resize(capacity, nullptr);
}
void put(int key, int value)
{
// int index = key % buckets.size();这里有问题，如果是负数取模，在cpp里面是负数，应该采用正数写法
int index = (key % buckets.size() + buckets.size()) % buckets.size();
Node *p = buckets[index];
while (p != nullptr)
{
if (p->key == key)
{
p->value = value;
return;
}
p = p->next;
}
Node *add = new Node(key, value);
add->next = buckets[index];
buckets[index] = add;
}  
int get(int key)
{
int index = (key % buckets.size() + buckets.size()) % buckets.size();
Node *p = buckets[index];
while (p != nullptr)
{
if (p->key == key)
{
return p->value;
}
p = p->next;
}
std::cout << "Non-existent key" << '\n';
return -1;
}
};
```