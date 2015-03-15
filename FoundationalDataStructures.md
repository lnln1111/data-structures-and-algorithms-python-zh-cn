　　本书我们讨论抽象数据类型(ADT)，包括栈，队列，双端队列，有序表，哈希表和分散表，树，优先队列，集合和图。在大多数情况下，我们可以选择用一个数组或者用某种链式数据结构。
　　由于这些数据结构是多数ADT的基础，我们把数组和链表称为基本数据结构。要清楚我们不把数组或链表看作ADT，但是更像是ADT的可选的实现
　　在这一章我们首先讨论数组。我们回顾Python对数组的支持并且呈现一个大小可变的拥有任意下标范围的数组实现，也讨论多维数组和矩阵。然后，我们讨论许多链接表的实现和有单个链接的列表的实现细节，链表。熟悉数组和链表很重要，因为它们在本书余下部分将会被广泛应用。

4.1.1 扩展Ｐython列表　－－数组
　　虽然python确实原生支持列表，但这样的支持也有弱点：比如，列表第一个（即最左）的索引总是０（或者说-n），并且最右的索引总是n-1（或者说-1)。然而，在某些特定的应用里，把列表用非０数字索引会更好。另一个事实，列表充许负指数，这也成为一个争论点。在很多应用中，只需要非负指数，并且试图用负指数是程序设计错误的表现。因为python充许负指数，这样的错误在运行时是不能被检测出来的，而且可能导致不正确的编程实现。
　　一种处理这些缺陷的方式是定义一个实现了所需功能的新类。我们可以用两个实例属性_data和_baseIndex定义一个数组类来完成。前者是一个python列表，后者是一个记录数组指数下限的普通整数。

4.1.2 init方法
　　下面这个程序是Array类的init方法的代码。init方法有三个参数，self, length和baseIndex。length参数设定预期的数组长度，baseIndex参数设定数组下限。init方法创建一个有所需长度的列表并且设定_baseIndex。注意第一项是０并且默认的数组长度也是０._```python

class Array(object):
def __init__(self, length=0, baseIndex=0):
assert length >= 0
self._data = [None for i in xrango(length)]
self._baseIndex = baseIndex
#......
```
　　在python中，当一个列表被分配，会发生两件事。首先，内存被分配给这个列表对象和它的元素，然后，列表的每个元素用适当的默认值来初始化（本例中所有的列表元素指向None对象）。
　　现在，我们应该清楚第一步花费固定数量的时间。因为有n=length个元素要被初始化，第二步花费O(n)的时间。因此，Array类的init的时间复杂度是O(n)。

4.1.3 copy方法
　　下面这个程序定义Array类的copy方法。这个方法可以复制一个给定的数组，创建一个浅拷贝。浅拷贝创建对Array的复制但是不复制包含在数组里的对象。
```python

class Array(object):
def __copy__(self):
result = Array(len(self._data))
for i, datum in enumerate(self._data):
result._data[i] = datum
result._baseIndex = self._baseIndex
return result
#...
```
　　copy方法用于和python的copy模块相结合，比如：
```python

from copy import copy


a = Array(5)
b = copy(a)
```
　　copy模块的copy函数目的是创建对一个对象的浅拷贝，如果对象提供一个copy方法，copy函数就调用该方法来创建这次拷贝。
　　上文的程序显示了一个简单的copy方法的实现。为了测定它的运行时间，我们需要认真考虑这个方法的执行。
　　首先，这创建了一个新的实例。就如前面讨论的，这次运行最坏的结果是O(n)，这里的n是这个新数组的长度。
　　然后，有一个循环逐个复制输入的数组的元素到新分配的数组。显然这个操作花费O(n)的时间来执行。最后，复制_baseIndex的实例属性花费了O(1)的时间。总之，__copy__的时间复杂度是T(n) = O(n)，n是被复制的数组的长度。
　　
4.1.4　__getitem__和__setitem__方法
　　python列表的元素通过中括号［］来访问，比如：
　　a[2](2.md) = a[3](3.md)
　　为了能够用相同的语法访问Array对象的元素，我们定义了如下的__getitem__和__setitem__方法：_```python

class Array(object):
def getOffset(self, index):
offset = index - self._baseIndex
if offset < 0 or offset >= len(self._data):
raise IndexError
return offset

def __getitem__(self, index):
return self._data[self.getOffset(index)]
def __setitem__(self, index, value):
self._data[self.getOffset(index)] = value
#...
```
　　getitem和setitem都调用getOffset方法通过从给定的索引减去_baseIndex实例属性来转换它。这样的方法可以使任意的下标范围都被支持。因为减法的总开销是常数，所以__getitem__和__setitem方法的时间复杂度都是O(1)。_

4.1.5　数组的属性
　　下面这个程序定义了Array的两个属性：data和baseIndex。这两个属性提供检查和修改数组对象内容的方法。
　　显然，两个baseIndex属性fget和fset访问器的运行时间都是一个常数。相似地，data属性fget访问器的运行时间也是一个常数。
```python

class Array(object):

def getData(self):
return self._data

data = property(
fget = lambda self: self.getData())

def getBaseIndex(self):
return self._baseIndex

def setBaseIndex(self, baseIndex):
self._baseIndex = baseIndex

baseIndex = property(
fget = lambda self: self.getBaseIndex(),
fset = lambda self, value: self.setBaseIndex(value))
#...
```
4.1.6 调整一个数组的大小
　　下面这个程序定义Array类的长度属性。fget访问器简单地调用len方法并返回数组的长度。fset访问器调用提供在运行时改变数组长度的setLength方法。这个方法可以被用于增加和减少数组的长度。
```python

class Array(object):

def __len__(self):
return len(self._data)

def setLength(self, value):
if len(self._data) != value:
newData = [None for i in xrange(value)]
m = min(len(self._data), value)
for i in xrange(m):
newData[i] = self._data[i]
self._data = newData

length = property(
fget = lambda self: self.__len__(),
fset = lambda self, value: self.setLength(value))
#...
```
4.2 多维数组
　　一个多维数组是一系列用n作下标来表示的条目。比如，一个二维数组的元素(i,j)是用x[i,j]来访问的。
　　python不提供对多维数组的内建支持。在这个章节，我们将尝试一个多维的数组类的实现，多维数组，这个是基于上一章讨论的一维数组的。

4.2.1 数组下标计算
　　计算机的内存本质上是一个一维数组－－内存地址就是这个数组的下标。因此，一个很自然实现一个多维数组的方法就是在一个一维数组里存储它的元素。为了这样做，我们需要从用来访问多维数组的元素的以n个下标的表达式映射到单一下标的一维数组表达式，比如，假设我们想实现一个2\*3的整数类型的数组，可以这样做：
　　b = Array(6)
　　然后我们需要确定b的哪个元素，即b[k](k.md)，将用形如a[i, j]的引用来访问，也就是说，我们需要k = f(i,j)这样的映射函数。
　　映射函数决定数组元素在内存中的存储方式，最常见的表示一个数组的方式是按行排序（即行优先），即通常所知的字典顺序。比如，下图显示了2\*3的二维数组按行排序布局：

　　在按行排序布局中，增加最快的是最右下标表示（列的索引），这样的结果是，矩阵每行中的元素最终储存在连续的内存位置。

4.2.2　实现
　　这个章节我们举例说明用一维数组实现一个多维数组。做法是定义一个名为MultiDimensionalArray的跟前面定义的Array相似的类。
　　总共有三个实例属性来实现MultiDimensionalArray类。第一个，_dimensions,是一个长度为n的数组，n是数组的维度，dimension[i](i.md)是第i个维度的大小。第二个实例属性,_factors，也是一个长度为n的数组。第三个实例属性，_data，是用于按行排序控制多维数组元素的一维数组。_

4.2.3 类MultiDimensionalArray的init方法
　　MuMultiDimensionalArray类的init方法在下面这个程序中有定义。dimensions参数是一个表现数组的维度的元组。比如，为了创建一个3\*5\*7的三维数组，我们可以这样创建一个
```python

MultiDimensionalArray：
　　a = MultiDimensionalArray(3,5,7)
#coding:utf-8
__author__ = 'bob'

class MultiDimensionalArray(object):
def __init__(self, dimensions):
self._dimensions = Array(len(dimensions))
self._factors = Array(len(dimensions))
product = 1
i = len(dimensions) - 1
while i >= 0:
self._dimensions[i] = dimensions[i]
self._factors = product
product *= self._dimensions[i]
i -= 1
self._data = Array(product)

#...
```
　　init方法复制数组的维度到_dimensions数组，然后它会运算_factors数组。这些运算花费O(n)，n是维度数。init方法然后分配一个长度为m的数组，表示为：

　　init方法的时间复杂度最糟糕的结果是O(m+n)

4.2.4 　 MultiDimensionalArray类的getitem和setitem方法
　　多维数组的元素可以用getitem和setitem方法来访问。比如，你可以这样访问一个三维数组的第(i, j, k)个元素：
　　value = a[i, j, k]
　　这样用元组(i, j, k)调用getitem方法作为索引表达。相似地，你可以这样修饰第(i, j, k)个元素：
　　a[i, j, k] = value
　　这样用元组(i, j, k)调用setItem方法作为索引表达。
　　下面这个程序显示了getitem和setitem方法怎样用getOffset方法实现。
```python

#coding:utf-8
__author__ = 'bob'

class MultiDimensionalArray(object):
def getOffset(self, indices):
if len(indices) != len(self._dimensions):
raise IndexError
offset = 0
for i, dim in enumerate(self._dimensions):
if indices[i] < 0 or indices[i] >= dim:
raise IndexError
offset += self._factors[i] * indices[i]
return offset

def __getitem__(self, indices):
return self._data[self.getOffset(indices)]

def __setitem__(self, indices, value):
self._data[self.getOffset(indices)] = value

#...
```
4.2.5　矩阵
　　二维的浮点型数组在很多不同的科学计算中都有应用。这些数组通常称为矩阵。数学家们已经研究了很多年矩阵的属性并且已经开发了一个几乎全能的操作矩阵的方法。在这部分我们研究二维的矩阵和查看简单的矩阵乘法运算的实现。
　　二维矩阵的维度称作行和列。下面这个程序定义了一个提供两个属性（numberOfRows和numberOfColumns）的Matrix类。
```python

#coding:utf-8
__author__ = 'bob'

class Matrix(object):
def __init__(self, numberOfRows, numberOfColumns):
assert numberOfRows >= 0
assert numberOfColumns >= 0
super(Matrix, self).__init__()
self._numberOfRows = numberOfRows
self._numberOfColumns = numberOfColumns

def getNumberOfRows(self):
return self._numberOfRows

numberOfRows = property(
fget= lambda self: self.getNumberOfRows())

def getNumberOfColumns(self):
return self._numberOfColumns

numberOfColumns = property(
fget= lambda self: self.getNumberOfColumns())

#...
```
　　Matrix类有两个普通整数型的实例属性，_numberOfRows和_numberOfColumns,它们纪录矩阵维度。

4.2.6　密集矩阵
　　最简单的实现一个矩阵的方法就是如下这个程序所示，用有两个维度的多维数组。DenseMatrix类扩展了前面一节讨论的Matrix类。ＤenseMatrix类增加了一个名为\_array的实例属性，这是一个多维数组。DenseMatrix类的init方法有两个参数，m=rows和n=cols，并且构建相应的m\*n多维数组。明显地，init方法的时间复杂度是O(mn)。
```python

#coding:utf-8
__author__ = 'bob'

class DenseMatrix(Matrix):
def __init__(self, rows, cols):
super(DenseMatrix, self).__init__(rows, cols)
self._array = MultiDimensionalArray(rows, cols)

def __getitem__(self, (i, j)):
return self._array[i, j]

def __setitem__(self, (i, j), value):
self._array[i, j] = value

#...
```
　　这个程序也定义了类DenseMatrix的getitem和setitem方法。这些方法都有一个序偶，(i, j)，作为索引表达式并且用这个序偶作为多维数组的索引。因为维度固定为２，所以这些方法的时间复杂度都是O(1)。

4.2.7　标准矩阵乘法
　　考虑一个m\*n的矩阵Ａ和n\*p的矩阵Ｂ，乘积Ｃ＝AＢ是一个m＊p的矩阵。生成矩阵的元素由下式给出：
（好吧，我打不出求和的符号，只好截图了。。。）
　　因此，为了计算生成矩阵，Ｃ，我们需要计算mp总和，这是每个n乘积项的和。下面这个算法给出了计算矩阵积的一个算法
```python

#coding:utf-8
__author__ = 'bob'

class DenseMatrix(Matrix):
def __mul__(self, mat):
assert self._numberOfColumns == mat.numberOfRows
result = DenseMatrix(
self.numberOfRows, mat.numberOfColumns)
for i in xrange(self.numberOfRows):
for j in xrange(mat.numberOfColumns):
sum = 0
for k in xrange(self.numberOfColumns):
sum += self[i, k] * mat[k, j]
result[i, j] = sum
return result
#...
```
　　这个算法首先检查要相乘的矩阵是否有一致的维度，即，第一个矩阵的列数必须和第二个矩阵的行数相等。这项检查在最坏的情况下会花费O(1)的时间。
　　然后，构建了一个结果将成形的矩阵（第６行），这个行为的时间复杂度是O(mp)。对于i和j的每个值，最深层的循环（１０－１１行）做了n次迭代。每次迭代花费一个常数数量的时间。
　　中间的循环体（９－１２行）为i和j的每个值花费O(n)的时间。中间循环做了p次迭代，i的每个值运行时间都分配了O(np)。然后，外围循环（７－１２行）做了m次迭代，它的总共运行时间是O(mnp)。最后，结果矩阵在１３行返回。这需要花费一个常数数量的时间。
　　总之，４－５行是O(1)；第６行是O(mp)；７－１２行是O(mnp)；第13行是O(1)。因此，标准的矩阵乘法运算的时间复杂度是O(mnp)。

4.3　单链表
　　单链表是所有有连接指令的数据结构中最基础的。一个单链表仅仅是一串连续的对象，每个对象指向它的后继结点。除去这个明显的简单性不说，单链表有大量的实现变种。下图给出了几种最常见的单链表变种。

　　上图展示了一些基本的单链表。表中每个结点都指向它的后继结点并且尾结点指向None。图（a）中的可变的，带标签的head结点用来记录链表。
　　当我们想在两端都增加结点时单链表显得低效。虽然在单链表的开头增加结点很容易，但是要在另一端（尾端）增加结点需要找出最后的结点。如果使用基本单链表，整个链表需要被反转，只为找到它的尾结点。
　　图（b）显示了一种更高效的在链表的末尾增加结点的方法。这种方法使用一个第二变量，tail，它指向链表最后一个结点。当然，这次效率的提升以增加tail变量的存储空间为代价。
　　图（c）中带标签的单链表图解说明了两种常见的编程技巧。在链表的开头增加了一个额外的结点，名为sentinel（岗哨）。这个结点从不用于控制数据并且它总是存在。用sentinel的主要好处是它可以简化特定的编程操作。比如，因为总是有一个“岗哨”，我们就不需要去修改head变量。当然，像图（c）中那样使用sentinel的缺点在于需要额外的空间，并且sentinel变量需要在链表初始化的时候就创建。
　　图（c）中的链表也是一个循环链表，尾结点指向sentinel，而不是用一个引用None来划分链表的末尾。这个编程技巧的好处是在链表的头部，尾部，和任意位置的插入操作都是一样的。因此，这样做使得在链表的头部和尾部插入结点相对简单。这个变种方法将所需的存储空间降到最低，代价是为特定的操作花费一点点额外的时间。
　　下图解释了空链表（即，没有元素的链表）在上图每种情况中是怎样表示的。注意sentinel在图（c）中总是存在的。另一方面，没有“岗哨”，对None的引用的链表变种用来象征空链表。

　　接下来，我们将介绍一个普通的单链表的实现细节。我们已经选择了实现变种（b）－－用head和tail的那个－－因为它能很好地支持扩展和前置操作。

4.3.1　实现
　　下图表示了我们之前选择的单链表方案的实现。用到了两种相关联的结构。链表的元素使用Element类的实例来实现，这些实例包括_list,_datum和_next，主体结构是LinkedList类的一个实例，这个实例也由两个实例属性，_head和_tail构成，它们分别指向第一和最后的结点。每个Element类的_list实例属性包含一个指向LinkedList相关联的实例。_datum实例属性用于指向链表的结点，_next指向下一个结点。

　　下面这个程序定义了LinkedList.Element类，它用于实现链表的结点，有三个实例属性，_list,_datum和_next，一个__init__方法和两个属性，datum和next。_```python

#coding:utf-8
__author__ = 'bob'

class LinkedList(object):

class Element(object):

def __init__(self, list, datum, next):
self._list = list
self._datum = datum
self._next = next

def getDatum(self):
return self._datum

datum = property(
fget= lambda self: self.getDatum)

def getNext(self):
return self._next

next = property(
fget= lambda self: self.getNext())
#...
```
　　我们来计算总共所需的存储空间，S(n)，在上面这个程序定义的类中用来控制一个有n个结点的链表：
　　S(n) = sizeof(LinkedList) + n **sizeof(LinkedList.Element)
　　         = 2** sizeof(id) + 3n **sizeof(id)
　　在Python中对象标识符占用一个常数数量的空间，因此，S(n) = O(n)。**

4.3.2　链表结点
　　LinkedList.Element类的方法在4.3.1节中已有定义，总共有三个方法－－init, getDatum和getNext。
　　init方法简单地初始化实例属性为已提供的值。给_list,_datum和_next实例属性分配值需要花费一个常数数量的时间。因此，init方法的时间复杂度是O(1)。
　　getDatum和getNext方法简单地返回相应的实例属性值。显然，每个方法的时间复杂度都是O(1)。_

4.3.3　LinkedList类的init方法
　　LinkedList类的init方法的代码在下面这个程序中给出。因为实例属性_head和_tail被设为None，所以链表默认是空的。init方法的时间复杂度明显是常数，即，T(n) = O(1)。
```python

class LinkedList(object):
def __init__(self):
self._head = None
self._tail = None
　＃...
```
4.3.4　purge方法
　　下面这个程序给出了LinkedList类的purge方法。这个方法的目的是丢弃现在的链表内容并且再次清空链表。显然，purge的运行时间是O(1)。
```python

class LinkedList(object):
def purge(self):
self._head = None
self._tail = None
#...
```
4.3.5　LinkedList属性
　　下面这个程序定义了LinkedList的三个属性。had和tail属性提供对相应的LinkedList属性的访问。isEmpty属性提供一个返回bool型结果的访问器来判断链表是否是空的。显然，每个访问器的时间复杂度是O(1)。
```python

#coding:utf-8
__author__ = 'bob'

class LinkedList(object):

def getHead(self):
return self._head

head = property(
fget= lambda self: self.getHead())

def getTail(self):
return self._tail

tail = property(
fget= lambda self: self.getTail())

def getIsEmpty(self):
return self._head is None

isEmpty = property(
fget= lambda self: self.getIsEmpty())

#...
```
4.3.6　第一和最后的属性
　　下面这个程序定义了另外两个LinkedList属性。第一个属性提供返回链表第一个结点的访问器，相似地，最后一个属性提供返回链表最后一个结点的访问器。两种方法的实现代码几乎一样。倘若链表是空的，就会引发ContainerEmpty异常。
```python

#coding:utf-8
__author__ = 'bob'

class LinkedList(object):
def getFirst(self):
if self._head is None:
raise ContainerEmpty
return self._head._datum

first = property(
fget= lambda self: self.getFirst())

def getLast(self):
if self._tail is None:
raise ContainerEmpty
return self._tail._datum

last = property(
fget= lambda self: self.getLast())

#...
```
　　我们将假设是在无bug的程序中，第一和最后的属性访问器都不会被空链表调用。在这种情况下，每个方法的时间复杂度都是常数，即，T(n) = O(1)。

4.3.7　前置方法
　　要预置一个结点到链表可以在链表的第一个结点前插入这个结点，这个前置的链表元素成为新的链表头结点。下面这个程序给出了LinkedList类的前置方法的算法。
```python

class LinkedList(object):

def prepend(self, item):
tmp = self.Element(self, item, self._head)
if self._head is None:
self._tail = tmp
self._head = tmp
　　
　　#...
```

　　前置方法首先创建一个新的LinkedList.Element，它的_datum实例属性用要被前置到链表的值，item，来初始化；_next实例属性通过初始化_head的当前值指向现存链表的第一个结点。如果最初是空的，_head和_next都指向新的结点，否则，只有_head需要更新。
　　注意，LinkedList.Element实例通过调用它的init方法来初始化。4.3.2节中阐述过，init方法的时间复杂度是O(1)。前置方法由于只需增加一个常数数量的工作，所以它的时间复杂度也是O(1)。

4.3.8　扩展方法
　　下面这个程序给出了扩展方法，它会在链表的尾部增加一个新的LinkedList.Element。增加的结点成为链表新的尾结点。
```python

class LinkedList(object):

def append(self, item):
tmp = self.Element(self, item, None)
if self._head is None:
self._head = tmp
else:
self._tail._next = tmp
self._tail = tmp
　　
　　#...
```
　　扩展方法首先分配一个新的 LinkedList.Element。它的_datum实例属性用要增加的值初始化，并且把_next实例属性设为None。如果链表最初是空的，那么_head和_tail都指向新的元素，否则，新的元素将被添加到现存的链表，并且_tail尾指针将更新。
　　扩展方法的时间复杂度本质上和前置方法是一样的，即，O(1)。_

4.3.9　copy方法
　　下面这个程序给出了LinkedList类的copy方法。该方法用来创建对一个给定的链表的浅复制。
```python

class LinkedList(object):

def __copy__(self):
result = LinkedList()
ptr = list._head
while ptr is not None:
result.append(ptr._datum)
ptr = ptr._next
return result

　　#...
```
　　copy方法首先创建一个新的空LinkedList实例。然后，它调用扩展方法逐个遍历给定的链表来增加条目到新的链表。
　　在上一节中我们讲到扩展方法的时间复杂度是O(1)，如果结果链表有n个结点，扩展方法将被调用n次，因此，copy方法的时间复杂度是O(n)。

4.3.10　提取方法
　　在这一节我们讨论LinkedList类的提取方法，这个方法的目的是从链表删除指定的结点。
```python

class LinkedList(object):

def extract(self, item):
ptr = self._head
prevPtr  = None
while ptr is not None and ptr._datum is not item:
prevPtr = ptr
ptr = ptr._next
if ptr is None:
raise KeyError
if ptr == self._head:
self._head = ptr._next
else:
prevPtr._next = ptr._next
if ptr == self._tail:
self._tail = prevPtr

　　#...
```
　　提取方法按顺序查找要被删除的条目。由于缺乏先验知识，我们不知道要被删除的链表元素的条目能否找到。事实上，特定的条目甚至可能根本不在链表中！
　　如果我们假设要被删除的条目在链表中，并且假设在每个可能的位置要找到它都是同等的概率，然后在要被删除的条目找到之前我们平均需要查找几乎整个链表。在最坏的情况下，条目将在末端找到。
　　如果要被删除的条目不存在，程序会引发KeyError异常。更简单的可选的方案可能是什么也不做——毕竟，如果要被删除的条目不存在，那就相当于已经删除了！然而，试图删除一个不存在的条目，更像是编程中的逻辑错误，这就是要引发异常的原因。
　　为了评定提取方法的时间复杂度，我们首先需要评定找到要被删除的条目的时间。如果要被删除的条目不存在，那么程序的运行时间是它引发异常（第10行）的所需的，即，T(n) = O(n)。
　　现在考虑要被删除的条目存在的情况。最坏的情况是该条目在末端。因此，找到这个结点的时间复杂度是O(n)。事实上一旦结点被找到就从链表中删除是一连串相对简单的操作。这些操作可以在常数数量的时间内完成，因此，时间复杂度总共是T(n) = O(n)。

4.3.11　insertAfter和insertBefore方法
　　下面这个程序给出了LinkedList.Element类的insertAfter和insertBefore方法。两种方法都携带一个指定了要插入到链表的变量。给定的条目要么在链表结点的前面或紧跟结点插入。
```python

class LinkedList(object):

class Element(object):

def insertAfter(self, item):
self._next = LinkedList.Element(
self._list, item, self._next)
if self._list._tail is self:
self._list._tail = self._next

def insertBefore(self, item):
tmp = LinkedList.Element(self._list, item, self)
if self is self._list._head:
self._list._head = tmp
else:
prevPtr = self._list._head
while prevPtr is not None and prevPtr._next is not self:
prevPtr = prevPtr._next
prevPtr._next = tmp

　　#...
```
　　insertAfter方法和4.3.8节的append几乎一样，然而append在链表末端后面插入条目，insertAfter在任意的链表结点后面插入条目。不过，insertAfter的时间复杂度和append一样，即，O(1)。
　　要在一个给定的链表结点前插入条目需要从头遍历链表来定位在给定的结点前面。最坏的情况下，给定的结点在链表末端，那么整个链表都需要遍历。因此，insertBefore方法的时间复杂度是O(n)。