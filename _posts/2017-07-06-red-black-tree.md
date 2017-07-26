---
layout: post
title: "红黑树"
date: 2017-07-15
excerpt: "数据结构"
tags: [红黑树]
---



{% include toc.html %}
# 红黑树 

#### 1.红黑树的定义 
红黑树是平衡二叉查找树（Balance Search Tree）的一种，它拥有良好的最坏运行时间，可以保证查找、删除、添加节点操作在O(log n)时间内，其中n表示节点数。

>红黑树每个节点都带有颜色属性，红色或者黑色，除了BST的性质以外，还有以下性质：  
1. 节点是红色或黑色。   
2. 根是黑色。   
3. 所有叶子都是黑色（叶子是NIL节点）。   
4. 每个红色节点必须有两个黑色的子节点。（从每个叶子到根的所有路径上不能有两个连续的红色节点。）    
5. 从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点。     

下面是一棵红黑树的例子，例子中为中序遍历降序排列，后面的操作中也按照这个顺序。  
![Alt text](/img/in-post/red_black_tree/demo.png)   
例子中以节点48作为参考节点，类比人类社会，将47节点叫做它的父节点，46叫做它的祖父节点，45叫做它的叔父节点。

#### 2.红黑树操作简单例子

###### step 1 插入节点（44）
因为第一个节点是根节点，根据性质2，节点为黑色。  
![Alt text](/img/in-post/red_black_tree/insert_step_1.png)

###### step 2 插入节点（45）
由于45大于44，所以作为节点44的右边孩子接入并设置为红色。可以看到性质5得到了保持。      
![Alt text](/img/in-post/red_black_tree/insert_step_2.png)

###### step 3 插入节点（46） 
46大于节点45应该作为右孩子接入设置为红色。显然违反了性质4  
![Alt text](/img/in-post/red_black_tree/insert_step_3_1.png)   
以节点45为轴做左旋转,违反了性质2、4、5。    
![Alt text](/img/in-post/red_black_tree/insert_step_3_2.png)  
 将45改为黑色（符合了性质2、4），将节点44改为红色（符合性质5）？？事实上我也可以将46节点涂成黑色？？得到全黑色节点   
![Alt text](/img/in-post/red_black_tree/insert_step_3_3.png)

###### step 4 插入节点（47）
47大于节点46作为右孩子接入设置为红色。违反了性质4。  
![Alt text](/img/in-post/red_black_tree/insert_step_4_1.png)   
这个时候47的父节点46与叔父节点都是红色 ，则祖父节点45颜色下降。但仍然违反性质2，因为47的祖父节点45是根节点。    
![Alt text](/img/in-post/red_black_tree/insert_step_4_2.png)    
将45颜色变为黑色。   
![Alt text](/img/in-post/red_black_tree/insert_step_4_3.png) 

###### step5 删除节点（45）
要删除节点45，首先将其左子树中最大节点44。   
![Alt text](/img/in-post/red_black_tree/insert_step_5_1.png)   
将44节点的值拷贝到45节点处，移除老的44节点。此时违反了性质5。   
![Alt text](/img/in-post/red_black_tree/insert_step_5_2.png)    
以46节点为轴进行坐旋转。但仍然违反性质5。   
![Alt text](/img/in-post/red_black_tree/insert_step_5_3.png)   
将47节点设置为黑色。   
![Alt text](/img/in-post/red_black_tree/insert_step_5_4.png) 

#### 3.红黑树代码实现    
上面简单例子中插入以及删除节点操作并没有完全涵盖所有的情况。下面来看一看红黑树java代码实现。根据具体每一类情况，形成与之对应的代码操作。红黑树插入节点都按照红色插入。（维基百科解释：如果设为黑色，就会导致根到叶子的路径上有一条路上，多一个额外的黑节点，这个是很难调整的。但是设为红色节点后，可能会导致出现两个连续红色节点的冲突，那么可以通过颜色调换和树旋转来调整。）

##### 3.1 红黑树插入操作   
##### <font color="blue">
情形1:之前是棵空树，新插入的节点是树根。根据性质2，根节点必须是黑色。
</font>   
![Alt text](/img/in-post/red_black_tree/insert_condition_1.png) （图1）   
{% highlight java %}   
    /**
     * 情形1:如果插入节点是根节点，则设置为黑色
     *
     * @param node
     */
    private void insertFixUpCase1(Node node) {
        if (node.parent == null) {
       
            //情形1:如果插入节点是根节点，则设置为黑色
            root = node;
            node.color = Color.BLACK;
        } else {
            insertFixUpCase2(node);
        }

    }
{% endhighlight %}   
   
##### <font color="blue">情形2:插入的节点的父节点是黑色的，这种情况下直接插入红色节点。（如图：插入节点46）
</font>
![Alt text](/img/in-post/red_black_tree/insert_condition_2_1.png)（图2.1）   
![Alt text](/img/in-post/red_black_tree/insert_condition_2_2.png)（图2.2）   
{% highlight java %} 
   private void insertFixUpCase2(Node node) {
        if (node.parent.color == Color.BLACK) {
            //父节点是黑色
            //情形2：如果父节点是黑色的，所有性质都没有受到影响
            //不需要做什么
        } else {
            insertFixUpCase3(node);
        }

    } 
{% endhighlight %}    
##### <font color="blue">情形3: 
插入的节点父节点以及叔父节点都是红色。从图3.1为插入前状态，插入节点42后成为节点43的左孩子，这时性质4遭到破坏，因为存在连续红色节点（图3.2）。要满足性质4需要将祖父节点（44）与父节点、叔父节点的颜色进行交换，如图3.3。性质2此时无法满足因为根节点44是红色。其实给出图片的例子中44是一个根节点，很多情况中44并不是根节点。此时将44按照新插入的节点从情形1开始处理，那么应该涂成黑色，如图3.4   
</font>
![Alt text](/img/in-post/red_black_tree/insert_condition_3_1.png)（图3.1）   
![Alt text](/img/in-post/red_black_tree/insert_condition_3_2.png)（图3.2）   
![Alt text](/img/in-post/red_black_tree/insert_condition_3_3.png)（图3.3）  
![Alt text](/img/in-post/red_black_tree/insert_condition_3_4.png)（图3.4）  
{% highlight java %}  

    private void insertFixUpCase3(Node node) {

        //父节点是红色

        Node uncleNode = getUncleNode(node);

        if (uncleNode != null) {

            if (uncleNode.color == Color.RED) {
                //叔父节点是红色
                //情形3:父节点和叔父节点都是红色。这种情况下，我们需要将祖父节点的颜色下沉一层，即祖父节点变成红色，父节点和叔父变成黑色。

                uncleNode.color = Color.BLACK;
                node.parent.color = Color.BLACK;

                getGrandParentNode(node).color = Color.RED;

                //调整
                insertFixUpCase1(getGrandParentNode(node));

            } else {
                //叔父节点是黑色
                //情形4：父节点是红色，叔父节点是黑色，祖父节点肯定是黑色的，要不然就违反了红黑树不能连续红色节点的性质
                insertFixUpCase4(node);
            }
        }
   } 
{% endhighlight %}
   
##### <font color="blue">情形4:
节点是父节点的右孩子（如图4.3中节点47），其父节点（44）是其祖父节点（50）的左孩子，其叔父节点是黑色（节点60）。这种情况的对称情况也属于同一情况。下面来看一下这种情形是怎么出现的，图4.1是初始状态，插入节点45，红色，图4.1到4.3这个过程实际上是情形3的处理过程。从4.3到4.4才是情形4的场景，这种情况下以新插入节点（这里是47）作为旋转轴做一次左旋转，对称的情况做右旋转。此时红黑树的性质5仍然没有满足，下面交给情形5处理。
</font>   
![Alt text](/img/in-post/red_black_tree/insert_condition_4_1.png)（图4.1）  
![Alt text](/img/in-post/red_black_tree/insert_condition_4_2.png)（图4.2）
![Alt text](/img/in-post/red_black_tree/insert_condition_4_3.png)（图4.3）
![Alt text](/img/in-post/red_black_tree/insert_condition_4_4.png)（图4.4）  
{% highlight java %}  
private void insertFixUpCase4(Node node) {

       if (node.parent.leftChild == node && getGrandParentNode(node).rightChild == node.parent) {

            rotateRight(node);

        } else if (node.parent.rightChild == node && getGrandParentNode(node).leftChild == node.parent) {
            rotateLeft(node);
        }


    }
{% endhighlight %}


##### <font color="blue">情形5:节点是父节点的左孩子，父节点是祖父节点的左孩子，叔父节点是黑色。我们直接看图5.1（实际上和图4.4是一张图，想知道来龙去脉可以看这里）。此时性质5被破坏，我们先以节点47为轴做右旋转得到图5.2，再将原先父节点（节点47）与原先祖父节点（节点50）颜色进行交换，完成，如图5.3。
</font>   
![Alt text](/img/in-post/red_black_tree/insert_condition_5_1.png)（图5.1）  
![Alt text](/img/in-post/red_black_tree/insert_condition_5_2.png)（图5.2）
![Alt text](/img/in-post/red_black_tree/insert_condition_5_3.png)（图5.3）


#### 4.红黑树具体应用 

#### 5.希望解决的问题
1.红黑树叶子结点为什么是nil的，有什么用处，在应用中有什么意义？   
2.删除操作中一再强调的叶子结点是nil结点，非nil的叶子结点？

#### 6.参考
https://www.cs.usfca.edu/~galles/visualization/RedBlack.html   
https://github.com/julycoding/The-Art-Of-Programming-By-July/blob/master/ebook/zh/03.01.md
https://zh.wikipedia.org/wiki/%E7%BA%A2%E9%BB%91%E6%A0%91
https://zh.wikipedia.org/wiki/%E6%A0%91%E6%97%8B%E8%BD%AC
https://en.wikipedia.org/wiki/Tree_rotation