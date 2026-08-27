---
tags: [数据结构, 算法, 嵌入式, 树, 二叉搜索树]
source: Data_Struct_Implementation/BST
created: 2026-08-27
---

# 二叉搜索树（非平衡实现）

#### 用法
```
make
./bst
```

## 代码

源码文件：`BST/bst.c`

```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

#define ASSERT_M(x, message) \
do { \
 if (!x) \
    printf("%s\n", message); \
} while (0);

#define TESTCASE(case, state) \
do { \
    if (state) { \
        printf("%s passes!\n", case); \
        printf("---------------\n"); \
    } \
    else \
        printf("%s starts!\n", case); \
} while (0);

typedef struct bstNode {
    int val;
    struct bstNode* left;
    struct bstNode* right;
} bstNode;

static bstNode* insertNode(bstNode* head, bstNode* inNode) {
    if (!head)
        return inNode;

    if (inNode->val < head->val)
        head->left = insertNode(head->left, inNode);
    else
        head->right = insertNode(head->right, inNode);

    return head;
}

static bstNode* searchParent(bstNode* head, bstNode* node) {
    if (head->left == node || head->right == node)
        return head;
    
    if (node->val < head->val)
        return searchParent(head->left, node);
    
    if (node->val >= head->val)
        return searchParent(head->right, node);

    return NULL;
}

bstNode* newNode(int val) {
    bstNode* new_node;

    new_node = (bstNode*) malloc(sizeof(bstNode));
    if (!new_node)
        return NULL;
    
    new_node->val = val;
    new_node->left = NULL;
    new_node->right = NULL;

    return new_node;
}

bstNode* search(bstNode* head, int value) {
    if (!head)
        return NULL;

    if (head->val == value)
        return head;
    
    return (value < head->val) ? search(head->left, value) : search(head->right, value); 
}

bstNode* insertVal(bstNode* head, int val) {
    if (!head)
        return newNode(val);

    if (val < head->val)
        head->left = insertVal(head->left, val);
    else
        head->right = insertVal(head->right, val);

    return head;
}

// 总是用 new_node->right 作为子分支的新头
// 返回新的头指针
bstNode* deleteNode(bstNode* head, bstNode* node) {
    bstNode* new_head = head;
    bstNode* dummy;

    if (!node || !head)
        return head;

    if (head == node) {
        new_head = insertNode(head->right, head->left);
        free(head);
        return new_head;
    }

    // 叶子节点，必须将其父指针置 NULL 以避免悬垂指针问题
    if (!node->right && !node->left) {
        dummy = searchParent(head, node);
        // 若该节点不属于当前树，则不做任何事
        if (!dummy) return head;
        else if (dummy->left == node) dummy->left = NULL;
        else if (dummy->right == node) dummy->right = NULL;
        free(node);
    }
    else {
        dummy = insertNode(node->right, node->left);
        if (dummy) {
            memcpy(node, dummy, sizeof(bstNode));
            free(dummy);
        }
    }
    
    return new_head;
}

int minVal(bstNode* head) {
    if (!head)
        return -1;

    while (head->left)
        head = head->left;
    
    return head->val;
}

int maxVal(bstNode* head) {
    if (!head)
        return -1;

    while (head->right)
        head = head->right;
    
    return head->val;
}

void traverseInOrder(bstNode* head) {
    if (!head)
        return;

    traverseInOrder(head->left);
    printf("val: %d,", head->val);
    traverseInOrder(head->right);
}

bstNode* BST_head = NULL;

int main() {
    int i;
    int array[] = {5,7,2,3,4,1,6,8,9};

    for (i = 0; i < 9; i++) {
        BST_head = insertVal(BST_head, array[i]);
    }

// test traverse
    TESTCASE("Case 1: In order print.", 0);
    traverseInOrder(BST_head);
    printf("\n============\n");
    TESTCASE("Case 1: In order print.", 1);

// test insertVal()
    TESTCASE("Case 2: Insert val into BST.", 0);
    BST_head = insertVal(BST_head, 9);
    traverseInOrder(BST_head);
    printf("\n============\n");

    BST_head = insertVal(BST_head, 0);
    traverseInOrder(BST_head);
    printf("\n============\n");

    BST_head = insertVal(BST_head, 17);
    traverseInOrder(BST_head);
    printf("\n============\n");
    TESTCASE("Case 2: Insert val into BST.", 1);

// test search and delete
    TESTCASE("Case 3: Search and delete.", 0);
    bstNode *test = NULL;

    ASSERT_M((search(BST_head, 9) != NULL), "Node should be found in the BST");
    ASSERT_M((search(BST_head, 1) != NULL), "Node should be found in the BST");
    ASSERT_M((search(BST_head, 100) == NULL), "Node should not be found in the BST");

// delete mid node

    traverseInOrder(BST_head);
    printf("\n============\n");

    test = search(BST_head, 1);
    if (test) {
        printf("Delete %d from BST\n", test->val);
        BST_head = deleteNode(BST_head, test);
        ASSERT_M((search(BST_head, 1) == NULL), "Node should Not be found in the BST");
    }

    traverseInOrder(BST_head);
    printf("\n============\n");

     test = search(BST_head, 17);
    if (test) {
        printf("Delete %d from BST\n", test->val);
        BST_head = deleteNode(BST_head, test);
        ASSERT_M((search(BST_head, 17) == NULL), "Node should Not be found in the BST");
    }

    traverseInOrder(BST_head);
    printf("\n============\n");

// delete head
    printf("Delete head %d from BST\n", BST_head->val);
    BST_head = deleteNode(BST_head, BST_head);
    traverseInOrder(BST_head);
    printf("\n============\n");
    TESTCASE("Case 3: Search and delete.", 1);

// test minvalue & maxval
    TESTCASE("Case 4: Min and Max.", 0);
    printf("Min val: %d  Max val: %d\n", minVal(BST_head), maxVal(BST_head));
    TESTCASE("Case 4: Min and Max.", 1);

    struct bstNode *root = NULL;
    
    root = insertVal(root, 20);
    insertVal(root,5);
    insertVal(root,1);
    insertVal(root,15);
    insertVal(root,9);
    insertVal(root,7);
    insertVal(root,12);
    insertVal(root,30);
    insertVal(root,25);
    insertVal(root,40);
    insertVal(root,45);
    insertVal(root,42);

    traverseInOrder(root);
    printf("\n============\n");

    return 0;
}
```

## 改进（Improvement）

上述实现是**非平衡 BST**，意味着最坏情况下查找时间为 O(n) 而非 O(log n)（例如：给定一个已排序的数组）。

对于已排序数组 {1,2,3,4,5}，我们的 BST 会呈现如下形状（退化成链表）：

```
1
 \
  3
   \
    4
     \
      5
```

而最优的**平衡 BST** 会是这样：

```
         3
       /   \
      1     4
       \     \
        2     5
```

另一项改进是：用 for/while 循环代替递归，从而为嵌入式设备节省额外的栈空间。

## 复杂度
- 查找 / 插入 / 删除（平均）：O(log n)
- 查找 / 插入 / 删除（最坏）：O(n)，当树退化为链表时

## 参考
https://www.codesdope.com/blog/article/binary-search-tree-in-c/

## 相关文档
- [[binaryHeap]] —— 二叉堆，另一种树形结构
- [[hashTable]] —— 哈希表，查找性能更优
