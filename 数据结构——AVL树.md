title: 数据结构——AVL树
permalink: Data_Structure_AVLTree
toc: true
mathjax: false
fancybox: false
tags: [算法]
categories: 编程
date: 2017-02-11 17:00:18

---

AVL树是一种特殊的二叉查找树，其特征在于：对所有节点来说，其左子树和右子树间的高度差小于等于1。本文简要总结下AVL树的几种基本操作。

<!--more-->

## 节点结构体定义

``` C
typedef struct Node_s {
	int element;
	struct Node_s * left, * right;
	int height;
} Node;
```

为了突出说明核心问题，节点数据类型使用最简单的`int`表示；`height`为树的高度，**叶子节点高度为0，每向上一层加1，即每个节点的深度为其左右子树最大深度加1**。可使用以下几个宏定义来计算及获取深度：

``` C
#define Height(T) (T == NULL ? -1 : T->height)
#define MAX(a, b) (a > b ? a : b)
#define CalHeight(T) (T->height = MAX(Height(T->left), Height(T->right)) + 1)
```

## AVL树的旋转

旋转操作是AVL树特有的操作，也是学习AVL树的核心,旋转的目的在于解决插入、删除等操作造成的AVL树不平衡问题。AVL树的不平衡一共只有4种情况（以插入为例说明）：

- LL：在左子树左节点进行插入
- LR：在左子树右节点进行插入
- RR：在右子树右节点进行插入
- RL：在右子树左节点进行插入

示意图：

![](http://gmf.shengnengjin.cn/281624280475098.jpg)
![](http://gmf.shengnengjin.cn/281625317193879.jpg)

关于旋转的详细分析可参考[第一篇参考资料](http://www.cnblogs.com/skywang12345/p/3576969.html)，此处仅给出实现代码及简要思路。

### LL及RR单旋转

二者是镜像操作，实现方法比较简单。

``` C
Node * RotateLL (Node * T) {
	Node * t = T->left;

	T->left = t->right;
	t->right = T;

	CalHeight(T);
	CalHeight(t);

	return t;
}
```

``` C
Node * RotateRR (Node * T) {
	Node * t = T->right;

	T->right = t->left;
	t->left = T;

	CalHeight(T);
	CalHeight(t);

	return t;
}
```

调整完节点关系后，需要重新计算一下节点的高度。

### LR及RL双旋转

二者也是镜像操作，可以视为两次单旋转的结合。

``` C
Node * RotateLR(Node * T) {
	T->left = RotateRR(T->left);
	return RotateLL(T);
}
```

``` C
Node * RotateRL(Node * T) {
	T->right = RotateLL(T->right);
	return RotateRR(T);
}
```

因为单旋转操作已经正确的调整了节点高度，双旋转中无需再调整节点高度。

## 常用操作

### 插入

一般使用递归形式，递归函数返回时，检查当前节点是否平衡，不平衡则执行旋转操作。

``` C
Node * Insert(Node * T, int val) {
	/* 新插入的元素必定为叶子节点 */
	if (T == NULL) {
		T = (Node *)malloc(sizeof(Node));
		T->element = val;
		T->left = T->right = NULL;
	} 
	/* 在左子树插入 */
	else if (val < T->element) {
		T->left = Insert(T->left, val);
		if (Height(T->left) - Height(T->right) == 2) {
			if (val < T->left->element)
				T = RotateLL(T);
			else
				T = RotateLR(T);
		}
	} 
	/* 在右子树插入 */
	else if (val > T->element) {
		T->right = Insert(T->right, val);
		if (Height(T->right) - Height(T->left) == 2) {
			if (val > T->right->element)
				T = RotateRR(T);
			else
				T = RotateRL(T);
		}
	}
	/* 若元素已存在，不执行任何操作 */

	/* 递归函数返回前调整节点高度 */
	/* 这保证了每一层递归函数返回的节点高度都是正确的 */
	/* 进而保证了整棵树的节点高度正确 */
	CalHeight(T);
	return T;
}
```

### 查找元素

根据二叉查找树的基本性质，可以很容易的写出查找最大元素、最小元素、任意元素的代码。

``` C
Node * FindMax(Node * T) {
	while(T->right != NULL) {
		T = T->right;
	}
	return T;
}
```

``` C
Node * FindMin(Node * T) {
	while(T->left != NULL) {
		T = T->left;
	}
	return T;
}
```

``` C
Node * Find(Node * T, int val) {
	while (T != NULL) {
		if (T->element == val)
			break;
		else if (val < T->element)
			T = T->left;
		else 
			T = T->right;
	}
	return T;
}
```

### 删除

删除是最复杂的操作，如果删除操作不多的话，可以考虑使用懒惰删除的策略，即增加一个标志位，表明当前节点是否被删除了。如果需要真实的删除元素，使用以下方法进行：

``` C
Node * Delete(Node * T, int val) {
	/* 待删除元素在左子树中 */
	if (val < T->element) {
		T->left = Delete(T->left, val);
		if (Height(T->right) - Height(T->left) == 2) {
			if (Height(T->right->left) < Height(T->right->right))
				T = RotateRR(T);
			else
				T = RotateRL(T);
		}
	} 
	/* 待删除元素在右子树中 */
	else if (val > T->element) {
		T->right = Delete(T->right, val);
		if (Height(T->left) - Height(T->right) == 2) {
			if (Height(T->left->right) < Height(T->left->left))
				T = RotateLL(T);
			else
				T = RotateLR(T);
		}
	} 
	/* 删除当前节点 */
	else {
		/* 当前节点有两个儿子 */
		/* 选择高度较大那一边进行删除，以此避免AVL树不平衡 */
		if (T->left && T->right) {
			/* 选择左树的话，用左树中最大节点代替当前节点，并删除最大节点原位置 */
			if (Height(T->left) > Height(T->right)) {
				Node * tmax = FindMax(T->left);
				T->element = tmax->element;
				T->left = Delete(T->left, tmax->element);
			} 
			/* 选择右树的话，用右树中最小节点代替当前节点，并删除最小节点原位置 */
			else {
				Node * tmin = FindMin(T->right);
				T->element = tmin->element;
				T->right = Delete(T->right, tmin->element);
			}
		} 
		/* 当前节点是叶子节点或只有一个儿子，直接删除 */
		else {
			Node * tmp = T;
			T = T->left ? T->left : T->right;
			free(tmp);
		}
	}

	/* 递归返回非空节点时，需要重新计算其高度 */
	if (T) 
		CalHeight(T);
	return T;
}
```

基本策略和插入一样，依然是递归的进行删除，若待删除节点有两个儿子时，使用树的删除操作中的一般方法，即**选择较高一侧子树中最大或最小元素代替当前元素，之后再删除那个最大或最小元素**。最大或最小元素一定是叶子元素，这样之后的删除操作就会很简单，且这样的替代删除策略不会导致树的不平衡。

### 遍历

同样有前序、中序、后序及层序四种遍历策略，就是树的通用遍历策略，可参考[二叉树的遍历算法](http://gaomf.cn/2017/02/12/BinaryTree_Traversal/)。

----------

> 参考资料：
> [AVL树(一)之 图文解析 和 C语言的实现](http://www.cnblogs.com/skywang12345/p/3576969.html)