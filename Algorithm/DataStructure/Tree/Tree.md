<h1 align="center">Tree（树）</h1>

## 1. BST(二叉搜索树)





## 2. 2-3Tree

 

## 3. Red-Black-Tree(红黑树)

1. Red Black Tree inserting: why make nodes red when inserted?
    Yep! Suppose you have a single node in your tree:
    
        5 (black)
    Now insert a new black node into the tree:
    
        5 (black)
         \
          9 (black)
    Now the invariant that every root-null path in the tree has the same number of black nodes is broken, since the path from the root left has one black node and the paths from the root right each have two.    
    
2. Why are newly inserted nodes are always colored red in red black tree insert operation?

    1. Inserting a red node is less likely to violate the red-black rules than inserting a black one. This is because, if the new red node is attached to a black one, no rule is broken. It doesn’t create a situation in which there are two red nodes together which breaks rule of red parent black children, and it doesn’t change the black height(number of black nodes from root to leaf) in any of the paths.
    2. Of course, if you attach a new red node to a red node, the rule of red parent-black children will be violated. However, with any luck this will happen only half the time. Whereas, if you add a new black node, it would always change the black height for its path, violating the black height rule.
    3. Also, it’s easier to fix violations of red parent-black children rule than black height rule.

3. Why don't we add a black node instead of a red node in Red Black tree insertion?

    I think this is due to the rules of red black trees...

    ```
    1. A node is either red or black.
    2. The root is black.
    3. All leaves (NIL) are black.
    4. Both children of every red node are black.
    5. Every simple path from a given node to any of its descendant leaves contains the same number of black nodes.
    ```

    An insertion is added at the bottom of the tree, replacing a leaf (black nil) node with a node with a value and 2 black nil children. Rule 5 stipulates that the count of black nodes must be the same on every path. If you added a black node, you would violate this rule. I will try to illustrate.

    ```
                           B(10)
                 R(5)                    B(15) 
        B(1)              B(6)       B(NIL)   B(NIL)
    B(NIL) B(NIL)    B(NIL) B(NIL)
    ```

    You will notice that on every path there are 3 black nodes. If you were to attempt to insert a new node 16 under 15 as a black node (keep in mind you are adding 2 black nil nodes under the node you are adding), that path would become longer (4). It would be incorrect like this:

    ```
                           B(10)
                 R(5)                     B(15) 
        B(1)              B(6)       B(NIL)     B(16)
    B(NIL) B(NIL)    B(NIL) B(NIL)          B(NIL) B(NIL)
    ```

    To keep all the rules satisfied, you need to insert a red node, and the count of black nodes on every path will remain equal.

    ```
                           B(10)
                 R(5)                     B(15) 
        B(1)              B(6)       B(NIL)     R(16)
    B(NIL) B(NIL)    B(NIL) B(NIL)          B(NIL) B(NIL)
    ```

4. 


参考文章：

* [清晰理解红黑树的演变---红黑的含义](https://www.cnblogs.com/tiancai/p/9072813.html)
* [清晰理解红黑树的演变---红黑的含义](https://blog.csdn.net/wy0123/article/details/79319076)

