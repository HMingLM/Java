[toc]

# 一、泛型

### 1.泛型概述

#### 1.1.概念
用于解决安全问题，是一个类型安全机制；

#### 1.2.好处

- 将运行时期出现的问题ClassCastException,转移到了编译时期，方便程序员解决问题，让运行时期问题减少，安全；


```
    ArrayList<String> al = new ArrayList<String>();//定义一个String类型的容器；
    al.add("abc01");
    al.add(4);//即为al.add(new Integer(4));//编译错误；
```


- 避免了强制转换的麻烦；


```
    Iterator<String> it = al.iterator();
    while(it.hasNext())
    {
        String s = it.nex();
        System.out.println(s.length());
    }
```


#### 1.3.泛型格式
通过<>来定义要操作的引用数据类型；

#### 1.4.使用
- 在集合框架中很常见，只要见到<>就要定义泛型；
- 其实<>就是用来接收类型的;
- 当使用集合时，将集合中要存储的数据类型作为参数传递到<>中即可；


```
    class LenComparator implements Comparator<String>
    {
        public int compare(String o1,String o2)
        {
            int num = new Integer(o1.length()).compareTo(new Integer(o2.length()));
            if(num==0)
            return o1.compareTo(o2);
            return num;
        }
    }
```


### 2.泛型类
当类中要操作的引用数据类型不确定的时候，定义泛型来完成扩展； 


```
    class Tool<QQ>
    {
        private QQ q;
        public void setobject(QQ q)
        {
            this.q = q;
        }
        public QQ getobject()
        {
            return q;
        }
    }
    class Demo
    {
        public static void main(String[] args)
        {
            Tool<Worker> t = new Tool<Worker>();
            t.setobject(new Worker());
            Worker w = t.getobject();
        }
    }
```


### 3.泛型方法
- 泛型类定义的泛型，在整个类中有效，如果被方法使用，那么泛型类的对象明确要操作的具体类型后，所有要操作的类型就固定了；
- 为了让不同方法可以操作不同类型，而且类型还不确定，那么可以将泛型定义在方法上；
- 格式：<>要在返回值前，修饰符后；
- 特殊：静态方法不可以访问类上的泛型；(可定义在方法上)


```
    class Demo
    {
        public <T> void show(T t)
        {
            System.out.println(t);
        }
        public <Q> void show(Q q)
        {
            System.out.println(q);
        }
    }
    class Demo1
    {
        public static void main(String[] args)
        {
            Demo d = new Demo();
            d.show("haha");
            d.show(new Integer(4));
            d.print("heihei");
        }
    }
```


### 4.泛型接口

### 5.泛型限定
?是通配符，也可以理解为占位符；

泛型的限定：
- **<? extends E>**:可以接收E类型或E的子类型；(上限)
- **<? wxtends E>**:可以接收E类型或E的父类型；(下限)

# 二、Map集合

Map集合：该集合存储键值对；一对一对地存，而且要保证键得唯一性；

### 1.共性方法
##### 1.1 添加
- **put(K key,V value)**:

> 添加元素时，如果出现添加相同的键，那么后添加的值会覆盖原有键对应值；
> 
> put方法还会返回被覆盖的值；

- **putAll(Map<? extends K,? edxtends V> m)**

##### 1.2 删除
- **clear()**
- **remove(Object key)**：删除指定键，并返回对应值；

##### 1.3 判断
- **containsValue(Object value)**
- **containsKey(Object key)**
- **isEmpty()**

##### 1.4 获取

- **get(Object key)**：获取对应值，不存在返回null；
- **size()**
- **values()**：获取Map集合中所有的值；

### 2.Map集合的两种取出方式

- **Set<k> keySet()**：将Map中所有的键存入到Set集合；因为Set具备迭代器，所以可由迭代方式取出所有的键，再根据get方法获取每一个键对应的值；


```
    Map<String,String> map = new HashMap<String,String>();
    map.put("02","lisi2");
    map.put("03","lisi3");
    Set<String> keyset = map.keySet();//获取map集合所有键的Set集合；
    Iterator<String> it = keyset.iterator();//获取其迭代器；
    while(it.hasNext())
    {
        String key = it.next();
        String value = map.get(key);
        System.out.println(key+":"+value);
    }
```


- **Set<Map.Entry<k,v>> entrySet()**：将map集合中的映射关系存入到了set集合中，而这个关系的数据类型就是Map.Entry；


```
    Map<String,String> map = new HashMap<String,String>();
    map.put("02","lisi2");
    map.put("03","lisi3");
    Set<Map.Entry<String,String>> entryset = map.entrySet();
    Iterator<Map.Entry<String,String>> it = new Iterator();
    while(it.hasNext())
    {
        Map.Entry<String,String> me = it.next();
        String key = me.getKey();
        String value = me.getValue();
        System.out.println(key+":"+value);
    }
```


### 3.Map子类对象特点

和Set很像，其实Set底层就是使用了Map集合；

- **Hashtable**:底层是哈希表数据结构，不可以存入null键null值；该集合是线程同步的；效率低；

为了成功地在哈希表中存储和获取对象，用作键的对象必须实现hashCode方法和equals方法；

- **HashMap**:底层是哈希表数据结构，允许使用null键和null值；该集合是线程不同步的；效率高；
- **TreeMap**:底层是二叉树数据结构；线程不同步；可以用于给Map集合中的键进行排序；


















