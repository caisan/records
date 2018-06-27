---
layout: post
title: 红黑树的原理及实现
tags:
- data-structure
categories: data-structure
description: 红黑树的原理及实现
---


本文我们主要介绍一下红黑树的实现原理。在了解红黑树之前，请先参看```2-3树```以及```avl树```的相关实现。


<!-- more -->


## 1. 红黑树

红黑树(Red-Black Tree)是一种自平衡二叉查找树。是在计算机科学中用到的一种数据结构，典型的用途是实现关联数组。它是在1972年由```Rudolf Bayer```发明的， 当时被称为平衡二叉B树(symmetric binary B-trees)。后来，在1978年被```Leo J. Guibas```和```Robert Sedgewich```修改为如今的```红黑树```。红黑树和```AVL树```类似， 都是在进行插入和删除操作时通过特定操作保持二叉查找树的平衡， 从而获得较高的查找性能。它虽然复杂， 但它的最坏情况运行时间也是非常好的，并且在实践中是高效的： 它可以在```O(logn)```时间内做查找、插入和删除操作， 这里n是树中元素的数目。


### 1.1 数据结构
红黑树的统计性能要好于```平衡二叉树```(即AVL树）， 因此红黑树在很多地方都有应用。在```C++ STL```中，很多部分（包括set、multiset、map、multimap）应用了红黑树的变体（SGI STL中的红黑树有一些变化，这些修改提供了更好的性能，以及对set操作的支持）。其他的平衡树还有： ```AVL树```、```SBT树```、```伸展树```、```TREAP树```。

### 1.2 红黑树的性质

红黑树是每个节点都带有颜色属性的二叉查找树， 颜色或```红色```或```黑色```。 在二叉查找树一般的要求外，对于任何有效的红黑树我们增加了如下的额外要求：

* 性质1： 节点是红色或黑色

* 性质2： 根节点是黑色

* 性质3： 每个叶节点(nil节点, 空节点)是黑色的。 注意： 这里的叶子节点是```nil叶子```

* 性质4： 每个红色节点的两个子节点都是黑色。（从每个叶子到根的所有路径上不能有两个连续的红色节点）

* 性质5： 从任一节点到其每个```叶子```的所有路径都包含相同数目的黑色节点

<pre>
注意： 
1） 红黑树中的叶子节点均指nil叶子

2） 性质5确保没有一条路径会比其他路径长出2倍。因而，红黑树是相对接近平衡的二叉树
</pre>

![ds-rb-tree](https://ivanzz1001.github.io/records/assets/img/data_structure/ds_rb_tree.jpg)


正是由于上面的这些约束产生了红黑树的关键性质： 从根到叶子的最长的可能路径不多于最短的可能路径的两倍。结果是这棵树大致上是平衡的。因为如插入、删除和查找某个值的在最坏情况下的时间复杂度为树的高度， 这个高度上的理论上限允许红黑树在最坏情况下都是高效的，而不同于普通的二叉查找树。

要知道为什么这些特性确保了这个结果， 我们注意到```性质4```导致了路径不可能有两个毗连的红色节点就足够了。最短的可能路径都是黑色节点，最长的可能路径有交替的红色和黑色节点。因为根据```性质5```所有最长的路径都有相同数目的黑色节点，这就表明了没有路径能多于任何其他路径的两倍长。

### 1.3 红黑树与2-3树
红黑树的```另一种定义```是满足下列条件的二叉查找树：

* 红链接均为左链接

* 没有任何一个节点同时和两条红链接相连

* 该树是完美黑色平衡的，即任意空链接到根节点的路径上的黑色链接数量相同

如果我们```将一个红黑树中的红链接画平```，那么所有的空链接到根节点的距离都是相同的； 如果我们将由红链接相连的节点合并，得到的就是一棵```2-3树```。相反，如果将一棵```2-3树```中的```3-节点```画作由红色左链接相连的两个```2-节点```, 那么不会存在能够和两条红链接相连的节点， 且树必然是完美平衡的。

![ds-rb-23-tree](https://ivanzz1001.github.io/records/assets/img/data_structure/ds_rb_23_tree.jpg)

## 2. 旋转的定义

因为很多书中对旋转的定义不一致，所以我们有必要在这里说明一下。假设```红黑树```节点数据结构如下：
{% highlight string %}
typedef struct RBNode{
   int key;
   unsigned char color;
   struct RBNode *left;
   struct RBNode *right;
   struct RBNode *parent;
}rb_node_t, *rb_tree_t;

{% endhighlight %}

* 以某一节点为轴，它的左枝顺时针旋转，作为新子树的根， 我们称之为```顺时针旋转```（clockwise)或者```右旋转```;

![ds-node-right-rotate](https://ivanzz1001.github.io/records/assets/img/data_structure/ds_node_right_rotate.jpg)

源代码如下：
{% highlight string %}
static struct rb_node_t *rb_rotate_right(rb_tree_t *root, rb_node_t *node)
{
    rb_node_t *left = node->left;

    if(node->left = left->right)
    {
       left->right->parent = node;
    }

    left->right = node;
   
    if(left->parent = node->parent)
    {
       if(node->parent->left == node)
       {
          node->parent->left = left;
       }
       else{
          node->parent->right = left;
       }
    }
    else{
       *root = left;
    }
   
    node->parent = left;
    return left;
}
{% endhighlight %}


* 以某一节点为轴，它的右枝逆时针旋转，作为新子树的根， 我们称为```逆时钟旋转```(anti clockwise)或者```左旋转```;

![ds-node-left-rotate](https://ivanzz1001.github.io/records/assets/img/data_structure/ds_node_left_rotate.jpg)

源代码如下：
{% highlight string %}
static struct rb_node_t *rb_rotate_left(rb_tree_t *root, rb_node_t *node)
{
    rb_node_t *right = node->right;

    if(node->right = right->left)
    {
       right->left->parent = node;
    }
    
    right->left = node;
    
    if(right->parent = node->parent)
    {
        if(node->parent->left == node)
        {
             node->parent->left = right;
        }
        else{
             node->parent->right = right;
        }
    }
    else{
       *root = right;
    }

    node->parent = right;
    return right;
}
{% endhighlight %}



## 3. 红黑树的插入

### 3.1 红黑树插入的基本原理
和```AVL树```一样，在插入和删除节点之后，红黑树也是通过旋转来调整树的平衡的。红黑树插入```节点z```的方法和普通二叉搜索树一样，都是将新```节点z```作为一个叶子节点插入到树的底部。不同的是，红黑树将```新节点z```作为一个红色节点，将其孩子指针指向```nil叶子```, 然后当新节点z的父节点为红色时，由于违反了```性质4```，因此需要对其进行调整（如果```新节点z```的父节点为黑色，且z本身是红色，因此不会违反任何性质）。
<pre>
红黑树调整算法的设计要遵循一个原则： 同一时刻红黑树只能违反最多一条性质。
</pre>

红黑树插入```节点z```后的调整有3种情况：

**(1) z的叔节点y是红色的**

![ds-rb-insert-1](https://ivanzz1001.github.io/records/assets/img/data_structure/ds_rb_insert_case1.jpg)

左图中插入的```新节点z```是一个红色节点，其父节点A是红色的，违反了```性质4```，所以需要进行调整（由于节点A是红色的，根据性质4，因树本身是平衡的，所以节点C必然是黑色的）。因为其```叔节点y```是红色的，于是可以修改节点A、节点B为黑色，此时节点C的黑高就会发生变化， 从原来的1（忽略子树a、b、r、d、e的黑高）变成了2， 因此还需要将节点C变成红色以保持其黑高不变。此时，由于节点C由黑色变成了红色，如果节点C的父节点是红色，那么会违反性质4， 于是节点C变成了```新的节点z```， 从这里开始向上回溯调整树。

注意：

* 对于新插入的节点z是节点A的左子树的情况与上述一致；

* 对于新插入的节点z是节点C的右子树的节点的情况与上述对称；

```情况1```是一种比较简单的情况。

**(2) z的叔节点y是黑色，且z是一个右孩子**

```情况2```不能向```情况1```那样通过修改z的父节点的颜色来维持```性质4```， 因为如果将z的父节点变成了黑色， 那么树的黑高就会发生变化， 必然会引起对性质5的违反。以上面情况1的图为例， 假设此时节点y为黑色， 那么节点C的右子树高度为2（忽略子树d和e）， 左子树高也相同（因为树是平衡的）， 如果简单的修改节点A为黑色， 那么节点C的左子树的黑高会比右子树大1， 此时即使将节点C修改为红色也于事无补。

此时可以通过旋转节点z的父节点使```情况2```变成```情况3```进行处理。
<pre>
注： 此种情况只可能在调整过程中出现
</pre>

**(3) z的叔节点是黑色，且z是一个左孩子**

```情况2```转变成```情况3```, 然后针对```情况3```进行处理的流程如下：

![ds-rb-insert-23](https://ivanzz1001.github.io/records/assets/img/data_structure/ds_rb_insert_case23.jpg)

```情况2```通过对节点A进行一次左旋转变成```情况3```，此时节点z不再是原来的B，而是节点A了， 此时树依然只是违反性质4。情况3通过对节点C进行了一次右旋转，然后改变节点B和节点C的颜色，得到右图。
<pre>
注： 情况3也只可能在调整过程中出现
</pre>

先来证明这以操作的正确性：

对于左图， 由于刚插入节点z的时候，只违反了```性质4```，性质5依然满足，假设子树a的黑高为ha，子树b的黑高为hb，依次类推，可知道ha==hb==hr==hd, hC=hd+1,  对节点A进行左旋转变成情况3（即中图）， 树依然只违反性质4， 新的节点z变为节点A。之后再对节点C进行右旋转并修改颜色得到右图， 此时节点A和节点C是平衡的， 节点B也是平衡的， 而且节点B的黑高为```hd+1```。由此可知，整个操作后， 该树的黑高不变，且满足所有红黑树的性质。

在红黑树的调整过程中，z始终指向一个红色节点，因此```z```永远不会影响其所在树的黑高，于是我们始终关注```节点z```的父节点是否为红色，如果是则意味着违反了性质4，需要进行调整； 否则，就可以退出循环了。在算法的最后，我们还需要关注```性质2```，将根节点的颜色改为黑色，根节点的颜色改变也是绝对不会引起树的不平衡的， 而将其改为黑色也是不会引起对性质4的违反的。 


### 3.2 插入相关算法








<br />
<br />
**[参看]:**


1. [查找（一）史上最简单清晰的红黑树讲解](http://blog.csdn.net/yang_yulei/article/details/26066409)

2. [红黑树的插入与删除](http://m.blog.csdn.net/article/details?id=51504764)

3. [浅谈算法和数据结构： 九 平衡查找树之红黑树](http://www.cnblogs.com/yangecnu/p/Introduce-Red-Black-Tree.html) 

4. [数据结构： 2-3树与红黑树](http://blog.csdn.net/aircattle/article/details/52347955)

5. [数据结构与算法](https://blog.csdn.net/hello_world_lvlcoder/article/category/6655685/1)

6. [红黑树(一)之 原理和算法详细介绍](http://www.cnblogs.com/skywang12345/p/3245399.html)
<br />
<br />
<br />

