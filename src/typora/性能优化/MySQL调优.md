# MySQL调优

## MySQL中B+Tree索引原理

B+树索引是B+树在[数据库](http://lib.csdn.net/base/mysql)中的一种实现，是最常见也是数据库中使用最为频繁的一种索引。B+树中的B代表平衡（balance），而不是二叉（binary），因为B+树是从最早的平衡二叉树演化而来的。在讲B+树之前必须先了解二叉查找树、平衡二叉树（AVLTree）和平衡多路查找树（B-Tree），B+树即由这些树逐步优化而来。



**二叉查找树**



二叉树具有以下性质：左子树的键值小于根的键值，右子树的键值大于根的键值。 



如下图所示就是一棵二叉查找树， 

![image-20180911103523036](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911103523036.png)



对该二叉树的节点进行查找发现深度为1的节点的查找次数为1，深度为2的查找次数为2，深度为n的节点的查找次数为n，因此其平均查找次数为 (1+2+2+3+3+3) / 6 = 2.3次

例如查找数字5：从第一个节点6开始查，5比6小，根据二叉查找树的性质，第二次查询从3开始查，5比3大，所以在3这个节点向3的右子树查，得到数字5，总共需要3次查询得到结果。



二叉查找树可以任意地构造，同样是2,3,5,6,7,8这六个数字，也可以按照下图的方式来构造： 

![image-20180911104204255](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911104204255.png)

但是这棵二叉树的查询效率就低了。因此若想二叉树的查询效率尽可能高，需要这棵二叉树是平衡的，从而引出新的定义——平衡二叉树，或称AVL树。
例如查找数字6：需要查询5次。



**平衡二叉树（AVL Tree）**

平衡二叉树（AVL树）在符合二叉查找树的条件下，还满足<u>任何节点的两个子树的高度最大差为1</u>。下面的两张图片，左边是AVL树，它的任何节点的两个子树的高度差<=1；右边的不是AVL树，其根节点的左子树高度为3，而右子树高度为1； 

![image-20180911104616620](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911104616620.png)



如果在AVL树中进行插入或删除节点，可能导致AVL树失去平衡，这种失去平衡的二叉树可以概括为四种姿态：LL（左左）、RR（右右）、LR（左右）、RL（右左）。它们的示意图如下： 

![image-20180911104902317](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911104902317.png)

这四种失去平衡的姿态都有各自的定义： 

LL：LeftLeft，也称“左左”。插入或删除一个节点后，根节点的左孩子（Left Child）的左孩子（Left Child）还有非空节点，导致根节点的左子树高度比右子树高度高2，AVL树失去平衡。



RR：RightRight，也称“右右”。插入或删除一个节点后，根节点的右孩子（Right Child）的右孩子（Right Child）还有非空节点，导致根节点的右子树高度比左子树高度高2，AVL树失去平衡。



LR：LeftRight，也称“左右”。插入或删除一个节点后，根节点的左孩子（Left Child）的右孩子（Right Child）还有非空节点，导致根节点的左子树高度比右子树高度高2，AVL树失去平衡。



RL：RightLeft，也称“右左”。插入或删除一个节点后，根节点的右孩子（Right Child）的左孩子（Left Child）还有非空节点，导致根节点的右子树高度比左子树高度高2，AVL树失去平衡。



AVL树失去平衡之后，可以通过旋转使其恢复平衡。下面分别介绍四种失去平衡的情况下对应的旋转方法。



**失去平衡的二叉查找树通过旋转重新获得平衡：**

LL的旋转。LL失去平衡的情况下，可以通过一次旋转让AVL树恢复平衡。步骤如下：

1. 将根节点的左孩子作为新根节点。
2. 将新根节点的右孩子作为原根节点的左孩子。
3. 将原根节点作为新根节点的右孩子。

LL旋转示意图如下： 

![image-20180911110455272](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911110455272.png)

RR的旋转：RR失去平衡的情况下，旋转方法与LL旋转对称，步骤如下：

1. 将根节点的右孩子作为新根节点。
2. 将新根节点的左孩子作为原根节点的右孩子。
3. 将原根节点作为新根节点的左孩子。

RR旋转示意图如下： 

![image-20180911110515811](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911110515811.png)

LR的旋转：LR失去平衡的情况下，需要进行两次旋转，步骤如下：

1. 围绕根节点的左孩子进行RR旋转。
2. 围绕根节点进行LL旋转。

LR的旋转示意图如下： 

![image-20180911110533321](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911110533321.png)

RL的旋转：RL失去平衡的情况下也需要进行两次旋转，旋转方法与LR旋转对称，步骤如下：

1. 围绕根节点的右孩子进行LL旋转。
2. 围绕根节点进行RR旋转。

RL的旋转示意图如下： 

![image-20180911110604624](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911110604624.png)



**平衡多路查找树（B-Tree）**

B-Tree是为磁盘等外存储设备设计的一种平衡查找树。因此在讲B-Tree之前先了解下磁盘的相关知识。



系统从磁盘读取数据到内存时是以磁盘块（block）为基本单位的，位于同一个磁盘块中的数据会被一次性读取出来，而不是需要什么取什么。



InnoDB存储引擎中有页（Page）的概念，页是其磁盘管理的最小单位。InnoDB存储引擎中默认每个页的大小为16KB，可通过参数innodb_page_size将页的大小设置为4K、8K、16K，在[MySQL](http://lib.csdn.net/base/mysql)中可通过如下命令查看页的大小：

```
mysql> show variables like'innodb_page_size';
```

而系统一个磁盘块的存储空间往往没有这么大，因此InnoDB每次申请磁盘空间时都会是若干地址连续磁盘块来达到页的大小16KB。InnoDB在把磁盘数据读入到磁盘时会以页为基本单位，在查询数据时如果一个页中的每条数据都能有助于定位数据记录的位置，这将会减少磁盘I/O次数，提高查询效率。

B-Tree结构的数据可以让系统高效的找到数据所在的磁盘块。为了描述B-Tree，首先定义一条记录为一个二元组[key, data] ，key为记录的键值，对应表中的主键值，data为一行记录中除主键外的数据。对于不同的记录，key值互不相同。



一棵m阶的B-Tree有如下特性： 



- 每个节点最多有m个孩子。 




- 除了根节点和叶子节点外，其它每个节点至少有Ceil(m/2)个孩子。 




- 若根节点不是叶子节点，则至少有2个孩子 




- 所有叶子节点都在同一层，且不包含其它关键字信息 




-  每个非终端节点包含n个关键字信息（P0,P1,…Pn, k1,…kn） 




- 关键字的个数n满足：ceil(m/2)-1 <= n <= m-1 




- ki(i=1,…n)为关键字，且关键字升序排序。 




- Pi(i=1,…n)为指向子树根节点的指针。P(i-1)指向的子树的所有节点关键字均小于ki，但都大于k(i-1)




B-Tree中的每个节点根据实际情况可以包含大量的关键字信息和分支，如下图所示为一个3阶的B-Tree： 

![image-20180911112748762](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911112748762.png)

每个节点占用一个盘块的磁盘空间，一个节点上有两个升序排序的关键字和三个指向子树根节点的指针，指针存储的是子节点所在磁盘块的地址。两个关键词划分成的三个范围域对应三个指针指向的子树的数据的范围域。以根节点为例，关键字为17和35，P1指针指向的子树的数据范围为小于17，P2指针指向的子树的数据范围为17~35，P3指针指向的子树的数据范围为大于35。



模拟查找关键字29的过程：

1. 根据根节点找到磁盘块1，读入内存。【磁盘I/O操作第1次】
2. 比较关键字29在区间（17,35），找到磁盘块1的指针P2。
3. 根据P2指针找到磁盘块3，读入内存。【磁盘I/O操作第2次】
4. 比较关键字29在区间（26,30），找到磁盘块3的指针P2。
5. 根据P2指针找到磁盘块8，读入内存。【磁盘I/O操作第3次】
6. 在磁盘块8中的关键字列表中找到关键字29。

分析上面过程，发现需要3次磁盘I/O操作，和3次内存查找操作。由于内存中的关键字是一个有序表结构，可以利用二分法查找提高效率。而3次磁盘I/O操作是影响整个B-Tree查找效率的决定因素。B-Tree相对于AVLTree缩减了节点个数，使每次磁盘I/O取到内存的数据都发挥了作用，从而提高了查询效率。



**B+Tree**

B+Tree是在B-Tree基础上的一种优化，使其更适合实现外存储索引结构，InnoDB存储引擎就是用B+Tree实现其索引结构。



从上一节中的B-Tree结构图中可以看到每个节点中不仅包含数据的key值，还有data值。<u>而每一个页的存储空间是有限的，如果data数据较大时将会导致每个节点（即一个页）能存储的key的数量很小，当存储的数据量很大时同样会导致B-Tree的深度较大，增大查询时的磁盘I/O次数，进而影响查询效率</u>（出现B+Tree的原因）。在B+Tree中，所有数据记录节点都是按照键值大小顺序存放在同一层的叶子节点上，而非叶子节点上只存储key值信息，这样可以大大加大每个节点存储的key值数量，降低B+Tree的高度。



B+Tree相对于B-Tree有几点不同：

1. 非叶子节点只存储键值信息。
2. 所有叶子节点之间都有一个链指针。
3. 数据记录都存放在叶子节点中。

将上一节中的B-Tree优化，由于B+Tree的非叶子节点只存储键值信息，假设每个磁盘块能存储4个键值及指针信息，则变成B+Tree后其结构如下图所示： 

![image-20180911112815491](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911112815491.png)

通常在B+Tree上有两个头指针，一个指向根节点，另一个指向关键字最小的叶子节点，而且所有叶子节点（即数据节点）之间是一种链式环结构。因此可以对B+Tree进行两种查找运算：一种是对于主键的范围查找和分页查找，另一种是从根节点开始，进行随机查找。



可能上面例子中只有22条数据记录，看不出B+Tree的优点，下面做一个推算：

InnoDB存储引擎中页的大小为16KB，一般表的主键类型为INT（占用4个字节）或BIGINT（占用8个字节），指针类型也一般为4或8个字节，也就是说一个页（B+Tree中的一个节点）中大概存储16KB/(8B+8B)=1K个键值（因为是估值，为方便计算，这里的K取值为〖10〗^3）。也就是说一个深度为3的B+Tree索引可以维护10^3 * 10^3 * 10^3 = 10亿 条记录。

实际情况中每个节点可能不能填充满，因此在数据库中，B+Tree的高度一般都在2~4层。[mysql](http://lib.csdn.net/base/mysql)的InnoDB存储引擎在设计时是将根节点常驻内存的，也就是说查找某一键值的行记录时最多只需要1~3次磁盘I/O操作。

数据库中的B+Tree索引可以分为聚集索引（clustered index）和辅助索引（secondary index）。上面的B+Tree示例图在数据库中的实现即为聚集索引，聚集索引的B+Tree中的叶子节点存放的是整张表的行记录数据。辅助索引与聚集索引的区别在于辅助索引的叶子节点并不包含行记录的全部数据，而是存储相应行数据的聚集索引键，即主键。当通过辅助索引来查询数据时，InnoDB存储引擎会遍历辅助索引找到主键，然后再通过主键在聚集索引中找到完整的行记录数据。



## SQL执行计划

explain主要是用来获取一个query的执行计划，描述mysql如何执行查询操作、执行顺序，使用到的索引，以及mysql成功返回结果集需要执行的行数。可以帮助我们分析 select 语句,让我们知道查询效率低下的原因,从而改进我们的查询，让查询优化器能够更好的工作。



**explain语法及描述**

![image-20180911164710495](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911164710495.png)

从上面我们可以看到explain的语法是在select语句之前加上explain关键字就行了。然后在执行explain之后会包括以下列信息：

- id：标识符,表示执行顺序.
- select_type：查询类型。
- table：查询的表。
- partitions：使用的哪个分区,需要结合表分区才能看到(since 5.7)
- type：查询类型,例如index索引
- possible_key：可能用到的索引,如果是多个索引以逗号隔开
- key：实际用到的索引,保存的是索引的名称,如果是多个索引以逗号隔开
- key_len：使用到索引的长度
- ref：引用索引对应的表中哪些行
- rows：显示mysql认为执行查询时必须要返回的行数
- filtered：通过过滤条件之后对比总数的百分比
- Extra：额外的信息,using file sort,using where



### **ID**

id表示select标识符，同时表明sql执行顺序，也就是说他是一个序列号.分为三种情况：

- #### id相同 – 顺序执行

![image-20180911164904231](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911164904231.png)

对于以上三表关联查询，我们可以看到id列的值相同，都是1.在这种情况下，它们的执行顺序是按顺序执行，也是就是按列名从上往下执行。

- #### id全不同 – 数字越大，越先执行

![image-20180911165003299](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911165003299.png)

在这里我们可以看到在这个子查询中，ID完全不同。这对这种情况，mysql的执行机制就是数字越大，越先执行。因为要先获取到子查询里的信息做为条件，然后才能查询外面的信息。

- #### id部分相同 – 先大的执行，小的顺序执行

![image-20180911165123559](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911165123559.png)

对于这种id部分相同的情况，其实就是前面2种情况的综合。执行顺序是先按照数字大的先执行，然后数字相同的按照从上往下的顺序执行。



### **select_type – 查询类型**

**select_type包括以下几种类型：**

- simple：表示不需要union操作或者不包含子查询的简单select查询。有连接查询时，外层的查询为simple，且只有一个
- primary：一个需要union操作或者含有子查询的select，位于最外层的单位查询的select_type即为primary。且只有一个
- union：union连接的两个select查询，第一个查询是dervied派生表，除了第一个表外，第二个以后的表select_type都是union
- dependent union：与union一样，出现在union 或union all语句中，但是这个查询要受到外部查询的影响
- union result：包含union的结果集，在union和union all语句中,因为它不需要参与查询，所以id字段为null
- subquery：除了from字句中包含的子查询外，其他地方出现的子查询都可能是subquery
- dependent subquery：与dependent union类似，表示这个subquery的查询要受到外部表查询的影响
- derived：from字句中出现的子查询，也叫做派生表，其他数据库中可能叫做内联视图或嵌套select



- #### **primary/subquery**

```sql
explain select * from student s where s. classid = (select id from classes where classno='2017001');	
```

![image-20180911165321425](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911165321425.png)



- #### **union / union result**

```sql
explain select * from student where id = 1 union select * from student where id = 2;

```

![image-20180911165441251](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911165441251.png)



- #### **dependent union/dependent subquery**

```sql
explain select * from student s where s.classid in (select id from classes where classno='2017001' 
union select id from classes where classno='2017002');
```

![image-20180911165518856](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911165518856.png)



- #### **derived**

```sql
explain select * from (select * from student) s;
```

**在mysql5.5、mysql5.6.x中：**

![image-20180911165554891](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911165554891.png)

**但是在5.7中显示会不一样：**

![image-20180911165607301](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911165607301.png)



- ### **table**

显示的查询表名，如果查询使用了别名，那么这里显示的是别名，如果不涉及对数据表的操作，那么这显示为null，如果显示为尖括号括起来的`<derived N>`就表示这个是临时表，后边的N就是执行计划中的id，表示结果来自于这个查询产生。如果是尖括号括起来的`<union M,N>`，与`<derived N>`类似，也是一个临时表，表示这个结果来自于union查询的id为M,N的结果集。



- ### **partitions – 分区**

partitions这列是建立在你的表是分区表才行。

```sql
explain select * from test_partition where id > 7;
```

![image-20180911165747531](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911165747531.png)

在5.7之前默认不显示分区信息需要手动指明。

explain partitions select * from emp;

![image-20180911165813560](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911165813560.png)



- ### **type – 查询结果类型**

表示按照某种类型来查询，例如按照索引类型查找，按照范围查找，主要是以下几种类型：

- const：表示 表中最多有一个匹配行
- eq_ref：对于每个来自于前面的表的记录，从该表中读取唯一一行
- ref：对于每个来自于前面的表的记录，所有匹配的行从这张表中取出
- ref_or_null：类似于ref，但是可以搜索包含null值的行
- index_merge：出现在使用一张表中的多个索引时，mysql会讲这多个索引合并到一起
- range：按指定的范围来检索，很常见
- index：从索取树中查找
- ALL：全表扫描

**从上面的分类描述中我们可以看出来，查询效率是根据从上往下排列的。**



#### **const**

使用唯一索引或者主键，返回记录一定是1行记录的等值where条件时，通常type是const。其他数据库也叫做唯一索引扫描

```sql
explain select * from student where id = 1;1
```

![image-20180911194711346](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911194711346.png)



#### **eq_ref**

对于每个来自于前面的表的记录，从该表中读取唯一一行。

出现在要连接过个表的查询计划中，驱动表只返回一行数据，且这行数据是第二个表的主键或者唯一索引，且必须为not null，唯一索引和主键是多列时，只有所有的列都用作比较时才会出现eq_ref。

```
explain select * from student s, student ss where s.id = ss.id;1
```



#### **ref**

对于每个来自于前面的表的记录，所有匹配的行从这张表中取出

不像eq_ref那样要求连接顺序，也没有主键和唯一索引的要求，只要使用相等条件检索时就可能出现，常见与辅助索引的等值查找。或者多列主键、唯一索引中，使用第一个列之外的列作为等值查找也会出现，总之，返回数据不唯一的等值查找就可能出现。

```sql
explain select * from student s, student_detail sd where s.id = sd.id;
```

![image-20180911194816648](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911194816648.png)



#### **ref_or_null**

类似于ref，但是可以搜索包含null值的行，实际用的不多。

```
explain select * from student_detail where address = 'xxx' or address is null;1
```

![image-20180911194904060](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911194904060.png)



#### **index_merge**

出现在使用一张表中的多个索引时，mysql会讲这多个索引合并到一起.

表示查询使用了两个以上的索引，最后取交集或者并集，常见and ，or的条件使用了不同的索引，官方排序这个在ref_or_null之后，但是实际上由于要读取所个索引，性能可能大部分时间都不如range.

![image-20180911194949467](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911194949467.png)

#### **range**

索引范围扫描，常见于使用>,<,is null,between ,in ,like等运算符的查询中。

![image-20180911195015794](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911195015794.png)



#### **index**

索引全表扫描，把索引从头到尾扫一遍，常见于使用索引列就可以处理不需要读取数据文件的查询、可以使用索引排序或者分组的查询。

![image-20180911195037060](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911195037060.png)

**ALL**

这个就是全表扫描数据文件，然后再在server层进行过滤返回符合要求的记录。

![image-20180911195055781](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911195055781.png)



### **6、possible_keys & key**

possible_key：查询可能使用到的索引都会在这里列出来。

key：查询真正使用到的索引，select_type为index_merge时，这里可能出现两个以上的索引，其他的select_type这里只会出现一个。

![image-20180911195113678](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911195113678.png)



**key_len**

用于处理查询的索引长度，如果是单列索引，那就整个索引长度算进去，如果是多列索引，那么查询不一定都能使用到所有的列，具体使用到了多少个列的索引，这里就会计算进去，没有使用到的列，这里不会计算进去。

留意下这个列的值，算一下你的多列索引总长度就知道有没有使用到所有的列了。要注意，mysql的ICP特性使用到的索引不会计入其中。另外，key_len只计算where条件用到的索引长度，而排序和分组就算用到了索引，也不会计算到key_len中。

![image-20180911195139162](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180911195139162.png)



**如果key字段值为null，则key_len字段值也为null,而且对于key_len越小越好，当然不能为null.**

### **ref**

如果是使用的常数等值查询，这里会显示const，如果是连接查询，被驱动表的执行计划这里会显示驱动表的关联字段，如果是条件使用了表达式或者函数，或者条件列发生了内部隐式转换，这里可能显示为func

### **rows**

这里是执行计划中估算的扫描行数，不是精确值

### **filtered**

使用explain extended时会出现这个列，5.7之后的版本默认就有这个字段，不需要使用explain extended了。这个字段表示存储引擎返回的数据在server层过滤后，剩下多少满足查询的记录数量的比例，注意是百分比，不是具体记录数。 
![这里写图片描述](https://img-blog.csdn.net/20170328012129618?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjQxMDczMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### **extra**

这个列可以显示的信息非常多，有几十种，常用的有：

- distinct：在select部分使用了distinc关键字
- no tables used：不带from字句的查询或者From dual查询
- 使用not in()形式子查询或not exists运算符的连接查询，这种叫做反连接。即，一般连接查询是先查询内表，再查询外表，反连接就是先查询外表，再查询内表。
- using filesort：排序时无法使用到索引时，就会出现这个。常见于order by和group by语句中
- using index：查询时不需要回表查询，直接通过索引就可以获取查询的数据。
- using join buffer（block nested loop），using join buffer（batched key accss）：5.6.x之后的版本优化关联查询的BNL，BKA特性。主要是减少内表的循环数量以及比较顺序地扫描查询。
- using sort_union，using_union，using intersect，using sort_intersection：
- using intersect：表示使用and的各个索引的条件时，该信息表示是从处理结果获取交集
- using union：表示使用or连接各个使用索引的条件时，该信息表示从处理结果获取并集
- using sort_union和using sort_intersection：与前面两个对应的类似，只是他们是出现在用and和or查询信息量大时，先查询主键，然后进行排序合并后，才能读取记录并返回。
- using temporary：表示使用了临时表存储中间结果。临时表可以是内存临时表和磁盘临时表，执行计划中看不出来，需要查看status变量，used_tmp_table，used_tmp_disk_table才能看出来。
- using where：表示存储引擎返回的记录并不是所有的都满足查询条件，需要在server层进行过滤。查询条件中分为限制条件和检查条件，5.6之前，存储引擎只能根据限制条件扫描数据并返回，然后server层根据检查条件进行过滤再返回真正符合查询的数据。5.6.x之后支持ICP特性，可以把检查条件也下推到存储引擎层，不符合检查条件和限制条件的数据，直接不读取，这样就大大减少了存储引擎扫描的记录数量。extra列显示using index condition
- firstmatch(tb_name)：5.6.x开始引入的优化子查询的新特性之一，常见于where字句含有in()类型的子查询。如果内表的数据量比较大，就可能出现这个
- loosescan(m..n)：5.6.x之后引入的优化子查询的新特性之一，在in()类型的子查询中，子查询返回的可能有重复记录时，就可能出现这个

除了这些之外，还有很多查询数据字典库，执行计划过程中就发现不可能存在结果的一些提示信息。

### **总结**

通过对上面explain中的每个字段的详细讲解。我们不难看出，对查询性能影响最大的几个列是：

- select_type：查询类型
- type：连接使用了何种类型
- rows：查询数据需要查询的行
- key：查询真正使用到的索引
- extra：额外的信息

尽量让自己的SQL用上索引，避免让extra里面出现file sort(文件排序),using temporary(使用临时表)。



## MySQL的聚集索引和非聚集索引

### **非聚集索引**

- 数据无序杂乱无章但是经常使用的列适合建立非聚集索引

### **聚集索引**

- 聚簇索引的数据的物理存放顺序与索引顺序是一致的。所以字段的值只要是有规律的增加或者减少就适合建立聚集索引，例如：日期



使用场景：

![image-20180929214316322](/Users/dyh/Library/Application Support/typora-user-images/image-20180929214316322.png)

## 索引优化

怎么加快查询速度，优化查询效率，主要原则就是应尽量避免全表扫描，应该考虑在where及order by 涉及的列上建立索引。

　　建立索引不是建的越多越好，原则是：

- 第一：一个表的索引不是越多越好，也没有一个具体的数字，根据以往的经验，**一个表的索引最多不能超过6个，**因为索引越多，对update和insert操作也会有性能的影响，涉及到索引的新建和重建操作。

- 第二：建立索引的方法论为：
  - 多数查询经常使用的列；
  - 很少进行修改操作的列；
  - 索引需要建立在数据差异化大的列上

## SQL语句优化

- 条件查询语句调优
  - like语句优化
  - where子句下的or语句优化（优化成union all）
  - where子句下的in或not in（用between、exist、left join优化）
  - where子句中字段使用表达式时优化（表达式不出现在=左边）
  - where子句使用!=或<>操作符优化（使用<>优化！）
  - where子句使用is null或is not null优化（优化成0）

- 功能行语句调优（如：group by、order by等等）

  - 在order by或者group by后的字段上使用有索引的列提高查询的效率 

- 连接查询语句调优

  - 能用inner join连接尽量用inner join连接
  - 子查询性能比外连接查询低，尽量用外连接替换子查询
  - 使用join时，尽量用小的结果驱动大的结果




参考博客：

MySQL的B+Tree索引原理：https://blog.csdn.net/ifollowrivers/article/details/73614549

MySQL索引及查询优化总结：https://cloud.tencent.com/developer/article/1004912

SQL语句调优：https://www.cnblogs.com/parryyang/p/5711537.html

