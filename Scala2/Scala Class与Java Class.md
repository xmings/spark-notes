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
#### 继承及混入
一个类只能有一个父类但是可以有多个混入（分别使用关键字extend和with）

#### 自类型

自类型用于声明一个必须混入其他特质的特质但不必extend其他特质时。这样，自类型就实现了缩减`this`。通过`=>`符号实现。
```scala
trait User {
  def username: String
}

trait Tweeter {
  def print(x: String)=println(x)
  this: User =>  // 重新赋予 this 的类型
  def tweet(tweetText: String) = {
    println(s"$username: $tweetText")
    this.print("Tweeter")
  }
}
```

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

#### 泛型和上下界
**型变**
```scala
// 假如X是Y的子类
class Foo[+A] // 协变covariant，就是允许Foo[X]转化为Foo[Y]
class Foo[-A] // 逆变contravariant，就是允许Foo[Y]转为Foo[X]
class Foo[A]  // 不变invariant，就是Foo[X]和Foo[Y]没有任何关系，不可以相互转换
```

**类型upper bound和lower bound**
```scala
A <: B  // B is upper bound  
A >: B  // B is lower boud
```

### 内部类
Scala的内部类和Java内部类的区别是:每个主类的实例的子类不相同
```scala
class Test{
  class InnerTest{
    def print(i: InnerTest)={
        println(i)
    }
  }
}

val t1 = new Test //t: Test = Test@77b0ff24
val i1 = new t1.InnerTest() //i1: t1.InnerTest = Test$InnerTest@50ac63b2
val t2 = new Test //t2: Test = Test@67c912d3
val i2 = new t2.InnerTest() //i2: t2.InnerTest = Test$InnerTest@12922d53
i1.print(i2) /*
<console>:28: error: type mismatch;
 found   : t2.InnerTest
 required: t.InnerTest
       i1.print(i2)
*/
```
// 可以通过MainClass#SubClass的方式指定类型，解决该问题，如下：
```scala
    def print(i: Test#InnerTest)={
        println(i)
    }
```

### 抽象类
抽象类可以被直接实例化。


### 复合类型
指定参数类型或者返回类型时可以指定该类型继承于哪些子类
def abc(x: A with B with C): D with E with F
> 该特性是因为允许抽象类直接被实例化导致的