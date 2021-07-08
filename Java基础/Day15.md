[toc]
# 一、collections

collections:专门对**集合**进行**操作**的工具类

- **Collections.sort(List<T> list)**:只能自然排序；

```
    Collections.sort(list,new StrLenComparator());//需要时加上比较器；
```

- **Collections.max(List<T> list)**:取最大值；
- **Collections.binarySearch(List<T> list,T key)**:二分搜索法，查找元素所在角标；必须是有序集合；
- **Collections.fill(List<T> list,T key)**:将集合中所有元素替换成指定元素；
- **Collections.replaceAll(List<T> list,T old,T new)**:将某一元素替换成指定的值；
- **Collections.reverse(List<T> list)**:反转；
- **Collections.reverseOrder()**:返回一个比较器，强行逆转了实现Comparable接口的对象collection的自然顺序;   另外，在()中传入一个比较器，可将比较器逆转；

```
    TreeSet<String> ts = new TreeSet<String>(Collections.reverseOrder(new StrLenComparator()));
```

- **Collections.synchronizedList(List<T> list)**:返回指定列表支持的同步(线程安全的)列表；
- **Collections.swap(List<T> list,int i,int j)**:在指定列表的两个指定位置处置换元素； 
- **Collections.shuffle(List<T> list)**:将指定列表重新随机排序；

# 二、Arrays

Arrays:用于**操作数组**的工具类；里面都是静态方法；

### 1.asList():将数组变成list集合；

- **好处**：可以使用集合的思想和方法来操作数组中的元素；

```
    import java.util.*; 
    List<String> list = Arrays.asList(arr);
```

- 如果数组中的元素都是对象，那么变成集合时，数组中的元素就直接转成集合中的元素；
- 如果数组中的元素都是基本数据类型，那么会将该数组作为集合中的元素存在；
- 注意：

> 将数组变成集合，不可以使用集合的增删方法(UnsupportedOperationException)；
> 
> 因为数组的长度是固定的；可使用contains(),get(),indexOf(),subList();

### 2.toArray():集合转成数组
Collection接口中的toArray方法

```
    ArrayList<String> al = new ArrayList<String>();
    al.add("abc1");
    al.add("abc2");
    String[] arr = al.toArray(new String[al.size()]);//指定类型的数组；
    System.out.println(Arrays.toString(arr));
```

    
> 指定类型的数组要定义多长
- 当指定类型的数组长度小于了集合的size，该方法内部会创建一个新的数组，长度为集合的size；
- 当指定类型的数组长度大于了集合的size，就不会创建新的数组，而是使用传递进来的数组；
- 所以要创建一个刚刚好的数组最优；

> 将集合转成数组的好处
- 为了限定对元素的操作，不需要进行增删了；

# 三、新特性

### 1.高级for循环

Iterable<T>接口

实现这个接口允许对象成为"foreach"语句的目标；

**格式**：

```
    for(数据类型 变量名 ：  被遍历的集合(Collection)或者数组
    {
        
    }
```

对集合进行遍历
- 只能获取集合中元素而不能对集合进行操作；
- 迭代器除了遍历，还可以进行remove集合中元素的动作；若是用ListIterator，还可以在遍历过程中对集合进行增删改查动作；

```
    ArrayList<String> al = new ArrayList<String>();
    al.add("abc1");
    al.add("abc2");
    
    for(String s : al)
    {
        System.out.println(s);
    }
    即
    Iterator<String> it = al.iterator();
    while(it.hasNext())
    {
        System.out.println(it.next());
    }
```

高级for和传统for的区别：
- 高级for有一个局限性：必须有被遍历的目标；
- 在遍历数组时建议使用传统for；因为可以定义角标；

### 2.可变参数

将要操作的元素作为参数传递即可；
隐式地将这些参数封装成了数组；
    
- **注意**：可变参数一定要定义在参数列表最后面；

```
    public static void show(String str,int... arr)
    {
        System.out.println(arr.length);
    }
    show("haha",2,3,4);
    show();
```


### 3.静态导入 

- 当类名重名时，要指定具体的包名；
- 当方法重名时，要指定具备所属的对象或者类；

**import static** XXX;:导入的都是某一类中所有的静态成员；

```
    import static java.lang.System.*;//导入了System类中所有静态成员；
    import static java.util.Arrays.*;//导入的是Arrays这个类中的所有静态成员；
```


