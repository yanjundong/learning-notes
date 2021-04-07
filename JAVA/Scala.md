# 概念

## 特性

- 既可以**面向对象**风格编程，也可以**函数式**风格编程；
- 同时支持命名式风格和函数式风格；
- 能和Java无缝地相互操作；

# 使用

## 循环输出

```scala
// for循环输出
def method1() = {
    for (i <- 1 to 3) {
        // 字符串插值
        print(s"$i, ")
    }
    println("scala")

    /* 偏向函数风格 */
    (1 to 3).foreach(item => print(s"$item, "))
    println("scala")
}
```

## 元组

```scala
  // 元组：方法返回多个值
  def method2() = {
    ("andy", "18", "andy@163.com")
  }
```

## 灵活传参

```scala
  // 方法中传入多个参数
  def max(values: Int*) = {
    values.foldLeft(Integer.MIN_VALUE) {Math.max}
  }
```



```scala
// 调用方法
method3()
method3("zhejiang")
// 命名传参
method3(city = "yuncheng", province = "shanxi")

// 方法中的参数提供默认值
def method3(province: String = "shaanxi", city: String = "xian") = {
    println(s"$province, $city")
}
```

上面的例子中，参数的默认值是由函数的创建者来决定的，在Scala中提供了一种赋默认值的方法可以**由调用者来决定所传递的默认值**。

```scala
object Test3 {
  def connectToNetwork(user: String)(implicit wifi: String): Unit = {
    println(s"User: $user connect to WIFI $wifi")
  }

  def atOffice(): Unit = {
    implicit def officeNetwork = "office-network"
    val cafeteriaNetwork = "cafe-connect"
    connectToNetwork("guest")(cafeteriaNetwork)
    connectToNetwork("Andy")
    connectToNetwork("Jill")
  }

  def main(args: Array[String]): Unit = {
    atOffice()
  }
}

/*
User: guest connect to WIFI cafe-connect
User: Andy connect to WIFI office-network
User: Jill connect to WIFI office-network
*/
```

## 操作符重载

从技术角度说，Scala中没有操作符，像`+`、`-` 这些操作符都是方法名，它是利用了Scala宽松的方法调用语法——不强制在对象引用和方法名中间使用 `.`。

```scala
object Test4 {
  class Salary(val money: Int) {
    def +(s: Salary): Int = {
      money + s.money
    }
  }

  def main(args: Array[String]): Unit = {
    val sal = new Salary(5000)
    val salary = new Salary(4000)
    println(sal + salary)
  }
}
```













