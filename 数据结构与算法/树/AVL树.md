## AVL树-平衡二叉搜索树
{docsify-updated}

- [AVL树-平衡二叉搜索树](#avl树-平衡二叉搜索树)
    - [创建平衡二叉搜索树](#创建平衡二叉搜索树)
    - [Top-Down 方法](#top-down-方法)
    - [Bottom-Up 方法](#bottom-up-方法)

AVL是 Adelson-Velsky 和E.M.Landis 名称的缩写，他们提出的平衡二叉树的概念，为了纪念他们，将平衡二叉树称为AVL树。

首先，让我们定义一下平衡二叉搜索树的含义：
+ 平衡二叉树是一棵二叉搜索树树，它的高度是 $O(log(n))$，其中n是树内的节点数。
+ 对于平衡树内的每个节点，左子树的高度与右子树的高度相差不得超过 1。

#### 创建平衡二叉搜索树
在创建平衡 BST 时，我们需要牢记高度条件。首先，让我们考虑一下将哪个节点作为根节点最好。

**由于我们需要平衡树，所以必须将中间的值作为根节点。然后，我们可以将中间值之前的值添加到树的左边。因此，所有较小的值都将被添加到左侧子树。同样，我们将中间值之后的值添加到树的右边，这样所有较大的值都会添加到右边的子树中。**

不过，获取中间的元素取决于列表的类型。如果列表支持随机存取（如数组），我们可以简单地获取中间元素，将其作为根，然后递归地添加左右子树。我们称这种方法为Top-down法。

但是，如果我们只有指向列表中第一个元素的指针，那么获取中间元素就比较麻烦了。因此，在这种情况下，我们将使用Bottom-up的方法。

#### Top-Down 方法

自上而下的方法使用排序数组来创建平衡的 BST。因此，我们可以在恒定时间内访问数组内的任意索引：
<center><img src="pics/quicklatex.com-bd18cef6ee8eb252f700ebf114b4d5fa_l3.svg" alt=""></center>

该函数的初始调用传递数组 $A$，$L$为0，$R$为 $n-1$，其中$n$为数组的大小。

一开始，我们会检查是否到达了一个空范围。在这种情况下，我们只需返回 null，表示对象为空。

接下来，我们获取中间索引并用 $A[mid]$初始化 root 之后，我们进行两次递归调用。第一次调用的对象是范围$[L, mid-1]$ , 表示索引$mid$之前的元素。另一方面，第二次调用的是范围 $[mid+1, R]$，它对应于索引$mid$之后的元素。

第一次调用返回左子树的根。因此，我们将其赋值给根节点的左指针。同样，第二次调用会返回右侧子树的根节点。因此，我们将其赋值给根节点的右指针。

自顶向下方法的复杂度为 $ O(n)$，其中为$n$数组内部元素的个数。

```
class BalanceBstree extends BsTree {

    public BalanceBstree topDown(int[] arr) {
        BalanceBstree balanceBstree = new BalanceBstree();
        root = build(arr, 0, arr.length - 1);
        return balanceBstree;
    }

    private Node build(int[] arr, int left, int right) {
        if (left > right) {
            return null;
        }

        int mid = (left + right) / 2;
        Node root = new Node(arr[mid]);
        root.left = build(arr, left, mid - 1);
        root.right = build(arr, mid + 1, right);
        return root;
    }
}
```

#### Bottom-Up 方法

自下而上的方法使用链表来构建平衡的 BST。因此，我们会有一个指向列表头部的指针，我们只能在恒定时间内向前移动这个指针。
<center><img src="pics/quicklatex.com-0ab014825b9004386afe016d6b44ec97_l3.svg" alt=""></center>