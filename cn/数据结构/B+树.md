# B+树

他是所有叶子的节点都在同一层也就是说从根节点到任何一个叶子结点他的路径都是一样的。

他的高度是比较低的，因为一个节点可以有多个数据，这样减少了每次查找操作磁盘的访问次数，提升了性能。

B+树的节点大小通常和我们磁盘块大小匹配，这样让我们磁盘读写可以读写或写入一个完整的节点，减少磁盘的IO次数。

~~~c++
          [ 30 ]
         /      \
      [ 10 20 ] [ 40 50 60 ]
~~~



