---
title: "红黑树"
date: 2020-01-18T14:02:52+08:00
draft: true
tags: 
 - c/C++
 - 数据结构
categories: 
 - C/C++
---









理解红黑树

## 性质

红黑树是一种自平衡的二叉查找树，具有良好的效率，可以在O(logN)时间内完成查找、增加、删除等操作。

红黑树具有如下的性质：

* 节点是红色或黑色的
* 根节点是黑色的
* 所有的叶子节点是黑色的
* 每个红色的节点有两个黑色的子节点（从根节点到叶子节点不能有两个连续的红色节点）
* 从任一节点到其每个叶子节点的所有路径都包含相同数目的黑色节点

**红黑树操作**

* 查找

* 删除

* 插入

* 旋转



对nginx里面实现的红黑树代码进行分析(src/core目录下的ngx_rbtree.h/ngx_rbtree.c)。

```c
struct ngx_rbtree_node_s {
    ngx_rbtree_key_t       key;
    ngx_rbtree_node_t     *left;
    ngx_rbtree_node_t     *right;
    ngx_rbtree_node_t     *parent;
    u_char                 color;
    u_char                 data;
};

typedef struct ngx_rbtree_s  ngx_rbtree_t;

typedef void (*ngx_rbtree_insert_pt) (ngx_rbtree_node_t *root,
    ngx_rbtree_node_t *node, ngx_rbtree_node_t *sentinel);

struct ngx_rbtree_s {
    ngx_rbtree_node_t     *root;
    ngx_rbtree_node_t     *sentinel;
    ngx_rbtree_insert_pt   insert;
};

```

## 旋转

### 左旋

```
LEFT-ROTATE(T, x)  
 y ← right[x]            // 前提：这里假设x的右孩子为y。下面开始正式操作
 right[x] ← left[y]      // 将 “y的左孩子” 设为 “x的右孩子”，即 将β设为x的右孩子
 p[left[y]] ← x          // 将 “x” 设为 “y的左孩子的父亲”，即 将β的父亲设为x
 p[y] ← p[x]             // 将 “x的父亲” 设为 “y的父亲”
 if p[x] = nil[T]       
 then root[T] ← y                 // 情况1：如果 “x的父亲” 是空节点，则将y设为根节点
 else if x = left[p[x]]  
           then left[p[x]] ← y    // 情况2：如果 x是它父节点的左孩子，则将y设为“x的父节点的左孩子”
           else right[p[x]] ← y   // 情况3：(x是它父节点的右孩子) 将y设为“x的父节点的右孩子”
 left[y] ← x             // 将 “x” 设为 “y的左孩子”
 p[x] ← y                // 将 “x的父节点” 设为 “y”
```

![img](https://images0.cnblogs.com/i/497634/201403/251734577643655.jpg)

### 右旋

```
RIGHT-ROTATE(T, y)  
 x ← left[y]             // 前提：这里假设y的左孩子为x。下面开始正式操作
 left[y] ← right[x]      // 将 “x的右孩子” 设为 “y的左孩子”，即 将β设为y的左孩子
 p[right[x]] ← y         // 将 “y” 设为 “x的右孩子的父亲”，即 将β的父亲设为y
 p[x] ← p[y]             // 将 “y的父亲” 设为 “x的父亲”
 if p[y] = nil[T]       
 then root[T] ← x                 // 情况1：如果 “y的父亲” 是空节点，则将x设为根节点
 else if y = right[p[y]]  
           then right[p[y]] ← x   // 情况2：如果 y是它父节点的右孩子，则将x设为“y的父节点的左孩子”
           else left[p[y]] ← x    // 情况3：(y是它父节点的左孩子) 将x设为“y的父节点的左孩子”
 right[x] ← y            // 将 “y” 设为 “x的右孩子”
 p[y] ← x                // 将 “y的父节点” 设为 “x”
```

![img](https://images0.cnblogs.com/i/497634/201403/251737465769614.jpg)



## 删除







## 插入

` 伪代码`

```
RB-INSERT(T, z)  
 y ← nil[T]                        // 新建节点“y”，将y设为空节点。
 x ← root[T]                       // 设“红黑树T”的根节点为“x”
 while x ≠ nil[T]                  // 找出要插入的节点“z”在二叉树T中的位置“y”
     do y ← x                      
        if key[z] < key[x]  
           then x ← left[x]  
           else x ← right[x]  
 p[z] ← y                          // 设置 “z的父亲” 为 “y”
 if y = nil[T]                     
    then root[T] ← z               // 情况1：若y是空节点，则将z设为根
    else if key[z] < key[y]        
            then left[y] ← z       // 情况2：若“z所包含的值” < “y所包含的值”，则将z设为“y的左孩子”
            else right[y] ← z      // 情况3：(“z所包含的值” >= “y所包含的值”)将z设为“y的右孩子” 
 left[z] ← nil[T]                  // z的左孩子设为空
 right[z] ← nil[T]                 // z的右孩子设为空。至此，已经完成将“节点z插入到二叉树”中了。
 color[z] ← RED                    // 将z着色为“红色”
 RB-INSERT-FIXUP(T, z)             // 通过RB-INSERT-FIXUP对红黑树的节点进行颜色修改以及旋转，让树T仍然是一颗红黑树
```

```
RB-INSERT-FIXUP(T, z)
while color[p[z]] = RED                                                  // 若“当前节点(z)的父节点是红色”，则进行以下处理。
    do if p[z] = left[p[p[z]]]                                           // 若“z的父节点”是“z的祖父节点的左孩子”，则进行以下处理。
          then y ← right[p[p[z]]]                                        // 将y设置为“z的叔叔节点(z的祖父节点的右孩子)”
               if color[y] = RED                                         // Case 1条件：叔叔是红色
                  then color[p[z]] ← BLACK                    ▹ Case 1   //  (01) 将“父节点”设为黑色。
                       color[y] ← BLACK                       ▹ Case 1   //  (02) 将“叔叔节点”设为黑色。
                       color[p[p[z]]] ← RED                   ▹ Case 1   //  (03) 将“祖父节点”设为“红色”。
                       z ← p[p[z]]                            ▹ Case 1   //  (04) 将“祖父节点”设为“当前节点”(红色节点)
                  else if z = right[p[z]]                                // Case 2条件：叔叔是黑色，且当前节点是右孩子
                          then z ← p[z]                       ▹ Case 2   //  (01) 将“父节点”作为“新的当前节点”。
                               LEFT-ROTATE(T, z)              ▹ Case 2   //  (02) 以“新的当前节点”为支点进行左旋。
                          color[p[z]] ← BLACK                 ▹ Case 3   // Case 3条件：叔叔是黑色，且当前节点是左孩子。(01) 将“父节点”设为“黑色”。
                          color[p[p[z]]] ← RED                ▹ Case 3   //  (02) 将“祖父节点”设为“红色”。
                          RIGHT-ROTATE(T, p[p[z]])            ▹ Case 3   //  (03) 以“祖父节点”为支点进行右旋。
       else (same as then clause with "right" and "left" exchanged)      // 若“z的父节点”是“z的祖父节点的右孩子”，将上面的操作中“right”和“left”交换位置，然后依次执行。
color[root[T]] ← BLACK
```



`nginx代码`

```c
void
ngx_rbtree_insert(ngx_rbtree_t *tree, ngx_rbtree_node_t *node)
{
    ngx_rbtree_node_t  **root, *temp, *sentinel;

    /* a binary tree insert */

    root = &tree->root;
    sentinel = tree->sentinel;

    if (*root == sentinel) {
        node->parent = NULL;
        node->left = sentinel;
        node->right = sentinel;
        ngx_rbt_black(node);
        *root = node;

        return;
    }

    tree->insert(*root, node, sentinel);

    /* re-balance tree */

    while (node != *root && ngx_rbt_is_red(node->parent)) {
        if (node->parent == node->parent->parent->left) {
            temp = node->parent->parent->right;
            if (ngx_rbt_is_red(temp)) {
                ngx_rbt_black(node->parent);
                ngx_rbt_black(temp);
                ngx_rbt_red(node->parent->parent);
                node = node->parent->parent;
            } else {
                if (node == node->parent->right) {
                    node = node->parent;
                    ngx_rbtree_left_rotate(root, sentinel, node);
                }
                ngx_rbt_black(node->parent);
                ngx_rbt_red(node->parent->parent);
                ngx_rbtree_right_rotate(root, sentinel, node->parent->parent);
            }
        } else {
            temp = node->parent->parent->left;
            if (ngx_rbt_is_red(temp)) {
                ngx_rbt_black(node->parent);
                ngx_rbt_black(temp);
                ngx_rbt_red(node->parent->parent);
                node = node->parent->parent;
            } else {
                if (node == node->parent->left) {
                    node = node->parent;
                    ngx_rbtree_right_rotate(root, sentinel, node);
                }
                ngx_rbt_black(node->parent);
                ngx_rbt_red(node->parent->parent);
                ngx_rbtree_left_rotate(root, sentinel, node->parent->parent);
            }
        }
    }

    ngx_rbt_black(*root);
}

```

