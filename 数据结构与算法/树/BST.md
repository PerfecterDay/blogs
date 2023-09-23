## BST- Binary Search Tree,Binary Sorted Tree
{docsify-updated}
> https://www.baeldung.com/cs/sorting-binary-tree  
> https://www.baeldung.com/cs/balanced-bst-from-sorted-list

- [BST- Binary Search Tree,Binary Sorted Tree](#bst--binary-search-treebinary-sorted-tree)
  - [BST-二叉搜索树的定义](#bst-二叉搜索树的定义)
  - [BST-二叉搜索树的构造](#bst-二叉搜索树的构造)
  - [平衡二叉搜索树](#平衡二叉搜索树)
    - [创建平衡二叉搜索树](#创建平衡二叉搜索树)
    - [Top-Down 方法](#top-down-方法)
    - [Bottom-Up 方法](#bottom-up-方法)


### BST-二叉搜索树的定义
二叉搜索树、二叉排序树或者叫BST，是一棵满足以下条件的二叉树：
+ 对于每个节点，左子树的节点的值都小于该节点值
+ 对于每个节点，右子树的节点的值都大于该节点值
+ 递归地，左子树和右子树也是BST

BST 的中序遍历就是一个排序。

### BST-二叉搜索树的构造
构造BST：
<center><img src="pics/bst-construct.svg" alt=""></center>

```
public class BinarySearchTree {
    public static void main(String[] args) {
        BsTree bsTree = new BsTree();
        bsTree.insert(8);
        bsTree.insert(2);
        bsTree.insert(3);
        bsTree.insert(9);
        bsTree.insert(10);
        bsTree.insert(1);
        bsTree.insert(20);
        System.out.println("in order");
        bsTree.inorderTraverse();
        System.out.println("pre order");
        bsTree.preorderTraverse();
        System.out.println("post order");
        bsTree.postorderTraverse();
        bsTree.bfsTraverse();
    }
}

class Node {
    int value;
    Node left;
    Node right;

    public Node(int value) {
        this.value = value;
        this.left = null;
        this.right = null;
    }
}

class BsTree {
    Node root = null;

    public BsTree insert(int value) {
        root = insert(this.root, value);
        return this;
    }

    private Node insert(Node node, int value) {
        if (node == null) {
            node = new Node(value);
            return node;
        } else {
            if (value < node.value) {
                node.left = insert(node.left, value);
            } else {
                node.right = insert(node.right, value);
            }
        }
        return node;
    }

    public void inorderTraverse() {
        recInorderTraverse(this.root);
        System.out.println("-----------------");
        iteInorderTraverse(this.root);
    }

    public void preorderTraverse() {
        itePreorderTraverse(this.root);
    }

    public void postorderTraverse() {
        itePostOderTraverse(this.root);
    }

    /*
    * 递归的中序遍历，返回的排序的数列
    * */
    private void recInorderTraverse(Node node) {
        if (node != null) {
            recInorderTraverse(node.left);
            System.out.println(node.value);
            recInorderTraverse(node.right);
        }
    }

    /*
     * 借助栈实现迭代的中序遍历，返回的排序的数列
     * */
    private void iteInorderTraverse(Node node) {
        if (node == null){
            return;
        }
        Stack<Node> stack = new Stack();
        while (node != null) {
            stack.push(node);
            node = node.left;
        }
        while (!stack.isEmpty()){
            Node indexNode = stack.pop();
            System.out.println(indexNode.value);
            if (indexNode.right != null){
                stack.push(indexNode.right);
            }
        }
    }

    /*
     * 借助栈实现迭代的前序遍历，返回的排序的数列
     * */
    private void itePreorderTraverse(Node node) {
        if (node == null){
            return;
        }
        Set<Node> visited = new HashSet<>();
        Stack<Node> stack = new Stack();
        while (node != null) {
            System.out.println(node.value);
            visited.add(node);
            stack.push(node);
            node = node.left;
        }
        while (!stack.isEmpty()){
            Node indexNode = stack.pop();
            if (!visited.contains(indexNode)){
                System.out.println(indexNode.value);
                visited.add(indexNode);
            }
            if (indexNode.right != null){
                stack.push(indexNode.right);
            }
        }
    }

    /*
     * 借助栈实现迭代的后序遍历，返回的排序的数列
     * */
    private void itePostOderTraverse(Node node) {
        if (node == null) {
            return ;
        }
        Stack<Node> stack = new Stack<>();
        Node prev = null;
        while (node != null || !stack.isEmpty()) {
            while (node != null) {
                stack.push(node);
                node = node.left;
            }
            node = stack.pop();
            if (node.right == null || node.right == prev) {
                System.out.println(node.value);
                prev = node;
                node = null;
            } else {
                stack.push(node);
                node = node.right;
            }
        }
    }

    public void bfsTraverse() {
        bfs(this.root);
    }

    /*
    * 借助队列实现广度优先遍历-层序遍历
    * */
    private void bfs(Node node) {
        if (node == null) {
            return;
        }
        Queue<Node> queue = new LinkedList();
        queue.add(node);
        while (!queue.isEmpty()) {
            Node indexNode = queue.poll();
            System.out.println(indexNode.value);
            if (indexNode.left != null) {
                queue.add(indexNode.left);
            }
            if (indexNode.right != null) {
                queue.add(indexNode.right);
            }
        }
    }
}
```
### 平衡二叉搜索树
首先，让我们定义一下平衡二叉搜索树的含义：
+ 平衡二叉树是一棵树，它的高度是 $O(log(n))$，其中n是树内的节点数。
+ 对于平衡树内的每个节点，左子树的高度与右子树的高度相差不得超过 1。

#### 创建平衡二叉搜索树
在创建平衡 BST 时，我们需要牢记高度条件。首先，让我们考虑一下将哪个节点作为根节点最好。

由于我们需要平衡树，所以必须将中间的值作为根节点。然后，我们可以将中间值之前的值添加到树的左边。因此，所有较小的值都将被添加到左侧子树。
同样，我们将中间值之后的值添加到树的右边，这样所有较大的值都会添加到右边的子树中。

不过，获取中间的元素取决于列表的类型。如果列表支持随机存取（如数组），我们可以简单地获取中间元素，将其作为根，然后递归地添加左右子树。我们称这种方法为自顶向下法。

但是，如果我们只有指向列表中第一个元素的指针，那么获取中间元素就比较麻烦了。因此，在这种情况下，我们将使用自下而上的方法。

#### Top-Down 方法
自上而下的方法使用排序数组来创建平衡的 BST。因此，我们可以在恒定时间内访问数组内的任意索引：
<center><img src="pics/quicklatex.com-bd18cef6ee8eb252f700ebf114b4d5fa_l3.svg" alt=""></center>

#### Bottom-Up 方法
自下而上的方法使用链表来构建平衡的 BST。因此，我们会有一个指向列表头部的指针，我们只能在恒定时间内向前移动这个指针。
<center><img src="pics/quicklatex.com-0ab014825b9004386afe016d6b44ec97_l3.svg" alt=""></center>