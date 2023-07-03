# SPLAY伸展树

SplayTree伸展树用于毕业设计之中，它本身是一种常见的数据结构算法。
<!--more-->

输入法候选框的基本原理就是Spaly Tree，每次要访问一个节点，都会将该节点旋到`root`，同时保持排序树的性质不变。长久下来，经常访问的数据就会更靠近`root`，就会更快的被访问，像CACHE一样，频繁的数据最快。

Splay Tree 的基本操作都基于伸展 (Splay) 操作,通过一系列的树旋转(Zig 和 Zag
操作),在把指定顶点移动到根顶点的同时,通过调整把树的结构平衡化。每次访问完一个顶点,就将该顶点进行 Splay 操作。越是经常被访问的顶点,其深度就越小,越容
易被寻找到,达到了缓存加速的效果。Splay Tree 的插入顶点过程也类似于一般的二叉查找树。

为了避免旋转的过程中,树退化成链的情况发生,Splay 采用了双旋的方法。考虑
旋转顶点 v 的父顶点和祖父顶点的情况,保证在旋转后不会产生退化成链的情况,尽可能地在把顶点 v 上浮的同时,平衡左右子树。

```C++
// C++ code for the above approach:
#include <bits/stdc++.h>
using namespace std;

struct node {
	int key;
	node *left, *right;
};

node* newNode(int key)
{
	node* temp = new node;
	temp->key = key;
	temp->left = temp->right = NULL;
	return temp;
}

node* rightRotate(node* x)
{
	node* y = x->left;
	x->left = y->right;
	y->right = x;
	return y;
}

node* leftRotate(node* x)
{
	node* y = x->right;
	x->right = y->left;
	y->left = x;
	return y;
}

node* splay(node* root, int key)
{
	if (root == NULL || root->key == key)
		return root;

	if (root->key > key) {
		if (root->left == NULL)
			return root;
		if (root->left->key > key) {
			root->left->left = splay(root->left->left, key);
			root = rightRotate(root);
		}
		else if (root->left->key < key) {
			root->left->right
				= splay(root->left->right, key);
			if (root->left->right != NULL)
				root->left = leftRotate(root->left);
		}
		return (root->left == NULL) ? root
									: rightRotate(root);
	}
	else {
		if (root->right == NULL)
			return root;
		if (root->right->key > key) {
			root->right->left
				= splay(root->right->left, key);
			if (root->right->left != NULL)
				root->right = rightRotate(root->right);
		}
		else if (root->right->key < key) {
			root->right->right
				= splay(root->right->right, key);
			root = leftRotate(root);
		}
		return (root->right == NULL) ? root
									: leftRotate(root);
	}
}

node* insert(node* root, int key)
{
	if (root == NULL)
		return newNode(key);

	root = splay(root, key);

	if (root->key == key)
		return root;

	node* temp = newNode(key);
	if (root->key > key) {
		temp->right = root;
		temp->left = root->left;
		root->left = NULL;
	}
	else {
		temp->left = root;
		temp->right = root->right;
		root->right = NULL;
	}
	return temp;
}

// Drivers code
int main()
{
	node* root = NULL;
	root = insert(root, 100);
	root = insert(root, 50);
	root = insert(root, 200);
	root = insert(root, 40);
	root = insert(root, 60);
	cout << "Preorder traversal of the modified Splay "
			"tree: \n";
	return 0;
}

```

---

> 作者: map[avatar:<nil> email:<nil> link:<nil> name:<nil>]  
> URL: https://hongwei7.online/splay%E4%BC%B8%E5%B1%95%E6%A0%91/  

