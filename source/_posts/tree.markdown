---
layout:     post
title:      "数据结构（一）－－－树"
subtitle:   "数据结构之树"
date:       2016-09-14
author:     "XuZheng"
# header-img: "post-bg-unix-linux.jpg"
tags:
    - 数据结构
---

> 详细介绍树与二叉树.

### 基本概念

树是n（n>＝0）个结点的有限集合T。当呢n ＝ 0 时，称为空树；当呢n > 0 时改集合应满足如下条件：
1. 其中必有一个称为**根**的特定结点,它没有直接前驱。
2. 剩下的结点被分成ｎ>=0个互不相交的集合T1、T2、......Tn，而且， 这些集合的每一个又都是树。树T1、T2、......Tn被称作根的**子树**(Subtree)。
  - 逻辑结构图如下

    ![](http://olpuebn54.bkt.clouddn.com/tree_img.png)

### 相关术语
- **结点**：包含一个数据元素及若干指向其他结点的分支信息。

- **结点的度**：一个结点的子树个数称为此结点的度。

- **叶结点**：度为0的结点。即无后继结点，也称终端结点。

- **分支结点**：度不为0的结点，特称非终端结点。

- **结点的层次**：从根结点开始，根结点的层次为1，根的直接后继层次为2，以此类推。

- **结点的层次编号**：将树中的结点按上层到下层、同层从左到右的次序排列成一个线性序列，依次给他们编以连续的自然数。

- **树的度**：树中所有结点的度的最大值。

- **树的高度**（深度）：树中所有及诶点层次的最大值。

- **有序树**：在树T中，如果各子树Ti之间是有先后次序，则称为有序树。

- **森林**：m（m>=0）棵互不相交的树的集合。 将一颗非空树的根结点删去，树就变成一个森林，反之，给森林增加一个统一的根结点，森林就变成一颗树。

### 二叉树
  - 定义

  二叉树是树形结构的一个重要类型。许多实际问题抽象出来的数据结构往往是二叉树的形式，即使是一般的树也能简单地转换为二叉树，而且二叉树的存储结构及其算法都较为简单，因此二叉树显得特别重要。
  二叉树(BinaryTree)是n(n≥0)个结点的有限集，它或者是空集(n=0)，或者由一个根结点及两棵互不相交的、分别称作这个根的左子树和右子树的二叉树组成。这个定义是递归的。由于左、右子树也是二叉树， 因此子树也可为空树。下图中展现了五种不同基本形态的二叉树。

  ![](http://olpuebn54.bkt.clouddn.com/binary_tree_img.png)
   - (a) 为空树
   - (b) 为仅有一个结点的二叉树
   - (c) 是仅有左子树而右子树为空的二叉树
   - (d) 是仅有右子树而左子树为空的二叉树
   - (e) 是左、右子树均非空的二叉树。

    这里应特别注意的是，二叉树的左子树和右子树是严格区分并且不能随意颠倒的，图 (c) 与图 (d) 就是两棵不同的二叉树。
  - 重要的概念

    - **完全二叉树**——只有最下面的两层结点度小于2，并且最下面一层的结点都集中在该层最左边的若干位置的二叉树；

    - **满二叉树**——除了叶结点外每一个结点都有左右子叶且叶结点都处在最底层的二叉树,。
  - 性质

    - **性质1**：在二叉树的第i层上最多有2 i-1 个节点 。（i>=1）

    - **性质2**：二叉树中如果深度为k,那么最多有2k-1个节点。(k>=1）

    - **性质3**：n0表示度数为0的节点 n2表示度数为2的节点

    - **性质4**：在完全二叉树中，具有n个节点的完全二叉树的深度为[log2n]+1，其中[log2n]+1是向下取整。

  - 遍历
    - 先序遍历

          void PreOrder(BiTree root){
            if(root != null){
                Visit(root.value);    //访问根结点
                PreOrder(root.lNext); //先序遍历左子树
                PreOrder(root.rNext); //先序遍历右子树
            }
          }
    - 中序遍历
          void InOrder(BiTree root){
            if(root != null){
                InOrder(root.lNext);  //中序遍历左子树
                Visit(root.value);    //访问根结点
                InOrder(root.rNext);  //中序遍历右子树
            }
          }
    - 后序遍历
          void InOrder(BiTree root){
            if(root != null){
                InOrder(root.lNext);  //后序遍历左子树
                InOrder(root.rNext);  //后序遍历右子树
                Visit(root.value);    //访问根结点
            }
          }

### 简单二叉树算法

  - 输出二叉树中的叶子结点

    ***使用先序遍历输出二叉树中的叶子结点***

        void PreOrder(BiTree root){
          if(root != null){
              if(BiTree.lNext == null && BiTree.rNext == null){
                System.out.print(BiTree.value) //输出叶子结点
              }
              PreOrder(root.lNext); //先序遍历左子树
              PreOrder(root.rNext); //先序遍历右子树
          }
        }

   ***后续遍历求二叉树高度***

         int PostTreeDepth(BiTree bt){
           int h1,hr,max;
           if(bt != null){
             h1 = PostTreeDepth(bt.lNext); // 求左子树的深度
             hr = PostTreeDepth(bt.rNext); // 求右子树的深度
             max = h1 > hr ? h1 : hr;      // 得到左右子树深度较大者
             return max+1;                 // 返回深度
           }else {
             return 0;              // 如果是空树，返回0
           }
         }
   ***先序遍历求二叉树高度***

        int dept = 1;
        int PreTreeDepth(BuTree bt,int h){
          // h为bt指向的结点所在层数，初值为1
          if(bt != null){
            if(h > dept){
              dept = h; // 如果该结点层数值大于dept，则更新dept的值
              PreTreeDepth(bt.lNext, h+1) //遍历左子树
              PreTreeDepth(bt.rNext, h+1) //遍历右子树
            }
          }
        }
