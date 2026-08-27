---
tags: [位操作, 算法, 嵌入式]
source: Data_Struct_Implementation/BitsManipulation/singleNumber
created: 2026-08-27
---

# 只出现一次的数字（Single Number）

给定一个数组，除某个元素只出现一次外，其余每个元素都出现三次。找出那个只出现一次的元素。

## 方法一：哈希（Hash）

```c++
class Solution {
public:
    void count_bits(int arr[], int val) {
        for (int i = 0; i < 32; i++) {
            if (val & (0x1 << i))
                arr[i] ++;
        }
    }
    
    int singleNumberII(vector<int>& nums) {
        int arr[32] = {};
        int ret = 0;
        
        for (auto num : nums) {
            count_bits(arr, num);
        }
        
        for (int i = 0; i < 32; i++) {
            arr[i] = arr[i]%3;
            ret |= (arr[i] << i);
        }
        
        return ret;
    }
};
```

## 方法二：位运算

```c
int getSingle(int arr[], int n)
{
    int ones = 0, twos = 0;
 
    int common_bit_mask;
 
    // 以 {3, 3, 2, 3} 为例来理解本算法
    for (int i = 0; i < n; i++) {
        /* 表达式 "ones & arr[i]" 给出既在 'ones' 又在
           新元素 arr[] 中出现的位。我们用按位或把这些位加到 'twos' 中。

           'twos' 的值在第 1、2、3、4 次迭代后分别被置为 0, 3, 3, 1 */
        twos = twos | (ones & arr[i]);
 
        /* 将新位与之前的 'ones' 异或，得到所有出现奇数次的位

           'ones' 的值在第 1、2、3、4 次迭代后分别被置为 3, 0, 2, 3 */
        ones = ones ^ arr[i];
 
        /* 公共位是那些出现第三次的位。
           所以这些位不应同时出现在 'ones' 和 'twos' 中。
           common_bit_mask 把这些位都置为 0，从而可从 'ones' 和 'twos' 中移除它们。

           'common_bit_mask' 的值在第 1、2、3、4 次迭代后分别被置为 00, 00, 01, 10 */
        common_bit_mask = ~(ones & twos);
 
        /* 从 'ones' 中移除公共位（第 3 次出现的位）

           'ones' 的值在第 1、2、3、4 次迭代后分别被置为 3, 0, 0, 2 */
        ones &= common_bit_mask;
 
        /* 从 'twos' 中移除公共位（第 3 次出现的位）

           'twos' 的值在第 1、2、3、4 次迭代后分别被置为 0, 3, 1, 0 */
        twos &= common_bit_mask;
 
        // 取消注释以查看中间值
        // printf (" %d %d n", ones, twos);
    }
 
    return ones;
}
 
int main()
{
    int arr[] = { 3, 3, 2, 3 };
    int n = sizeof(arr) / sizeof(arr[0]);
    printf("The element with single occurrence is %d ",
           getSingle(arr, n));
    return 0;
}
```

## 分析

- **方法一（哈希）**：统计每个比特位在所有数字中出现的次数，对 3 取模。出现 3 次的数字在每个比特位上会被抵消，剩下的就是只出现一次的数字。时间 O(32n)，空间 O(32)。
- **方法二（位运算）**：用 `ones` 和 `twos` 两个变量模拟“每个比特位出现次数 mod 3”的状态机。`ones` 记录出现 1 次的位，`twos` 记录出现 2 次的位，出现第 3 次时二者同时清除。时间 O(n)，空间 O(1)，非常高效。

## 相关文档
- [[flipBitsNumber]] —— 位运算计数
- [[countBitsLookUpTable]] —— 统计置位个数
