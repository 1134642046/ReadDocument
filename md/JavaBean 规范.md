\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.cnblogs.com\](https://www.cnblogs.com/sunTin/p/7172932.html)

### JavaBean 规范

（1）JavaBean 类必须是一个公共类，并将其访问属性设置为 public  ，如： public class user{…}  
（2）JavaBean 类必须有一个空的构造函数：类中必须有一个不带参数的公用构造器，例如：public User() {…}

（3）一个 javaBean 类不应有公共实例变量，类变量都为 private  ，如： private int id;

 (4)javaBean 属性是具有 getter/setter 方法的成员变量。也可以只提供 getter 方法，这样的属性叫只读属性；也可以只提供 setter 方法，这样的属性叫只写属性；  如果属性类型为 boolean 类型，那么读方法的格式可以是 get 或 is。例如名为 abc 的 boolean 类型的属性，它的读方法可以是 getAbc()，也可以是 isAbc()；

一般 JavaBean 属性以小写字母开头，驼峰命名格式，相应的 getter/setter 方法是 get/set 接上首字母大写的属性名。例如：属性名为 userName，其对应的 getter/setter 方法是 getUserName/setUserName。

但是，还有一些特殊情况：

1、如果属性名的第二个字母大写，那么该属性名直接用作 getter/setter 方法中 get/set 的后部分，就是说大小写不变。例如属性名为 uName，方法是 getuName/setuName。

2、如果前两个字母是大写（一般的专有名词和缩略词都会大写），也是属性名直接用作 getter/setter 方法中 get/set 的后部分。例如属性名为 URL，方法是 getURL/setURL。

3、如果首字母大写，也是属性名直接用作 getter/setter 方法中 get/set 的后部分。例如属性名为 Name，方法是 getName/setName，这种是最糟糕的情况，会找不到属性出错，因为默认的属性名是 name。

所以在 JavaBean 命名时应该注意符合以上命名规范。

### 为什么 JavaBean 属性名要求：前两个字母要么都大写，要么都小写。

一般情况下。Java 的属性变量名都已小写字母开头，如：userName,showMessage 等，但也存在着特殊情况，考虑到一些特定的有意思的英文缩略词如 (USA,XML 等),JavaBean 也允许大写字母起头的属性变量名，不过必须满足 “**变量的前两个字母要么全部大写，要么全部小写**” 的要求, 如: IDCode、ICCard、idCode 等属性变量名是合法的，而 iC、iCcard、iDCode 等属性变量名是非法的。正是由于这个原因造成了下面这种情况：   
举个例子:   
Java 代码

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
public class RegionDTO  implements Serializable{    
    public String cId;    
    public String getCid() {    
        return cid;    
    }    
    public void setCid(String cid) {    
        this.cid = cid;    
    }    
    public String cName;        
        
    public String getCName() {    
        return cName;    
    }    
    public void setCName(String name) {    
        cName = name;    
    }    
    
}
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

封装成 List 后，然后在页面上用 C 标签进行显示 

  ${item.cId}// 报错 RegionDTO 没有这个属性！！！  

转自：http://chanson.javaeye.com/blog/419028 

 

探究原因
----

### 1、背景 

本文讲的普通 JavaBean 只是一个拥有 Property（域 / 类变量）及其 setter/getter 的普通 Java 类。   
有一定 Java 开发经验的人可能会知道，普通 JavaBean 的 Property（域 / 类变量）的命名不能采用以下形式：aA\*\*\* 或者 Aa\*\*\*, 如："aDdress" 或 "Address"，否则，在 web 应用中会报无法找到这个 Property(因为根据 "规则"，需要找的是 "ADdress" 或 "address")。但对于其中的原因，一般人都不明白，难道这是 Sun 公司当初定的规范吗？   
Java 开源以后，我们终于可以解开其中的谜：   

### 2、普通 JavaBean 处理涉及到相关类 

### 在 web 应用中，Servlet 容器或者 EJB 容器一般会使用 java.beans 包中的类来加载这些 JavaBean。 

BeanInfo（接口）   
   |   
SimpleInfo（类）   
   |   
GenericBeanInfo（类）   
GenericBeanInfo 是 JavaBean 数据装载类。   
Introspector 是 JavaBean 处理中最重要的一个处理类。   
另外的一些辅助类，就不一一列举了。   

### 3、解密 

3.1 开始   
在应用中，我们通常会用以下代码来获取一个普通 JavaBean 相关的信息：   
[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
BeanInfo mBeanInfo = null; 
try { 
mBeanInfo = Introspector.getBeanInfo(Person.class); 
} catch (IntrospectionException e) { 
e.printStackTrace(); 
}
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码") 3.2 深入   
在 Introspector 类的 getBeanInfo 方法中，我们发现其中与 Property 处理相关的行：   
[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
private GenericBeanInfo getBeanInfo() 
        throws IntrospectionException { 
        …… 
        PropertyDescriptor apropertydescriptor\[\] = getTargetPropertyInfo(); 
        …… 
    }
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码") 3.3 继续深入   
在 Property 处理方法中，我们发现其处理方式是根据 getter/setter 的方法来得到 Property（域 / 类变量）   

```
private PropertyDescriptor\[\] getTargetPropertyInfo() throws IntrospectionException{ 
   …… 
if(s.startsWith("get")) obj = new PropertyDescriptor(decapitalize(s.substring(3)), method, null); 
…… 
}
```

3.4 关键   
接下来，最关键的就是下面这个方法：   
[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
public static String decapitalize(String s) 
    { 
        if(s == null || s.length() == 0) 
            //空处理 
            return s; 
        if(s.length() > 1 && Character.isUpperCase(s.charAt(1)) && Character.isUpperCase(s.charAt(0))){ 
            //长度大于1，并且前两个字符大写时，返回原字符串 
            return s; 
        } else{ 
            //其他情况下，把原字符串的首个字符小写处理后返回 
            char ac\[\] = s.toCharArray(); 
            ac\[0\] = Character.toLowerCase(ac\[0\]); 
            return new String(ac); 
        } 
    }
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")