
# 1面向对象补充


**1.1 对象上转**

```java
// 假设Person 是 Student 父类
Person per = new Student([参数][参数])
```
_>这里最让人困惑的地方就是以当父类和子类都有一个名字方法，per 只能调用父类和子类同名的方法，但是最后真正调用的方法是子类的，而访问成员变量，per访问到的却是父类的变量，而不是子类的

**1.2 Object 方法重写** 

```java
@Override  
public boolean equals(Object obj) {  
    if (obj == null){  
        return false;  
    }  
    else if(obj instanceof Person person){  
     return this.name.equals(person.name) &&  
             this.age == person.age &&  
             this.sex.equals(person.sex);  
    }  
    return false;  
}
```
_>这里最让人困惑的地方是字符串的对比是使用equals,而int 类型数据使用 == ，这个是因为int是基本数据类型没有继承Object,所以没有equals方法，equals 方法的默认实现是使用 == ， == 是比较栈里面的数据， 基本数据在栈里面保存的是数据本身，而对象在栈里面保存的是对象的地址，使用== 会比较栈里面的数据，而String 数据类型的equals 方法被重写了，使用使用equals方法会比较真实的数据本体