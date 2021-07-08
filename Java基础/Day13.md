[toc]
# 集合框架

![avatar](D:/其他/图片/study/集合框架.png)

## 一、体系概述
#### 1.为什么出现集合类？

面向对象语言对事物的体现都是以对象的形式，所以为了方便对多个对象的操作，就对对象进行存储，集合就是存储对象最常用的一种方式；

#### 2.数组和集合类同是容器，有何不同？

数组虽然也可以存储对象，但长度是固定的，而集合长度是可变的；数组中可以存储基本数据类型，而集合只能存储对象；

#### 3.为什么会出现这么多容器？
因为每一个容器对数据的存储方式(数据结构)都有不同；

#### 4.集合类的特点 

- 集合只用于存储对象,存储的都是对象的引用(地址)；
- 集合长度是可变的；
- 集合可以存储不同类型的对象；

**collection:根接口**

## 二、共性方法

> import java.util.*;
> 
> ArrayList al = new ArrayList();
> 
> 创建一个集合容器，使用Collection接口的子类ArrayList；
  
- **add(Object obj)**:添加元素；参数类型是Object，以便接收任意类型对象；
- **addAll(Collection<?> c)**:添加指定collection中所有元素；
- **size()**:获取个数(集合长度)；
- **remove(Object obj)**:删除指定元素；
- **removeAll(Collection<?> c)**:删除那些同时也包含在指定collection中的所有元素；
- **clear()**:清空集合
- **contains(Object obj)**:判断元素是否存在；
- **containsAll(Collection<?> c)**:判断是否包含指定conllection中的所有元素；
- **isEmpty()**:判断集合是否为空；
- **retainAll(Collection<?> c)**:仅保留那些也包含在指定collection中的元素(取交集);

## 三、迭代器

迭代器：其实就是集合的取出元素的方式；

- **iterator()**:获取在此collection的元素上进行迭代的迭代器，用于取出元素；
> Iterator it =  al.iterator();
- **hasNext()**:若仍有元素可以迭代，则返回true；
- **next()**:返回迭代的下一个元素；(迭代时循环中next调用一次，就要hasNext判断一次)
- **remove()**:从迭代器指向的collectio中移除迭代器返回的最后一个元素；

## 四、List集合

List<---Collection--->set
    
- **List**:元素是有序的，元素可重复，因为该集合体系有索引；
- **set**:元素是无序的，元素不可重复；无索引；

#### 1.List特有方法
凡是可操作角标的方法都是该体系特有的方法；

- **add(int index,Object obj)**:在指定位置插入指定元素；
- **addAll(int i ndex,Collection<?> c)**:在指定位置插入指定指定collection中所有元素；
- **get(int index)**:获取指定位置的元素；
- **remove(int index)**:移除指定位置的元素；
- **set(int index,Object obj)**:将指定位置的元素替换成指定元素；
- **subList(int fromIndex,int toIndex)**:返回指定部分；(包含头不包含尾)
- **listIterator()**:List特有的列表迭代器；

#### 2.List其它共性方法

- **indexOf(Object obj)**:判断指定元素第一次出现的位置；(不包含则返回-1)
- **lastindexOf(Object obj)**:反向索引；(最后出现的位置)
- **remove(Object obj)**:移除第一次出现的指定元素；

#### 3.ListIterator

- ListIterator:是Iterator的子接口；只能通过List集合的listIterator方法获取；
- 迭代时不能通过集合对象的方法操作集合中的元素，而Iterator方法有限，想要**添加、修改**的操作，需要用到ListIterator；


```
    ListIterator li = al.listiterator();
    while(li.hasNext())
    {
        Object obj = li.next();
        if(obj.equals("java02")
        //li.add("java009");
        li.set("java006");
    }
    while(li.hasPrevious())
    {
        S.o.p(li.previous());
    }
```


#### 4.List集合具体对象的特点
ArrayList<---List--->LinkedList

> 按需求选用容器：
> 
> ArrayList：查询和查询，不频繁的增删(用得较多)
> 
> LinkedList：元素多，频繁增删

- **ArrayList**：底层的数据结构使用的是数组结构；特点：查询速度很快但增删稍慢；线程不同步；
- **LinkedList**：底层使用的是链表数据结构；特点：增删速度很快但查询慢；
- **Vector**：底层是数组数据结构；线程同步；被ArrayList替代了；

枚举elements()是Vector特有的取出方式，相当于迭代，被迭代器取代；

#### 5.Linkedlist
特有方法:

前期版本的方法若集合中没有元素会出现异常；
后期版本出现替代方法,若集合中没有元素，会返回null；

- addFirst();--->offerFirst();
- addLast();--->offerLast();


- getFirst();--->peekFirst();
- getLast();--->peekLast();
- 获取元素，但不删除元素；


- removeFirst();--->pollFirst();
- removeLast();--->pollLast();
- 获取元素，但元素被删除；

> 堆栈数据结构：先进后出
> 
> 队列数据结构：先进先出

**注意**：List集合判断元素是否相同，依据是==元素的==equals方法；contains和remove都需要到equals

## 五、Set方法

- set:元素是无序的(存入和取出的顺序不一定一致)，元素不可重复；无索引；
- Set集合的功能和Collection是一致的(无特有)；

#### 1.HashSet
HashSet：底层数据结构是哈希表；线程是非同步的；

> HashSet保证元素唯一性：
> 
> 通过元素的两个方法hashCode和equals来完成；
> 
> 若元素的HashCode值相同，才会判断equals是否为true；
> 
> 若元素的HashCOde值不同，不会调用equals；

**所以HashSet存储自定义对象时一般都会复写方法hashCode和equals**；


```
    public int hashCode()
    {
        return name.hashCode()+age*39;(乘以一个数保证哈希值唯一)
    }
    public boolean equals(object obj)
    {
        if(!(obj instanceof Person))
        return fasle;
        Person p = (person)obj;
        return this.name.equals(p.name) && this.age == p.age;
    }
```


**注意**：
- 对于判断元素是否存在，以及删除等操作，依赖的方法是元素的hashcode和equals方法；
- 而ArrayList依赖equals方法；


#### 2.TreeSet
- TreeSet：可以对Set集合中的元素进行排序;这些元素必须具备比较性以进行排序；(排序时若主要条件相同，一定要判断一下次要条件)；
- 底层数据结构是**二叉树**；

###### 2.1.TreeSet排序的第一种方式：让元素自身具备比较性
这种方式也成为元素的自然顺序，或者叫默认顺序
- **implements Comparable**:强制让对象具备比较性；
- 元素需要实现comparable接口，**复写compareTo(Object obj)方法**自定义比较方式
- 保证数据唯一性的依据：compareTo方法return 0
- TreeSet存储自定义对象

例子：往TreeSet集合中存储自定义对象学生，按学生年龄进行排序；(次要条件是姓名)


```
    class Student implements Comparable
    public int compareTo(Object obj)
    {
        if(!(obj instanceof Student))
        throw new RuntimeException("不是学生对象");
        Student s = (Student)obj;
        if(this.age>s.age)
        reutrn 1;(正数)
        if(this.age==s.age)
        {
            return this.name.compareTo(s.name);
            //String本身具备compareTo方法
        }
        return -1;(负数)
    }
```

###### 2.1.TreeSet的第二种排序方式：让集合自身具备比较性

当元素自身不具备比较性时，或者具备的比较性不是所需要的，就需要让集合自身具备比较性；

**方法：定义比较器实现Comparator，复写compare(Object o1,Object o2),将比较器对象作为参数传递给TreeSet集合的构造函数;这样，在集合初始化时，就有了比较方式；**


```
    TreeSet ts = new TreeSet(new MyCompare());
    class mycompare implements comparator
    {
        public int compare(Object o1,Object o2)
        {
            Student s1 = (Student)o1;
            Student s2 = (Student)o2;
            int num = s1.getName().compareTo(s2.getName());
            if(num==0)
            {
                return new Integer(s1.getAge()).compareTo(new Integer(s2.getAge());
            }
            return num;
        }
    }
```

- 当两种排序都存在时，以比较器为主；
