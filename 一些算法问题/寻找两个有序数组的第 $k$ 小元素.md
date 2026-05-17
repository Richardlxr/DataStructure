**核心标签：** `分治法 (Divide and Conquer)`、`二分查找 (Binary Search)`、`竞技编程`

**时间复杂度：** $O(\log(n+m))$

**空间复杂度：** $O(1)$

## 一、 核心算法思想：降维打击

在两个有序数组 $a$ 和 $b$ 中寻找第 $k$ 小的元素，如果使用双指针合并扫描，时间复杂度是 $O(k)$。为了达到 $O(\log(n+m))$，必须采用**每次排除一半不可能答案**的二分思想。

**破局点（核心推论）：**

分别考察两个数组的第 $\lfloor k/2 \rfloor$ 个元素：$a[k/2]$ 和 $b[k/2]$。

如果 $a[k/2] < b[k/2]$，那么 $a[1]$ 到 $a[k/2]$ 这部分元素，**绝对不可能**是两数组合并后的第 $k$ 小元素。

_证明：极限情况下，即使 $b$ 数组的前 $k/2 - 1$ 个元素全比 $a[k/2]$ 小，合并后比 $a[k/2]$ 小的元素总数最多也只有 $(k/2 - 1) + (k/2 - 1) = k - 2$ 个。因此 $a[k/2]$ 及其前面的元素必须被淘汰。_

---

## 二、 避坑指南（高频易错点复盘）

在从逻辑推导走向代码实现的过程中，极易踏入以下四个陷阱：

### ❌ 陷阱 1：C++ 运算符优先级背刺

- **错误写法：** `target >> 1 + begin_a - 1`
    
- **致命原因：** C++ 中加减法（`+`, `-`）的优先级**高于**位移运算符（`>>`）。编译器会将其解析为 `target >> (1 + begin_a - 1)`，导致寻址完全崩溃。
    
- **正确对策：** 凡是位运算替代除法，**必须加括号**：`(target >> 1) + begin_a - 1`。
    

### ❌ 陷阱 2：逻辑配额与物理边界混淆

- **错误想法：** 维护一个 `end_a`，如果 `end_a > k` 则认为数组越界。
    
- **致命原因：** 数组是否失去比较资格，只取决于它的**物理内存长度** $n$。如果要找第 100 小的数（$k=100$），但数组只有 2 个元素（$n=2$），数组依然有资格参与前两轮的比较。
    
- **正确对策：** 彻底抛弃 `end` 变量。真正的越界退出条件是起点指针超出了物理内存：`start_a > n`。
    

### ❌ 陷阱 3：内存越界引发的 Segmentation Fault

- **错误写法：** `int mid_a = start_a + (target >> 1) - 1;` 接着直接访问 `a[mid_a]`。
    
- **致命原因：** 当数组剩余元素不足 `target / 2` 时，直接计算出的 `mid_a` 会超出数组最大下标 $n$，导致非法内存访问。
    
- **正确对策：** 采用防御性编程，使用 `std::min` 兜底，强行把越界的指针“按”在数组的最后一位：`int mid_a = std::min(start_a + (target >> 1) - 1, n);`。
    

### ❌ 陷阱 4：刻舟求剑式的目标扣减

- **错误写法：** 淘汰数据后，无脑执行 `target = target - (target >> 1)`。
    
- **致命原因：** 因为引入了上一步的 `std::min` 兜底，你实际跨过（淘汰）的元素数量，可能由于数组过短而**小于**预期的 `target / 2`。如果盲目扣减，会导致目标 $k$ 缩水过快，最终结果偏小。
    
- **正确对策：** 锱铢必较。计算实际在内存中跳过了几个元素：`target = target - (mid_a - start_a + 1)`。
    

---

## 三、 完美代码实现（四步闭环逻辑）

在 `while(true)` 循环中，严格按照以下四个优先级步骤执行，做到滴水不漏：

```c
#include <iostream>

#include <algorithm> // 必须引入，为了使用 std::min

const int MAX_SIZE = 200005;
int a[MAX_SIZE];
int b[MAX_SIZE];
int n, m, q;

int Search(int k) {
    int start_a = 1;
    int start_b = 1;
    int target = k; // 剩余需要寻找的目标配额

    while (true) {
        // 第一步：物理耗尽的边界（优先判断）
        // 如果某个数组的元素已经被全部淘汰，剩下的名额全从另一个数组里拿
        if (start_a > n) {
            return b[start_b + target - 1];
        }
        if (start_b > m) {
            return a[start_a + target - 1];
        }

        // 第二步：逻辑终点的边界
        // 经过不断减半，只需要找剩下元素中的第 1 小，直接比对两军主帅
        if (target == 1) {
            return (a[start_a] < b[start_b]) ? a[start_a] : b[start_b];
        }

        // 第三步：探测时的“撞墙”保护
        // 往前推进 target/2 步，但绝不能超出数组的物理边界 n 或 m
        int mid_a = std::min(start_a + (target >> 1) - 1, n);
        int mid_b = std::min(start_b + (target >> 1) - 1, m);

        // 第四步：锱铢必较的配额扣减
        if (a[mid_a] < b[mid_b]) {
            // 淘汰 a 数组前半部分
            // 必须减去实际淘汰的元素个数 (mid_a - start_a + 1)
            target = target - (mid_a - start_a + 1);
            start_a = mid_a + 1; // 指针后移
        } else {
            // 淘汰 b 数组前半部分
            target = target - (mid_b - start_b + 1);
            start_b = mid_b + 1;
        }
    }
}

int main() {
    // 优化输入输出流速度（竞技编程常用）
    std::ios_base::sync_with_stdio(false);
    std::cin.tie(NULL);

    std::cin >> n >> m >> q;
    for (int i = 1; i <= n; i++) {
        std::cin >> a[i];
    }
    for (int i = 1; i <= m; i++) {
        std::cin >> b[i];
    }

    int k;
    for (int i = 1; i <= q; i++) {
        std::cin >> k;
        std::cout << Search(k) << '\n';
    }
    return 0;
}
```