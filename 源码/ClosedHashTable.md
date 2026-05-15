```cpp
#include <iostream>

#include <vector>

using namespace std;

// 闭散列表的节点
struct HashNode {
    int key;
    int value;
    int state; // 状态机：0-Empty, 1-Active, 2-Deleted

    // 默认构造，状态为 0 (Empty)
    HashNode(): key(0), value(0), state(0) {}
};

class ClosedHashTable {
    private: vector < HashNode > array; // 纯数组！没有链表指针！
    int size;

    int hashFunc(int key) {
        return key % size;
    }

    public: ClosedHashTable(int capacity = 10) {
        size = capacity;
        array.resize(size);
    }

    // ============================================
    // 🔥 任务：运用线性探测和墓碑机制完成以下操作
    // ============================================

    // Task 1: 插入/更新 (参考 PPT P76)
    // 逻辑：遇到 state == 1 且 key 不相等，就一直往后找。
    // 如果遇到 state == 0 (空) 或者 state == 2 (墓碑)，说明这里可以停车！
    void insert(int key, int value) {
        int initPos = hashFunc(key);
        int pos = initPos;

        do {
            // TODO: 判断当前 pos 能不能停车？(提示：只要不是 Active 状态，就可以覆盖抢占！)
            // 如果能停车：把 key, value 写进去，把 state 改为 1。然后 return 结束。
            // 如果已经被占了，但 key 刚好和我们要插入的一样，说明是更新操作，把 value 改掉，return
            // 结束。
            if (array[pos].state == 0 || array[pos].state == 2) {
                array[pos].key = key;
                array[pos].value = value;
                array[pos].state = 1;
                return;
            } else {
                if (array[pos].key == key) {
                    array[pos].value = value;
                    return;
                }
            }
            // 往后找下一个车位 (取模是为了绕回数组开头)
            pos = (pos + 1) % size;
        } while (pos != initPos); // 如果绕了一圈回到原点，说明数组全满了

        cout << "Table is full!" << endl;
    }

    // Task 2: 查找 (参考 PPT P78)
    // 逻辑：遇到 state == 0，说明线索断了，彻底没戏，返回 -1。
    // 遇到 state == 2 (墓碑)，别停下，继续往后找！
    int find(int key) {
        int initPos = hashFunc(key);
        int pos = initPos;

        do {
            // TODO: 实现查找逻辑
            // 1. 如果当前 state 是 0，返回 -1
            // 2. 如果当前 state 是 1，并且 key 匹配，返回当前的 value
            // 3. 否则，继续找下一个
            if (array[pos].state == 0)
                return -1;
            else if (array[pos].state == 1 && array[pos].key == key) {
                return array[pos].value;
            }
            pos = (pos + 1) % size;
        } while (pos != initPos);

        return -1;
    }

    // Task 3: 删除 (参考 PPT P77)
    // 逻辑：和查找一模一样，只不过找到之后，不是返回值，而是把 state 改成 2！
    void remove(int key) {
        int initPos = hashFunc(key);
        int pos = initPos;

        do {
            // TODO: 实现删除逻辑
            // 提示：遇到 0 直接 return。找到匹配的，立上墓碑 (state=2) 然后 return。
            if (array[pos].state == 0)
                return;
            else if (array[pos].state == 1 && array[pos].key == key) {
                array[pos].state = 2;
                return;
            }
            pos = (pos + 1) % size;
        } while (pos != initPos);
    }
};

// --- 测试代码 ---
int main() {
    ClosedHashTable table(10);

    table.insert(5, 500);
    table.insert(15, 1500); // 15 % 10 = 5，发生冲突！应该停在 6 号位
    table.insert(25, 2500); // 发生冲突！停在 7 号位

    cout << "Find 15: " << table.find(15) << endl; // 期望: 1500

    table.remove(5); // 删除 5 号位的车，留下墓碑

    // 这是终极测试！如果你的墓碑机制写对了，这里应该能穿过 5 号的墓碑找到 15！
    cout << "Find 15 after remove 5: " << table.find(15) << endl; // 期望: 1500
    cout << "Find 5: " << table.find(5) << endl; // 期望: -1

    return 0;
}
```