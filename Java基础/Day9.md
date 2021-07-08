[toc]
## 一.包
#### 1.==package==
- 对类文件进行分类管理
- 给类提供多层命名空间
- 写在程序文件的第一行
- 类名的全称是：包名.类名
- 包也是一种封装形式

> package pack;(所有字母小写)
> 
> class PackageDemo{}

> javac -d . PackageDemo.java
> 
> java pack.PackageDemo

#### 2.包与包之间进行访问

- 被访问的包中的类以及类中的成员，需要==public==修饰；
- 不同包中的子类还可以直接访问父类中被==protected==权限修饰的成员；
- 包与包之间可以使用的权限只有两种，public，protected

--- | public | protected | default | private
---|---|---|---|---
同一个类中 | ok | ok | ok | ok
同一个包中 | ok | ok | ok | 
子类 | ok | ok |   |  
不同包中 | ok |   |   |  

- ==public类的类名必须跟java文件名保持一致==，所以一个Java文件不能出现两个及以上的共有类或接口；

#### 3.包里可以还有包（子包）

- 为了简化类名的书写，使用关键字==import==：导入的是包中的类
- import packb.haha.hehe.Demo;
- 导入hehe中所有类：import packb.haha.hehe.*;（不建议）









