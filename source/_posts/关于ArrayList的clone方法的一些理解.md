---
title: 关于ArrayList的clone方法的一些理解
date: 2017-08-02 10:55:37
categories:
 - Android
tags: 
 - ArrayList
 - clone
---

### 1. ArrayList的clone方法源码

```java
/**
 * Returns a shallow copy of this <tt>ArrayList</tt> instance.  (The
 * elements themselves are not copied.)
 *
 * @return a clone of this <tt>ArrayList</tt> instance
 */
public Object clone() {
    try {
        ArrayList<?> v = (ArrayList<?>) super.clone();
        v.elementData = Arrays.copyOf(elementData, size);
        v.modCount = 0;
        return v;
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError(e);
    }
}
```
该方法返回一个Object对象，所以在使用此方法的时候要强制转换。
ArrayList的本质是维护了一个Object的数组，所以克隆也是通过数组的复制实现的，对象不会被克隆，属于浅克隆。

<!-- more -->

### 2. ArrayList的clone浅复制的巧妙使用
当你需要使用remove方法移除掉集合中的对象，而非要修改集合中的对象的时候，可以选择使用。

```java
//添加两个元素
Student stJack=new Student("Jack", 13);
Student stTom=new Student("Tom", 15);
list.add(stJack);
list.add(stTom);

//克隆
ArrayList<Student> listCopy=(ArrayList<Student>) list.clone();

//移除且不修改
listCopy.remove(1);
System.out.println(list);
System.out.println(listCopy);
```
输出结果
```java
[student[name=Jack,age=13],[name=Tom,age=15]]
[student[name=Jack,age=13]]
```
内存堆栈示意图

- remove之前

![remove之前][1]

- remove之后

![remove之后][2]

> 可以看出：移除且不修改集合中的元素，只是在List内部的数组中移除了指向元素的地址，可以放心的使用clone。

### 3. 实现List的深克隆

如果你想要修改克隆后的集合，那么克隆前的也会被修改。那么就需要使用深克隆。
在浅度克隆的基础上，对于要克隆的对象中的非基本数据类型的属性对应的类，也实现克隆，这样对于非基本数据类型的属性，复制的不是一份引用，即新产生的对象和原始对象中的非基本数据类型的属性指向的不是同一个对象

**深克隆步骤：**
> 要克隆的类和类中所有非基本数据类型的属性对应的类
1、都实现java.lang.Cloneable接口
2、都重写java.lang.Object.clone()方法

代码实例

```java
public class testClone {
    public static void main(String[] args) {
            ArrayList<Student> list=new ArrayList<Student>();
            //添加两个元素
            Student stJack=new Student("Jack", 13);
            Student stTom=new Student("Tom", 15);
            list.add(stJack);
            list.add(stTom);
            //深克隆
            ArrayList<Student> listCopy=new ArrayList<Student>();
            for (Student student : list) {
                listCopy.add(student.clone());
            }
            //移除且不修改
            listCopy.get(0).setAge(20);
            System.out.println(list);
            System.out.println(listCopy);
    }
}

class Student implements Cloneable{
    private String name;
    private int age;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public Student(String name, int age) {
        super();
        this.name = name;
        this.age = age;
    }
    @Override
    public String toString() {
        return "Student [name=" + name + ", age=" + age + "]";
    }
    @Override
    protected Student clone(){
        Student stuent = new Student(this.name,this.age); 
        return stuent; 
    }
       
}
```


  [1]: http://ol9j5v5dg.bkt.clouddn.com/before_remove.png
  [2]: http://ol9j5v5dg.bkt.clouddn.com/after_remove.png