---
title: Red black tree
date: 2015-04-15
category: data structure
---

## Intro

Before introducing a red black tree, we need to know what is a binary search tree.
<!-- excerpt -->

### Binary search tree

Binary search tree's features

+ insert cost order lgn
+ find cost order lgn
+ delete cost order lgn

If we want all operations are order lgn time, we need to make sure the tree is balanced. A red black tree is balanced all the time. We can say a red black tree is a self balanced binary search tree.

### Red black tree

What is a red black tree.

1. a node is either red or black
2. the root and leaves(nils) are black
3. every red node has black parent
4. all simple paths from a node x to a descendent leaf of x have same black nodes.

![rb_tree_sample](/images/rbtree/red-black-tree-sample.jpg)

### 2-3-4 tree

if we merge red nodes up to black nodes, we can get the 2-3-4 tree from the red black tree. 

in 2-3-4 tree 

+ every internal node has 2-4 children.
+ every leaf has same depth namely black-height(root) by rule 4.

in 2-3-4 tree

2^h <= leaves <= 4^h

h<=log(n+1)

By rule 3  rbh <= 2h <= 2log(n+1)

### updates:(insert and delete)

- BST operations
- color changes
- restructuring of links via rotations

#### Rotation:

![rb_tree_rotate](/images/rbtree/rbtree-rotate.jpeg)

``` c
    bool RotateLeft(RB_Node* node) {
        if (node == m_nullNode || node->right == m_nullNode) {
            return false;
        }

        RB_Node* right = node->right;
        right->parent = node->parent;
        node->right = right->left;
        if (right->left != m_nullNode) {
            right->left->parent = node;
        }
        if (node->parent == m_nullNode) {// node is root
            m_root = right;
            m_nullNode->left = m_root;
            m_nullNode->right = m_root;
        } else {
            if (node == node->parent->left) {
                node->parent->left = right;
            } else {
                node->parent->right = right;
            }
        }
        node->parent = right;
        right->left = node;
        return true;
    }

    bool RotateRight(RB_Node* node) {
        if (node == m_nullNode || node->left == m_nullNode) {
            return false;
        }

        RB_Node* left = node->left;
        node->left = left->right;
        if (left->right != m_nullNode) {
            left->right->parent = node;
        }
        left->parent = node->parent;
        if (node->parent == m_nullNode) {
            m_root = left;
            m_nullNode->left = m_root;
            m_nullNode->right = m_root;
        } else {
            if (node == node->parent->right) {
                node->parent->right = left;
            } else {
                node->parent->left = left;
            }
        }
        node->parent = left;
        left->right = node;
        return true;
    }
```

#### Insert

rb tree's insert pseudo-code

```
RB-Insert(T, x):
    Tree-Insert(T, x)
    color[x]<-RED
    while x != red[T] and color[x] = RED
    do if p[x] = left[p[p[x]]] // case a
        then y <- right[p[p[x]]] // y is uncle 
            if color[y] = RED
                then <case 1>
            else if x = right[p[x]]
                then <case 2>
            <case 3>
        else  // same as case a, but reversing left <-> right
            .
            .
            .
    color[root[T]] <- black
```

![rb_tree_insert_case]({{filename}}/images/rbtree/insert-case.jpeg)

Tree-Insert function is a normal BST tree insert function and then we change the x color to RED. Finally, we need to re-balance the tree.

``` c
void InsertFixUp(RB_Node* node) {
        while (node->parent->RB_COLOR == RED) {                 // violate 3
            if (node->parent == node->parent->parent->left) {   // left side
                RB_Node* uncle = node->parent->parent->right;
                if (uncle->RB_COLOR == RED) {                   // case 1
                    node->parent->RB_COLOR = BLACK;
                    uncle->RB_COLOR = BLACK;
                    node->parent->parent->RB_COLOR = RED;
                    node = node->parent->parent;                //recursive
                } else {                                        //uncle's color is black
                    if (node == node->parent->right) {          // case 2
                        node = node->parent;
                        RotateLeft(node);
                    }
                    // case 3
                    node->parent->RB_COLOR = BLACK;
                    node->parent->parent->RB_COLOR = RED;
                    RotateRight(node->parent->parent);
                }
            } else { //left side
                RB_Node* uncle = node->parent->parent->left;
                if (uncle->RB_COLOR == RED) {
                    node->parent->RB_COLOR = BLACK;
                    uncle->RB_COLOR = BLACK;
                    node->parent->parent->RB_COLOR = RED;
                    node = node->parent->parent;
                } else {
                    if (node == node->parent->left) {
                        node = node->parent;
                        RotateRight(node);
                    }
                    node->parent->RB_COLOR = BLACK;
                    node->parent->parent->RB_COLOR = RED;
                    RotateLeft(node->parent->parent);
                }
            }
        }
        m_root->RB_COLOR = BLACK;
    }
```

#### Delete

Delete is a much more complicate action in RB-tree than BST.

In BST if we delete a node that has no leaf or has only one leaf, it is a simple action. We just delete the node and connect its child to the node's parent. If the deleted node has two leaves, we need to find the left tree's biggest element and use this element to replace the deleted node, at last, delete the left tree's biggest element, if the biggest element has a child, connect the child to its parent. Reblanced RB-tree.

Pseudo code:

```
RB-DELETE(T, z)
    y = z
    y-origional-color = y.color
    if z.left == T.nil
        x = z.right
        RB-TRANSPLANT(T, z, z.right) // z.right replace z
    else if z.right == T.nil
        x= z.left
        RB-TRANSPLANT(T. z, z.left)
    else y = TREE-MINIMUM(z.right)
        y-origional-color = y.color
        x = y.right
        if y.p == z
            x.p = y
        else RB-TRANSPLANT(T, y, y.right)
            y.right =z.right
            y.right.p = y
        RB-TRANSPLANT(T, z, y)
        y.left = z.left
        y.left.p = y
        y.color = z.color
    if y-origional-color == BLACK
        RB-DELETE_FIXUP(T, x)
```

```
RB-DELETE_FIXUP(T, x)
    while x != T.root and x.color == BLACK
        if x == x.p.left
            w = x.p.right
            if w.color == RED
                w.color = BLACK     //case 1
                x.p.color == RED    //case 1
                LEFT-ROTATE(T,x, p) //case 1
                w = x.p.right       //case 1
            if w.left.color == BLACK and w.right.color == BLACK
                w.color == RED      //case 2
                x = x.p             //case 2
            else
                if w.right.color == BLACK
                    w.left.color = BLACK    //case 3
                    w.color = RED           //case 3
                    RIGHt-ROTATE(T, w)      //case 3
                    w = x.p.right           //case 3
                w.color = x.p.color         //case 4
                x.p.color = BLACK           //case 4
                w.right.color = BLACK       //case 4
                LEFT-ROTATE(T, x, p)        //case 4
                x = T.root                  //case 4
        else // same as then clause with "right" and "left" exchange
            .
            .
    x.color = BLACK

```

![rb_tree_insert_case]({{filename}}/images/rbtree/delete-case.jpeg)

``` c
bool Delete(KEY key) {
        RB_Node* delete_point = find(key);
        if (delete_point == m_nullNode) {
            return false;
        }
        if (delete_point->left != m_nullNode &&
                delete_point->right != m_nullNode) {
            RB_Node* successor = InOrderSuccessor(delete_point);
            delete_point->data = successor->data;
            delete_point->key = successor->key;
            delete_point = successor;
        }
        RB_Node* delete_point_child;
        if (delete_point->right != m_nullNode) {
            delete_point_child = delete_point->right;
        } else if (delete_point->left != m_nullNode) {
            delete_point_child = delete_point->left;
        } else {
            delete_point_child = m_nullNode;
        }
        delete_point_child->parent = delete_point->parent;
        if (delete_point->parent == m_nullNode) {
            m_root = delete_point_child;
            m_nullNode->parent = m_root;
            m_nullNode->left = m_root;
            m_nullNode->right = m_root;
        } else if (delete_point == delete_point->parent->right) {
            delete_point->parent->right = delete_point_child;
        } else {
            delete_point->parent->left = delete_point_child;
        }

        if (delete_point->RB_COLOR == BLACK &&
                !(delete_point_child == m_nullNode && delete_point_child->parent == m_nullNode)) {
            DeleteFixUp(delete_point_child);
        }
        delete delete_point;
        return true;
    }

    void DeleteFixUp(RB_Node* node) {
        while (node != m_root && node->RB_COLOR == BLACK) {
            if (node == node->parent->left) {
                RB_Node* brother = node->parent->right;
                if (brother->RB_COLOR == RED) { // case 1 x's brother is red
                    brother->RB_COLOR = BLACK;
                    node->parent->RB_COLOR = RED;
                    RotateLeft(node->parent);
                } else { // case 2 x's brother is black
                    if (brother->left->RB_COLOR == BLACK
                        && brother->right->RB_COLOR == BLACK) {
                        brother->RB_COLOR = RED;
                        node = node->parent;
                    } else {
                        if (brother->right->RB_COLOR == BLACK) { // case 3
                            brother->left->RB_COLOR = BLACK;
                            brother->RB_COLOR = RED;
                            RotateRight(brother);
                            brother = node->parent->right;
                        }
                        // case 4 brother
                        brother->RB_COLOR = node->parent->RB_COLOR;
                        node->parent->RB_COLOR = BLACK;
                        brother->right->RB_COLOR = BLACK;
                        RotateLeft(node->parent);
                        node = m_root;
                    }
                }
            } else {
                RB_Node* brother = node->parent->left;
                if (brother->RB_COLOR == RED) {
                    brother->RB_COLOR = BLACK;
                    node->parent->RB_COLOR = RED;
                    RotateRight(node->parent);
                } else {
                    if (brother->left->RB_COLOR == BLACK &&
                            brother->right->RB_COLOR == BLACK) { // case 2
                        brother->RB_COLOR = RED;
                        node = node->parent;
                    } else {
                        if (brother->left->RB_COLOR == BLACK) {
                            brother->right->RB_COLOR = BLACK;
                            brother->RB_COLOR = RED;
                            RotateLeft(brother);
                            brother = node->parent->left;
                        }
                        brother->RB_COLOR = node->parent->RB_COLOR;
                        node->parent->RB_COLOR = BLACK;
                        brother->left->RB_COLOR = BLACK;
                        RotateRight(node->parent);
                        node = m_root;
                    }
                }
            }
        }
        m_nullNode->parent = m_root;
        node->RB_COLOR = BLACK;
    }
```
