### Scala类 与 Java类

### 类的定义
> - scala类的定义有点反主流，在class名后面就是参数列表，这样就省掉了单独写构造方法。
> - 类的参数和方法的参数，形参和类型以分号分割（和ES6和Python3相同），可以通过类型后面通过`=`指定默认值。
> - 类的形参可以通过val和var定义，不带val或var的参数是私有的，带val参数是不可变的，默认private val的。方法的参数implict强制为val；
> - scala方法的定义又有像Python，使用def关键字。
> - 可以使用带this或者省略来访问成员变量和方法。
> - 使用override关键字重写方法

```scala
class Test(name: String, age: Int){
    def sayHello() = {
        println(s"Hello, ${this.name}")
    }

    def addAge(year: Int):Int = {
        // age = age + year // error: reassignment to val，因为修改了val的值
        // val age = age + year // error: recursive value age needs type
        // year = year + 1 // error: reassignment to val
        return year
    }
}
val t = new Test("wms",100) // 输出 t: Test = Test@7f66d549
t.sayHello() // 输出 Hello, wms
t.name //error: value name is not a member of Test

class Test(val name: String, var age: Int, sex: String){}
val t = new Test("wms", 20, "") //t: Test = Test@68c3aa38
t.name // res0: String = wms
t.name = "jack" // error: reassignment to val
t.age // res1: Int = 20
t.age = 21 //t.age: Int = 21
t.sex // error: value sex is not a member of Test
t.sex = "男" //error: value sex is not a member of Test
```

> 题外话，scala的设计思想好像就是"谁好抄谁，你们都一样，我再造一个"，比如上面class的参数列表的位置，勿喷。

### 成员的访问修饰符
scala有三个访问修饰符，默认是public
- public 
- private

### setter和getter
scala造了一个符号`_=`用于表示setter，切记方法名和`_=`之间没有空格，如下，可以说`name_=`就是setter的方法名
```scala
class Test(){
    private var _name = ""

    def name = _name
    def name_= (name: String):Unit = {
        this._name = name
    }
}

val t = new Test() // t: Test = Test@3d268a2c
t.name // res11: String = ""
t.name = "wms" //t.name: String = wms
```

### 静态方法、单例模式、单例对象、伴生对象
- 单例对象是一种特殊的类，有且只有一个实例。
- 和惰性变量一样，单例对象是延迟创建的，当它第一次被使用时创建。
- 当对象定义于顶层时(即没有包含在其他类中)，单例对象只有一个实例。
- 当对象定义在一个类或方法中时，单例对象表现得和惰性变量一样。
- 在Java中static成员对应于 Scala 中的伴生对象的普通成员。

**伴生**：当一个class和一个object定义在同一个文件中且名字相同（没说签名相同），这个object就叫伴生对象
```scala
// scala 命令行的:paste模式粘贴代码
class Test(name: String) // 伴生类

object Test{   // 伴生对象
    def apply(name: String):Test = {
        return new Test(name)
    }
}

val t = Test("wms") //t: Test = Test@52bef30b
```
**伴生对象+模式匹配+伴生类打造工厂方法**
```scala
class Email(val username: String, val domainName: String)

object Email {
  def fromString(emailString: String): Option[Email] = {
    emailString.split('@') match {
      case Array(a, b) => Some(new Email(a, b))
      case _ => None
    }
  }
}

val scalaCenterEmail = Email.fromString("scala.center@epfl.ch")
scalaCenterEmail match {
  case Some(email) => println(
    s"""Registered an email
       |Username: ${email.username}
       |Domain name: ${email.domainName}
     """)
  case None => println("Error: could not parse email")
}
```

### case 类
case类一特性就是自动创建伴生对象，case类之所以叫`case`类就是因为这种类总是和`match ... case` 搭配使用
1. case 类的参数是public的，而普遍的类的参数是private
```scala
case class Test(val name: String, var age: Int, sex: String){}
val t = new Test("wms", 20, "") // t: Test = Test(wms,20,)
t.sex  // String = "" 是public所以可以访问
t.sex = "男" // error: reassignment to val
```
> 但是如何查看它的伴生对象有哪些方法了?

2. case 类之间 `==` 比较时比较的是值，而不是引用比较
3. case 类的实例的`copy`方法是浅拷贝，可以使用其他copy方法作深拷贝

### 抽象类、接口和Trait
scala中也可以定义abstract类，但没有interface，取而代之的是trait。不好举例，看看官方例子吧，虽然例子也一般

```scala
trait Iterator[A] {
  def hasNext: Boolean
  def next(): A
}

class IntIterator(to: Int) extends Iterator[Int] {
  private var current = 0
  override def hasNext: Boolean = current < to
  override def next(): Int =  {
    if (hasNext) {
      val t = current
      current += 1
      t
    } else 0
  }
}


val iterator = new IntIterator(10)
iterator.next()  // returns 0
iterator.next()  // returns 1
```

### 继承、多态、泛型
继承就不用说了，extends也用过，就实验一下多态和泛型

#### 多态
```scala
class Test{
  def say(words: String) = {
      println(words)
  }

  def say(who: String, words: String){
    println(s"$who: $words")
  }
}

val t = new Test
t.say("abc") //abc
t.say("wms", "abc") //wms: abc
```
嗯，效果和java一样

#### 泛型
泛型是吧，scala偏不用`<>`，脑子好使，就造一个`[]`

> 型变是复杂类型的子类型关系与其组件类型的子类型关系的相关性。 Scala支持 泛型类 的类型参数的型变注释，允许它们是协变的，逆变的，或在没有使用注释的情况下是不变的。 在类型系统中使用型变允许我们在复杂类型之间建立直观的连接，而缺乏型变则会限制类抽象的重用性。

```scala
class Foo[+A] // A covariant class
class Bar[-A] // A contravariant class 
class Baz[A]  // An invariant class
```

##### 协变

##### 型变
##### 不变