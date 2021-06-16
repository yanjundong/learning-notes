# C++基础

## 基础

### 常量

C++定义常量有两种方式：

- `#define`  宏常量：`#define Day 7`
  -  通常在文件上方定义，表示一个常量
- `const`修饰的变量 `const int day = 7`
  - 通常在变量定以前加关键字const，修饰该变量为常量，不可修改

### 关键字

C++关键字如下：

![image-20210112195025998](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210112195025998.png)

### 标识符命名规则

- 标识符不能是关键字
- 标识符只能由字母、数字、下划线组成
- 标识符第一个字符必须为字母或下划线
- 标识符中字母区分大小写

## 数据类型

### 整型

作用：整型变量表示的是整数类型的数据

C++中能够表示证书的类型有以下几种方式，**区别在于所占内存空间不同：**

| 数据类型  | 占用空间                                            | 取值范围           |
| --------- | --------------------------------------------------- | ------------------ |
| short     | 2字节                                               | （-2^15 ~ 2^15-1） |
| int       | 4字节                                               | （-2^31 ~ 2^31-1） |
| long      | Windows为4字节，Linux为4字节（32位），8字节（64位） | （-2^31 ~ 2^31-1） |
| long long | 8字节                                               | （-2^63 ~ 2^63-1） |

### sizeof关键字

作用：统计数据类型所占内存大小

例子：

```c++
sizeof(short);
int num = 10;
sizeof(num);
```

### 实型（浮点型）

作用：表示小数

浮点型变量分为两种：

- 单精度float
- 双精度double

二者的区别在于表示的有效数字范围不同：

| 数据类型 | 占用空间 | 有效数字范围    |
| -------- | -------- | --------------- |
| float    | 4字节    | 7位有效数字     |
| double   | 8字节    | 15~16位有效数字 |

> 默认情况下，输出一个小数，会显示出6位有效数字。

### 字符型

作用：显示单个字符

例子：

```c++
char ch = 'a';
```

| 字符 | 十进制 |
| ---- | ------ |
| 0    | 48     |
| A    | 65     |
| a    | 97     |

### 字符串型

两种风格：

- C风格字符串：`char str[] = ""`

- C++风格字符串：`string str = ""`

  ```c++
  #include <string>
  
  int main() {
      string str = "hello world";
      return 0;
  }
  ```

### 布尔类型bool

bool类型占1个字节大小

## 指针

### 空指针和野指针

**空指针：**指针变量指向内存中编号为0的空间

注意：空指针指向的内存是不可以访问的

**野指针：**指针变量指向非法的内存空间

### const修饰指针

const修饰指针有3中情况：

- const修饰指针：常量指针
  - 特点：指针的指向可以修改，但是指针指向的值不能改
- const修饰常量：指针常量
  - 特点：指针的指向不可以改，指针指向的值可以改
- const即修饰指针，也修饰常量
  - 特点：指针的指向和指向的值都不可以改

示例：

```c++
int a = 10;
int b = 10;
/** 常量指针 */
const int *p1 = &a;
// *p1 = 20; 错误
p1 = &b;

/** 指针常量 */
int* const p2 = &a;
*p2 = 100;
// p2 = &b; 错误

const int* const p3 = &a;
// *p3 = 20; 错误
// p3 = &b; 错误
```

## 结构体

### 概念

属于用户==自定义的数据类型==，允许存储不同的数据类型

### 定义和使用

```c++
struct Student {
    string name;
    int age;
}

// 使用
Student stu1;
Student stu2 = {"张三", 18};
```

### 结构体数组

```c++
Student stus[3] = {
    {"张三", 18},
    {"李四", 19},
    {"王五", 20}
}
```

### 结构体中const使用场景

作用：用const来防止误操作

实例：

```c++
/** 加const防止函数中修改数据*/
void printStudent(const Student *stu) {
    // stu->age = 100; 错误
    cout << "name = " << stu->name << "age = " << stu->age << endl;
}
```


# 内存分区模型

C++程序在执行时，将内存大致分为4个区域：

- 代码区：存放函数体的二进制代码，由操作系统进行管理的
- 全局区：存放全局变量和静态变量以及常量
- 栈区：由编译器自动分配释放，存放函数的参数值，局部变量等
- 堆区：由程序员分配和释放，若程序员不释放，程序结束后由操作系统回收

**内存分区的意义：**

不同区域存放的数据，赋予不同的生命周期，给我们更大的灵活编程。

## 1 程序运行前

在程序编译后，生成了exe可执行程序，未执行该程序前分为两个区域：

**代码区：**

- 存放CPU执行的机器指令
- 代码区是`共享`的，共享的目的是对于频繁被执行的程序，只需在内存中有一分代码即可
- 代码区是`只读`的，原因是防止程序意外的修改了它的指令。

**全局区：**

- 全局变量和静态变量存放在此
- 全局区还包含了常量区，字符串常量和其它常量（const修饰的全局变量）也存放在此
- ==该区域的数据在程序结束后由操作系统释放==

![image-20210113110918493](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210113110918493.png)

## 2 程序运行后

**栈区：**

- 由编译器自动分配释放，存放函数的参数值，局部变量等
- **注意事项：**==不要返回局部变量的地址==，栈区开辟的数据由编译器自动释放

```c++
/** **注意事项：**==不要返回局部变量的地址==，栈区开辟的数据由编译器自动释放 */
int* func() {
    int a = 10; // 局部变量，存放在栈区，栈区的数据在函数执行完后自动释放
    return &a;
}

int main() {
    int *p = func();
    cout << *p << endl; // 第一次可以正确打印，是因为编译器做了保留
    cout << *p << endl; // 第二次打印是错误的
}
```

> 形参数据也会存放在栈区

**堆区：**

- 由程序员分配释放，若程序员不释放，程序结束后由操作系统回收
- 在C++中主要利用new在堆区开辟内存

## 3 new操作符

C++利用`new`操作符在堆区开辟数据

堆区开辟的数据，由程序员手动开辟，手动释放，释放利用操作符`delete`

利用`new`创建的数据，会返回该数据对应的类型的指针。

```c++
int* func() {
    // 利用new关键字，将数据开辟到堆区
    // 指针本质也是局部变量，放在栈上，指针保存的数据是在堆区。
    int* p = new int(10);
    return p;
}

void func2() {
    int* array = new int[10];
    // 使用数组
    
    // 释放堆区数组，需要加 [] 才可以
    delete[] array;
}

int main() {
    int *p = func();
    cout << *p << endl; // 第一次可以正确打印
    cout << *p << endl; // 第二次可以正确打印
}
```

# 引用

## 基本用法

作用：给变量起别名

示例：

```c++
int a = 10;
int &b = a;

cout << a << endl;	// 10
cout << b << endl;	// 10

b = 100;

cout << a << endl;	// 100
cout << b << endl;	// 100
```

## 注意事项

- 引用必须初始化
- 引用在初始化后就不能改变了

## 引用做函数参数

作用：函数传参时，可以利用引用的技术让形参修饰实参

```c++
/** 地址传递 */
void swap01(int* a, int* b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

/** 引用传递,形参会修饰实参 */
void swap02(int &a, int &b) {
    int temp = a;
    a = b;
    b = temp
}
```

## 引用做为函数返回值

作用：引用是可以作为函数返回值存在的

注意：不要返回局部变量引用

**用法：函数调用作为左值**

```c++
int& func() {
    static int a = 20;
    return a;
}

int main() {
    // 如果函数做左值，那么必须返回引用
    int &ref = func();
    cout << ref << endl;	//20
    cout << ref << endl;	//20
    
    func() = 2000;
    
    cout << ref << endl;	//2000
    cout << ref << endl;	//2000
    return 0;
}
```

## 引用的本质

本质：引用的本质在C++内部实际是一个指针常量

示例：

```c++
/** 发现是引用，转换为 int* const ref = &a; */
void func(int& ref) {
    ref = 100;	// ref是引用，转换为 *ref = 100;
}

int main() {
    int a = 100;
    
    /** 发现是引用，自动转换为 int* const ref = &a;  
    指针常量是指针指向不可更改，也说明为什么引用不可更改*/
    int &ref = a;	
    ref = 20;	// ref是引用，内部自动转换为 *ref = 100;
    
    cout << a << endl;
    cout << ref << endl;
    
    return 0;
}
```

## 常量引用

作用：常量饮用主要用来修饰形参，防止误操作

在函数形参列表中，可以加 ==const修饰形参==，防止形参改变实参。

# 函数

## 函数默认参数

在C++中，函数的形参列表中的形参是可以有默认值的。

示例：

```c++
int func(int a, int b = 10, int c = 20) {
    return a + b + c;
}
```

注意：

- 如果某个位置参数有默认值，那么从这个位置往后的参数都必须有默认值。
- 如果函数声明时有默认值，函数实现时就不能有默认值。

## 函数占位参数

C++中函数的形参列表里可以有占用参数，用来占位，在调用函数时必须填补该位置。



```c++
void func(int a, int) {
    // ...
}

func(10, 0); // 占位参数必须填补
```

## 函数重载

### 概述

作用：函数名可以相同，提高复用性

**函数重载满足条件：**

- 同一个作用域下
- 函数名称相同
- 函数参数**类型不同** 或者 个数不同 或者 **顺序不同**

注意：函数的返回值不可以作为函数重载的条件

### 注意事项

- 引用作为重载条件

  ```c++
  void func(int &a) {	// int &a = 10; 不合法
      cout << "func(int &a)" <<endl;
  }
  
  void func(const int &a) {	// const int &a = 10; 合法
      cout << "func(const int &a)" <<endl;
  }
  
  int main() {
      int a = 10;
      func(a);	// func(int &a)
      
      func(10);	// func(const int &a)
  }
  ```

- 函数重载碰到函数默认参数 

  ```c++
  void func(int a, int b = 10) {
      
  }
  
  void func(int a) {
      
  }
  
  func(10);	// 错误，出现二义性
  ```

# 类和对象

C++面向对象的三大特性：==封装、继承、多态==

## 封装

### 意义

封装的意义：

- 将属性和行为作为一个整体，表现生活中的事物。
- 将属性和行为加以权限控制

### struct 和 class 区别

在C++中struct 和 class 唯一的区别就在于 **默认的访问权限不同**：

- struct默认权限为公共
- class 默认权限为私有

```c++
class C1 {
    int m_a; // 默认情况下是私有权限
}

struct S1 {
    int m_b; // 默认情况下是公共权限
}
```

## 对象的初始化和清理

### 构造函数和析构函数

对象的初始化和清理工作是编译器强制要我们做的事情，因此如果我们不提供构造和析构，编译器会提供，但==编译器提供的构造函数和析构函数是空实现==。

- 构造函数：主要作用在于创建对象时为对象的成员属性赋值，构造函数由编译器自动调用，无需手动调用。

- 析构函数：主要作用在于对象**销毁前**系统自动调用，执行一些清理工作。

构造函数语法：`类名() {}`

析构函数语法：`~类名() {}`

```c++
class Student {
public:
    string name;
    int age;
    Student() {
        
    }
    Student(string name1, int age1) {
        name = name1;
        age = age1;
    }
    // 析构函数不可以有参数，不可以发生重载
    ~Student() {
        cout << "执行析构函数" <<endl;
    }
}
```

### 构造函数的分类和调用

分类：

- 按参数分为：有参构造和无参构造

- 按类型分为：普通构造和拷贝构造（类似Java中的克隆）

三种调用方法：

- 括号法
- 显示法
- 隐式转换法

```c++
class Student {
public:
    string name;
    int age;
    // 无参构造函数
    Student() {
        
    }
    // 有参构造函数
    Student(string name1, int age1) {
        name = name1;
        age = age1;
    }
    // 拷贝构造函数
    Student(const Student &stu) {
        name = stu.name;
        age = stu.age;
    }
    // 析构函数不可以有参数，不可以发生重载
    ~Student() {
        cout << "执行析构函数" <<endl;
    }
}

int main() {
    /** 括号法 */
    Student stu1;
	// Student stu1(); 错误，调用无参构造函数时不能加括号，如果加了编译器会认为这是一个函数声明
    Student stu2("张三", 10);
    Student stu3(stu2);
    
    
    /** 显示法 */
    Student stu4 = Student("张三", 10);
    Student stu5 = Student(stu4);
	// Student("张三", 10); 单独写就是匿名对象，当前行结束后，立马调用析构函数
    
    /** 隐式转换法 */
    Stduent stu6 = stu5;	//Student stu6 = Student(stu5);
}
```

### 拷贝构造函数调用时机

- 使用一个已经创建好的对象来初始化一个新对象
- 值传递的方式给函数参数传值
- 以值方式返回局部对象

### 构造函数调用规则

默认情况下，C++编译器至少给一个类添加3个函数

1. 默认无参构造函数
2. 默认析构函数
3.  默认拷贝构造函数，对属性进行值拷贝

构造函数调用规则如下：

- 如果用户定义有参构造函数，C++不再提供默认无参构造，但会提供默认拷贝构造
- 如果用户定义拷贝构造函数，C++不再提供其它构造函数

### 浅拷贝和深拷贝

- 浅拷贝：简单的赋值拷贝操作
- 深拷贝：在堆区重新申请空间，进行拷贝操作

### 初始化列表

C++提供了初始化列表语法，用来初始化属性。

```c++
class Person {
public:
    int a;
    int b;
    int c;
    
    // 初始化列表初始化属性
    Person(int a1, int b1, int c1)a(a1), b(b1), c(c1) {}
}

// 应用
Person p(10,20,30);
```

### 静态成员

静态成员分为：

- 静态成员变量
  - 所有对象共享同一份数据
  - 在编译阶段分配内存
  - 类内声明，类外初始化
- 静态成员函数
  - 所有对象共享同一个函数
  - 静态成员函数只能访问静态成员变量

```c++
class Student {
public:
    static int m_A; //静态成员变量
private:
    static int m_B; 
}
Student::m_A = 10;
Student::m_B = 10;
```

## C++对象模型和this指针

### this指针的用途

- 当形参和成员变量同名时，通过this指针来区分
- 在类的非静态成员函数中返回对象本身，可使用 `return *this;`

```c++
class Person {
public:
    int age;
    Person(int age) {
        this.age = age;
    }
    
    // 如果改成 Person 会调用拷贝构造函数返回一个新的对象，而不是返回 p1 本体
    Person& addAge() {
        this.age++;
        // this是指向p1的指针，而 *this 指向的就是p1这个对象本体
        return *this;
    }
}

int main() {
    
    Person p1(10);
    
    // 链式编程思想
    p1.addAge().addAge().addAge();
    
    return 0;
}
```

### 空指针访问成员函数

C++中空指针是可以调用成员函数的，但是要注意没有使用 this 指针。

如果使用了this指针，需要加以判断保证代码的健壮性。

### const修饰函数

**常函数：**

- 成员函数后加 const 就称这个函数为 常函数
- 常函数内不可以修改成员属性
- 成员属性声明时加关键字 `mutable`后，在常函数中依然可以修改。

**常对象：**

- 声明对象前加const称该对象为常对象
- 常对象只能调用常函数

```c++
class Person {
public:    
    // this指针的本质就是一个指针常量，指针的指向是不可以修改的
    // 在成员函数后加const，修饰的是this指向，让指针修改的值不可以修改
    void showPerson() const {
        this->m_B = 100; // 设为 mutable 后允许修改值
    }
    
    int m_A;
    mutable int m_B;
}
```

## 友元

在程序中，有些私有属性 也想让类外特殊的一些函数或者类进行访问，就需要用到友元。

友元的目的就是让一个函数或者类 访问另一个类中的私有成员。

友元的关键字是 `friend`

友元的三种实现方式：

- 全局函数做友元
- 类做友元
- 成员函数做友元

### 全局函数做友元

```c++
class Building {
    // goodBrother全局函数是 Building 的友元函数，可以访问其中的私有成员
    friend void goodBrother(Building &building);
public:
    Building() {
        this.sittingRoom = "客厅";
        this.bedRoom = "卧室";
    }
    string sittingRoom;
private:
    string bedRoom;
}

void goodBrother(Building &building) {
    cout << building.sittingRoom << endl;
    cout << building.bedRoom << endl; // 访问私有属性
}
```

### 类做友元

```c++
class Brother {
    
}

class Building {
    friend class Brother;
    //...    
}
```

### 成员函数做友元

```c++
class Brother {
public:
    // vist1函数不可以访问 Building 中的私有成员
    void vist1() {
        
    }
    // vist2函数可以访问 Building 中的私有成员
    void vist2() {
        
    }
}

class Building {
    friend void Brother::vist2();
    //...    
}
```

## 运算符重载

对已有运算符重新进行定义，赋予另一种功能，以适应不同的数据类型。

### 加号运算符重载

```c++
#include <iostream>
using namespace std;

class Person {
public:
    Person(int age) {
        this->age = age;
    }

    // 成员函数重载
/*    Person operator+(Person& person) {
        int temp = person.age + this->age;
        return Person(temp);
    }*/
    int age;
};

/* 全局重载函数 */
Person operator+(Person p1, Person p2) {
    int temp = p1.age + p2.age;
    return Person(temp);
}

int main() {
    Person p1(20);
    Person p2(30);
    Person p3 = p1 + p2;
    cout << p3.age << endl;

    return 0;
}

```

### 左移运算符重载

```c++
class Person {
public:
    friend ostream &operator<<(ostream &os, const Person &person) {
        os << "age: " << person.age;
        return os;
    }

    Person(int age) {
        this->age = age;
    }
    
    int age;
};

int main() {
    Person p1(20);
    cout << p1 << endl;	// age: 20

    return 0;
}
```

### 递增运算符重载

```c++
class Person {
public:

    Person(int age) {
        this->age = age;
    }

    /* 重载前置 ++ 运算符 */
    Person &operator++() {
        this->age++;
        return *this;
    }

    /* 重载后置 ++ 运算符 */
    Person operator++(int) {
        // 记录当前本身的值，然后让本身的值 + 1，但是返回以前的值
        Person temp = *this;
        this->age++;
        return temp;
    }

    int age;
};

int main() {
    Person p1(20);
    cout << ++p1 << endl;	// 21
    cout << p1++ << endl;	// 21
    cout << p1 << endl;	//22

    return 0;
}

```

### 赋值运算符重载

```c++
#include <iostream>

using namespace std;

class Person {
public:
    friend ostream &operator<<(ostream &os, const Person &person) {
        os << "age: " << *person.age;
        return os;
    }

    Person(int age) {
        this->age = new int(age);
    }

    Person& operator=(Person &person) {
        // 先判断是否有属性在堆区，如果有先释放内存，再进行深拷贝
        if (this->age != NULL) {
            delete this->age;
            this->age = NULL;
        }

        // 深拷贝
        this->age = new int(*person.age);
        return *this;
    }

    int* age;
};

int main() {
    Person p1(20);
    Person p2(30);
    Person p3(40);
    p1 = p2 = p3;
    cout << p1 << endl; //40
    cout << p2 << endl; //40
    cout << p3 << endl; //40
    
    return 0;
}
```

### 仿函数

- 重载函数调用运算符`()`
- 由于重载后使用的方法非常像函数的调用，因此成为 **仿函数**。
- 仿函数没有固定写法，非常灵活。

## 继承

### 基本语法

```c++
class Person {
    
}

class Student : public Person {
    
}

class Teacher : public Person {
    
}
```

### 继承方法

继承方法一共有3中：

- 公共继承
- 保护继承
- 私有继承

![](https://gitee.com/yanjundong97/picBed/raw/master/images/20210114230133.png)

### 继承中的对象模型

- 父类中所有非静态成员属性都会被子类继承下去，但父类中的私有成员属性是被编译器隐藏了，因此访问不到。

```c++
#include <iostream>
using namespace std;

class Person {
public:
    int p_A;
protected:
    int p_B;
private:
    int p_C;
};

// 公共继承
class Student : public Person {
public:
    int p_D;
};

int main() {
    cout << sizeof(Student) << endl;    //16
    return 0;
}
```

### 继承中的构造和析构顺序

子类继承父类后，当创建子类时，构造和析构函数的调用顺序。

```c++
#include <iostream>
using namespace std;

class Person {
public:
    Person() {
        cout << "Person 构造函数" << endl;
    }

    virtual ~Person() {
        cout << "Person 析构函数" << endl;
    }
};

class Student : public Person {
public:
    Student() {
        cout << "Student 构造函数" << endl;
    }

    virtual ~Student() {
        cout << "Student 析构函数" << endl;
    }
};

int main() {
    Student student;

    return 0;
}

/**
Person 构造函数
Student 构造函数
Student 析构函数
Person 析构函数
*/
```

> 当创建子类时，也会调用父类的构造函数，调用顺序如上输出。

### 继承同名成员处理方式

- 访问子类同名成员 直接访问即可
- 访问父类同名成员 需要加作用域

```c++
#include <iostream>
using namespace std;

class Person {
public:
    Person(int pA) : p_A(pA) {}

    int p_A;
};

class Student : public Person {
public:
    Student(int pA, int pA1) : Person(pA1), p_A(pA) {}

    int p_A;
};

int main() {
    Student student(10, 100);
    cout << student.p_A << endl; //10
    cout << student.Person::p_A << endl; //100

    return 0;
}
```

### 继承同名静态成员处理方式

静态成员和非静态成员出现同名，处理方法一致：

- 访问子类同名成员 直接访问即可
- 访问父类同名成员 需要加作用域

```c++
#include <iostream>
using namespace std;

class Person {
public:
    static int p_A;
};
int Person::p_A = 10;

class Student : public Person {
public:
    static int p_A;
};

int Student::p_A = 100;

int main() {
    cout << "<<<<<通过对象来访问<<<<<" <<endl;
    Student student;
    cout << student.p_A << endl;
    cout << student.Person::p_A << endl;

    cout << "<<<<<通过类名来访问<<<<<" <<endl;
    cout << Student::p_A << endl;
    cout << Student::Person::p_A << endl;

    return 0;
}
/**
    <<<<<通过对象来访问<<<<<
    100
    10
    <<<<<通过类名来访问<<<<<
    100
    10
*/
```

### 多继承语法

C++允许一个类继承多个类

示例：

```c++
class Student : public Base1, public Base2 {
    
}
```

==多继承可能会引发父类中有同名成员出现，需要加作用域区分。==

**因此C++实际开发中不建议使用多继承。**

### 菱形继承

概念：

- 两个派生类继承自同一个基类
- 又有某个类同时继承这两个派生类
- 这种继承称为菱形继承，又称钻石继承

典型的菱形继承：

![image-20210115102321600](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210115102321600.png)

菱形继承问题：

1. 羊继承了动物的数据，驼同样继承了动物的数据，当草泥马使用数据时，就**会产生二义性**。
2. 草泥马继承自动物的数据继承了两份，其实我们应该清楚，**这份数据我们只需要一份就可以**。

对于第一个问题，可以通过加作用域解决。

```c++
class Base {
public:
    int p_A;
};

class Father1 : public Base {};

class Father2 : public Base {};

class Son: public Father1, public Father2 {};

int main() {
    Son son;
    son.Father1::p_A = 20;
    son.Father2::p_A = 30;

    cout << son.Father1::p_A << endl;	//20
    cout << son.Father2::p_A << endl;	//30
    cout << sizeof(Son) << endl;    //8
    return 0;
}
```

对于第二个问题，需要通过`虚继承`来解决。

```c++
class Base {
public:
    int p_A;
};

// 继承前加virtual关键字后，变成虚继承
// 此时公共的父类称为虚基类
class Father1 : virtual public Base {};

class Father2 : virtual public Base {};

class Son: public Father1, public Father2 {};

int main() {
    Son son;
    son.Father1::p_A = 20;
    son.Father2::p_A = 30;

    cout << son.Father1::p_A << endl;   //30
    cout << son.Father2::p_A << endl;   //30
    cout << son.p_A << endl;    //30
    cout << sizeof(Son) << endl;    //24
    return 0;
}
```

## 多态

### 基本概念

多态分为两类：

- **静态多态**：函数重载 和 运算符重载属于静态多态，复用函数名
- **动态多态**：派生类和虚函数实现运行时多态

静态多态和动态多态的区别：

- 静态多态的函数地址早绑定 - 在编译阶段就可以确定函数地址
- 动态多态的函数地址晚绑定 - 在运行阶段才可以确定函数地址

```c++
class Father {
public:
    // 需要实现动态多态，必须要设置该函数为 `虚函数`
    virtual void method1() {
        cout << "Father --- method1" << endl;
    }
};

class Son: public Father {
public:
    void method1() {
        cout << "Son --- method1" << endl;
    }
};

class Daughter: public Father {
public:
    void method1() {
        cout << "Daughter --- method1" << endl;
    }
};

int main() {
    Father* son = new Son();
    (*son).method1();   // Son --- method1

    Father* daughter = new Daughter();
    (*daughter).method1();  // Daughter --- method1
    return 0;
}
```

> 与Java不同的是，在父类中的方法必须指定为 `虚函数` 才能实现动态多态。
>
> 如果上面的代码中，父类中的方法没有 `virtual` 关键字，就只有静态多态：
>
> ```c++
> class Father {
> public:
>     void method1() {
>         cout << "Father --- method1" << endl;
>     }
> };
> // ...
> int main() {
>     Father* son = new Son();
>     (*son).method1();   // Father --- method1
> 
>     Father* daughter = new Daughter();
>     (*daughter).method1();  // Father --- method1
>     return 0;
> }
> ```

### 纯虚函数和抽象类

纯虚函数语法：`virtual 返回值类型 函数名 (参数列表) = 0;`

当类中有了纯虚函数，这个类也就是 `抽象类`。

抽象类的特点：

- 无法实例化对象
- 子类必须重写抽象类中的纯虚函数，否则也属于抽象类

### 虚析构和纯虚析构

多态使用时，如果子类中有属性开辟到堆区，那么父类指针在释放时无法调用到子类的虚构代码。

解决方法：将父类中的析构函数改成**虚析构**或者**纯虚析构**

虚析构和纯虚析构的共性：

- 都可以解决父类指针释放子类对象
- 都需要有具体的函数实现

虚析构和纯虚析构的区别：

- 如果是纯虚析构，该类属于抽象类，无法实例化对象

解决的问题如下所示，在释放 son 对象后，没有调用 Son 的析构函数，导致没有释放堆中的数据，造成了内存泄漏。

```c++
class Father {
public:
    Father() {
        cout << "Father 构造函数" << endl;
    }
     ~Father() {
        cout << "Father 析构函数" << endl;
    }
    virtual void method1() = 0;
};

class Son: public Father {
public:
    Son() {
        cout << "Son 构造函数" << endl;
        s_A = new int(10);
    }
    ~Son() {
        if (s_A != NULL) {
            delete s_A;
            s_A = NULL;
        }
        cout << "Son 析构函数" << endl;
    }
    void method1() {
        cout << "Son --- method1" << endl;
    }

    int* s_A;
};

int main() {
    Father* son = new Son();
    (*son).method1();
    delete son;
    return 0;
}

/**
	Father 构造函数
    Son 构造函数
    Son --- method1
    Father 析构函数
*/
```

解决办法一：把父类的析构函数设置为 虚析构函数：

```c++
class Father {
public:
    //...
     virtual ~Father() {
        cout << "Father 析构函数" << endl;
    }
    virtual void method1() = 0;
};
/**
    Father 构造函数
    Son 构造函数
    Son --- method1
    Son 析构函数
    Father 析构函数
*/
```

解决办法二：把父类的析构函数设置为 纯虚析构函数：

```c++
class Father {
public:
    Father() {
        cout << "Father 构造函数" << endl;
    }
    virtual ~Father() = 0;
    virtual void method1() = 0;
};
Father::~Father() {
    cout << "Father 纯虚析构函数" << endl;
}
/**
    Father 构造函数
    Son 构造函数
    Son --- method1
    Son 析构函数
    Father 纯虚析构函数
*/
```

> 总结:
>
> 1. `虚析构`或`纯虚析构`就是用来解决通过父类指针释放子类对象
> 2. ==如果子类中没有堆区数据，可以不写为虚析构或纯虚析构==
> 3. 拥有纯虚析构函数的类也属于抽象类

# 文件操作

C++对文件操作需要包含头文件 `<fstream>`

文件类型分为两种：

- 文本文件 - 文件以文本的 **ASCII 码**存储在计算机中
- 二进制文件 - 文件以文本的**二进制**形式存储在计算机中

操作文件的三大类：

1. `ofstream`：写操作
2. `ifstream`：读操作
3. `fstream`：读写

## 文本文件

### 写文件

 文件打开方式：

| 打开方式      | 解释                       |
| ------------- | -------------------------- |
| `ios::in`     | 为读文件而打开文件         |
| `ios::out`    | 为写文件而打开文件         |
| `ios::ate`    | 初始位置：文件尾           |
| `ios::app`    | 追加方式写文件             |
| `ios::trunc`  | 如果文件存在先删除，再创建 |
| `ios::binary` | 二进制方式                 |

注意：文件打开方式可以配合使用，利用 `|` 操作符。

```c++
#include <iostream>
#include <fstream>
using namespace std;

void fileWrite() {
    // 创建流对象
    ofstream fileOp;

    // 打开文件
    fileOp.open("D:/file.txt", ios::out);

    // 写数据
    fileOp << "name : Jack" << endl;
    fileOp << "sex : man" << endl;
    fileOp << "age : 20" << endl;

    // 关闭文件
    fileOp.close();
}

int main() {
    fileWrite();
    return 0;
}
```

![image-20210115154400398](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210115154400398.png)

### 读文件

```c++
#include <iostream>
#include <fstream>
#include <string>
using namespace std;

void fileRead() {

    ifstream fileOp;

    // 打开文件并判断是否打开成功
    fileOp.open("D:/file.txt", ios::in);
    if (!fileOp.is_open()) {
        cout << "文件打开失败！" << endl;
        return;
    }
    char buffer[1024] = {0};
    /* 第一种方法 */
    /*while (fileOp >> buffer) {
        cout << buffer << endl;
    }*/
    /* 第二种方法 */
    while (fileOp.getline(buffer, sizeof(buffer))) {
        cout << buffer << endl;
    }
    /* 第三种方法 */
    string str;
    while (getline(fileOp, str)) {
        cout << str << endl;
    }
    /* 第四种方法 */
    char c;
    while ((c = fileOp.get()) != EOF) {
        cout << c;
    }
    fileOp.close();
}

int main() {
    fileRead();
    return 0;
}
```

## 二进制文件

### 写文件

二进制文件写文件主要利用流对象调用成员函数 write

函数原型：`ostream& write(const char* buffer, int len);`

参数解释：字符指针buffer指向内存中的一段存储空间，len是读写的字节数。

```c++
class Student {
public:
    int s_A;
    int s_B;
};

void fileWrite() {
    ofstream fileOp;

    fileOp.open("D:/file.txt", ios::out | ios::binary);

    Student student;
    student.s_A = 5;    student.s_B = 10;
    fileOp.write((const char *) &student, sizeof(Student));

    fileOp.close();
}
```

### 读文件

二进制文件读文件主要利用流对象调用成员函数 read

函数原型：`istream& read(char* buffer, int len);`

参数解释：字符指针buffer指向内存中的一段存储空间，len是读写的字节数。

```c++
void fileRead() {

    ifstream fileOp;

    // 打开文件并判断是否打开成功
    fileOp.open("D:/file.txt", ios::in | ios::binary);
    if (!fileOp.is_open()) {
        cout << "文件打开失败！" << endl;
        return;
    }
    Student student;
    fileOp.read((char*) &student, sizeof(Student));
    cout << student.s_A << " " << student.s_B << endl;

    fileOp.close();
}
```

# 模版

- C++另一种编程思想称为 `泛型编程`，主要利用的技术就是模版
- C++提供了两种模版机制：**函数模版**和**类模版**

## 函数模版

### 语法

```c++
template <class T>
void mySwap(T& a, T& b) {
    T temp = a;
    a = b;
    b = temp;
}

int main() {
    /* 1.自动类型推导 */
    int a = 10;
    int b = 20;
    mySwap(a, b);

    /* 2.显示指定类型 */
    mySwap<int>(a, b);
    return 0;
}
```

> 模版必须要确定出T的数据类型，才可以使用。

### 普通函数和函数模版的区别

- 普通函数在调用时可以发生隐式类型转换
- 函数模版 用自动类型推导，不可以发生隐式类型转换
- 函数模版 用显示指定类型，可以发生隐式类型转换

### 普通函数和函数模版的调用规则

- 如果函数模版和普通函数都可以实现，优先调用普通函数
- 可以通过空模版参数列表来强制调用函数模版
- 函数模版也可以发生重载
- 如果函数模版可以产生更好的匹配，优先调用函数模版

```c++
void method(int a, int b) {
    cout << "调用的普通函数" << endl;
}

template <typename T>
void method(T& a, T& b) {
    cout << "调用的函数模版" << endl;
}

template <typename T>
void method(T& a, T& b, T& c) {
    cout << "调用重载后的函数模版" << endl;
}

int main() {
    int a = 10;
    int b = 20;
    int c = 30;
    method(a, b);   //调用的普通函数
    method<>(a, b); //调用的函数模版
    method<int>(a, b);  //调用的函数模版
    method(a, b, c);  //调用重载后的函数模版

    char c1 = 'a';
    char c2 = 'b';
    /** 如果函数模版可以产生更好的匹配，优先调用函数模版 */
    method(c1, c2); //调用的函数模版
}

```

> 既然提供了函数模版，就不要提供普通函数了，容易出现二义性。

### 模版的局限性

```c++
template <class T>
void method(T& a, T& b) {
    if(a == b) {
        // ...
    }
}
```

对于上面的代码，如果在调用时提供的是 `Person` 类型，就不能使用了。

解决方法一：运算符重载

解决方法二：具体化，利用具体化的原型实现版本，具体化会优先调用

```c++
template<> void method1(Person &p1, Person &p2) {
    //...
}
```

## 类模板

```c++
template<class T>
class Student {
public:
    Student(T name) {
        this->name = name;
    }
    T name;
};

int main() {
    Student<string> student("Rose");

    cout << student.name << endl;
    return 0;
}
```

### 函数模板和类模板的区别

- 类模板没有自动类型推导的使用方式

- 类模板在模版参数列表中可以有默认参数

  ```c++
  template<class T1, class T2 = int>
  class Student {
  public:
      Student(T1 name, T2 age) {
          this->name = name;
          this->age = age;
      }
      T1 name;
      T2 age;
  };
  
  int main() {
      Student<string> student("Rose", 20);
      cout << stu.name << " " << stu.age << endl;
      return 0;
  }
  
  ```

### 类模板中成员函数的创建时机

类模板中的成员函数和普通类中的成员函数创建时机是有区别的：

- 普通类中的成员函数一开始就可以创建
- ==类模板中的成员函数在调用时才创建==

### 类模板对象做函数参数

类模板实例化出的对象，向函数传参的方式：

- 指定传入的类型 - 直接显示对象的数据类型
- 参数模板化 - 将对象中的参数变为模版进行传递
- 整个类模板化 - 将这个对象类型模板化进行传递

```c++
template<class T1, class T2 = int>
class Student {
public:
    Student(T1 name, T2 age) {
        this->name = name;
        this->age = age;
    }
    T1 name;
    T2 age;
};

/* 1. 指定传入的类型 */
void method1(Student<string, int> stu)  {
    cout << stu.name << " " << stu.age << endl;
}

/* 2. 参数模板化 */
template<class T1, class T2>
void method2(Student<T1, T2> stu)  {
    cout << stu.name << " " << stu.age << endl;
}

/* 3. 整个类模板化 */
template<class T1>
void method3(T1 stu)  {
    cout << stu.name << " " << stu.age << endl;
}

int main() {
    Student<string> student("Rose", 20);
    method1(student);
    method2(student);
    method3(student);
    return 0;
}

```

 ### 类模板与继承

当类模板碰到继承时，需要注意以下几点：

- 当子类继承的父类是一个类模板时，子类在声明的时候，要指定父类中 T 的类型
- 如果不指定，编译器无法给子类分配内存
- 如果想灵活指定父类中 T 的类型，子类也需要变成类模板

```c++
template<class T>
class Father {
    T m;
}

// 错误，必须要知道父类中 T 的类型，否则不能继承给子类
class Son : public Father {}

// 正确用法
class Son : public Father<int> {}

template<class T, class U>
class Son : public Father<T> {
    U n;
}
```

### 成员函数的类外实现

```c++
template<class T, class U>
class Student {
public:
    Student(T name, U age);
    
    void printStu();
    T name;
    U age;
};

template<class T, class U>
Student<T, U>::Student(T name, U age) {
    this->name = name;
    this->age = age;
}
template<class T, class U>
void Student<T, U>::printStu() {
    cout << this->name << " " << this->age << endl;
}
```

### 类模板分文件编写

问题：

- 类模板中成员函数的创建时机是在调用阶段，导致分文件编写时链接不到

解决：

- 方法一：直接包含 `.cpp` 文件

  ```c++
  /** Student.h */
  #ifndef LEECODE_STUDENT_H
  #define LEECODE_STUDENT_H
  
  #include <iostream>
  #include <string>
  
  using namespace std;
  
  template<class T, class U>
  class Student {
  public:
      Student(T name, U age);
  
      void printStu();
      T name;
      U age;
  };
  
  #endif //LEECODE_STUDENT_H
  ```

  ```c++
  /** Student.cpp */
  #include "Student.h"
  
  template<class T, class U>
  Student<T, U>::Student(T name, U age) {
      this->name = name;
      this->age = age;
  }
  template<class T, class U>
  void Student<T, U>::printStu() {
      cout << this->name << " " << this->age << endl;
  }
  ```

  ```c++
  #include <iostream>
  #include <string>
  /** 直接包含 .cpp 文件*/
  #include "Student.cpp"
  
  using namespace std;
  
  int main() {
      Student<string, int> stu("Jack", 20);
      stu.printStu();
  
      return 0;
  }
  ```

- 方法二：将声明和实现写到同一个文件中，并更改后缀为 `.hpp` ，hpp是约定的名称，并不是强制。【主流的解决办法】

  ```c++
  /** Student.hpp */
  #ifndef LEECODE_STUDENT_HPP
  #define LEECODE_STUDENT_HPP
  
  #include <iostream>
  #include <string>
  
  using namespace std;
  
  template<class T, class U>
  class Student {
  public:
      Student(T name, U age);
  
      void printStu();
      T name;
      U age;
  };
  
  template<class T, class U>
  Student<T, U>::Student(T name, U age) {
      this->name = name;
      this->age = age;
  }
  template<class T, class U>
  void Student<T, U>::printStu() {
      cout << this->name << " " << this->age << endl;
  }
  
  #endif //LEECODE_STUDENT_HPP
  ```

  ```c++
  #include <iostream>
  #include <string>
  #include "Student.hpp"
  
  using namespace std;
  
  int main() {
      Student<string, int> stu("Jack", 20);
      stu.printStu();
  
      return 0;
  }
  ```

### 类模板与友元

 # STL

## 基本概念

- STL(`Standard Template Libary`, 标准模板库)
- STL从广义上分为`容器(container)`、`算法(algorithm)`、`迭代器(iterator)`
- **容器**和**算法**之间通过**迭代器**进行无缝连接。
- STL几乎所有的代码都采用了模板类和模版函数

## 六大组件

STL大体分为六大组件，分别是：**容器、算法、迭代器、仿函数、适配器(配接器)、空间配置器**

1. 容器：各种数据结构，如vector、list、deque、set、map等，用来存放数据。
2. 算法：各种常用的算法，如sort、 find、copy、for_each等
3. 迭代器：扮演了容器与算法之间的胶合剂。
4. 仿函数：行为类似函数，可作为算法的某种策略。
5. 适配器：一种用来修饰容器或者仿函数或迭代器接口的东西。
6. 空间配置器：负责空间的配置与管理。

## string容器

### 基本概念

- string是C++风格的字符串，而string本质上是一个类

string和`char*`的区别：

- `char*` 是一个指针
- string是一个类，类内部封装了`char*`，管理这个字符串，是一个`char*`型的容器。

### string构造函数

- `string();`
- `string(const char* s);`
- `string(const string& str)`
- `string(int n, char c)` //使用n个字符c初始化

### string赋值操作

赋值的函数原型：

- `string& operator=(const char* s);`  // char*类型字符串赋值给当前的字符串
- `string& operator=(const string &s) ;`
- `string& operator=(char c);`
- `string& assign(const char *s );`
- `string& assign(const char *s, int n);`	// 把字符串s的前n个字符赋值给当前的字符串
- `string& assign(const string &s) ;`
- `string& assign(int n, char c);`

### string字符串拼接

函数原型：

- `string& operator+=(fonst char* str);`
- `string& operator+=( const char c);`
- ``string& operator+=( const string& str);`
- `string& append(const char *s) ;`
- `string& append(const char *s, int n);`
- `string& append(const string &s) ;`
- ``string& append(const string &s，int pos,int n);`

### string查找与替换

- `int find(const string& str, int pos = 0) const;`	//查找str第一次出现位置,从pos开始查找
- `int find( const char* s, int pos = 0) const;`	//查找s第一次出现位置,从pos开始查找
- `int find(const char* s, int pos, int n) const;` /从pos位置查找s的前n个字符第一次位置
- `int find(const char c, int pos = 0) const;` //查找字符c第一次出现位置
- `int rfind(const string& str, int pos = npos ) const;` //查找str最后一次位置,从pos开始查找
- `int rfind( const char* s， int pos = npos) const;` //查找s最后一次出现位置,从pos开始查找
- `int rfind(const char* s， int pos，int n) const;` //从pos查找s的前n个字符最后一次位置
- `int rfind(const char c, int pos = 0) const;` //查找字符c最后一次出现位置
- `string& replace(int pos， int n，const string& str);` //替换从pos开始n个字符为字符串str
- `string& replace(int pos， int n,const char* s);` //替换从pos开始的n个字符为字符串s

## vector容器

类似Java中的 `ArrayList`。

### 构造函数

- `vector<T> v;`
- `vector(v.begin(), v.end())` // 将 `[begin, end)` 区间中的数据拷贝给本身
- `vector(n, elem)`
- `vector(const vector &vec)`

### 赋值操作

给vector容器进行赋值

**函数原型：**

- `vector& operator=(const vector &vec);`
- `assign(beg, end);` // 将 `[begin, end)` 区间中的数据拷贝赋值给本身
- `assign(n, elem);`

### 容量和大小

- `empty()`
- `capacity()` //容器的容量
- `size()` // 返回容器中元素的个数
- `resize(int num);` //重新指定容器的长度num，如果容器变长，则以默认值填充新位置。如果容器变短，则超出部分会被删除。
- `resize(int num, elem)`

### 插入和删除

- `push_back(ele);` //尾部插入元素ele
- `pop_back();` // 删除最后一个元素
- `insert(const_iterator pos, ele);`
- `insert(const_iterator pos, int count, ele);`
- `erase(const_iterator pos);`
- `erase(const_iterator start, const_iterator end);`
- `clear();`

```c++
#include <iostream>
#include <vector>
using namespace std;

void printVector(vector<int>& array) {
    for (int i = 0; i < array.size(); ++i) {
        cout << array[i] << " ";
    }
    cout << endl;
}

int main() {
    vector<int> nums;
    nums.push_back(1);
    nums.push_back(2);
    nums.push_back(3);
    nums.push_back(4);
    nums.push_back(5);

    printVector(nums);  // 1 2 3 4 5

    // 删除最后一个元素
    nums.pop_back();
    printVector(nums);  // 1 2 3 4

    // 插入 第一个参数为迭代器
    nums.insert(nums.begin(), 10);
    printVector(nums);  // 10 1 2 3 4

    nums.insert(nums.begin(), 2, 10);
    printVector(nums);  // 10 10 10 1 2 3 4

    // 删除
    nums.erase(nums.begin());
    printVector(nums);  // 10 10 1 2 3 4

    nums.erase(nums.begin(), nums.begin()+2);
    printVector(nums);  // 1 2 3 4

    return 0;
}
```

### 数据存取

- `at(int index);` 
- `operator[];`
- `front();`
- `back();`

### 互换容器

**功能描述：**实现两个容器内元素进行互换

函数原型：

- `swap(vec)` // 将vec与本身的元素互换

> 实际用法：巧用 swap 收缩内存空间
>
> ```c++
> vector<int>(v).swap(v);
> ```

### 遍历

```c++
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

void printItem(int item) {
    cout << item << endl;
}

int main() {

    vector<int> nums;
    nums.push_back(10);
    nums.push_back(20);
    nums.push_back(30);
    nums.push_back(40);
    /* forEach 遍历方法*/
    for_each(nums.begin(), nums.end(), [](int value){cout << value << " ";});
    for_each(nums.begin(), nums.end(), printItem);
    /* for 遍历方法 */
    for (vector<int>::iterator begin = nums.begin(); begin != nums.end(); begin++) {
        cout << *begin << endl;
    }
    return 0;
}
```

### 预留空间

**功能描述：**减少vector在动态扩展容量时的扩展次数

**函数原型：**

- `reserve(int len);`

## deque容器

### 基本概念

- 双端数组，可以对头端进行插入、删除操作
- 比vector更快的头部插入
- 访问速度没有vector快

deque内部工作原理：

deque内部有一个**中控器**，维护每段缓冲区中的内容，缓冲区中存放真实数据。

中控器维护的是每个缓冲区的地址，使得使用deque时像一片连续的内存空间。



![image-20210118215419443](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210118215419443.png)

### 操作

deque的操作与vector类似

## stack容器

### 常用方法

构造函数：

- `stack<T> stk;`
- `stack(const stack &stk);`

赋值操作：

- `stack& operator=(const stack &stk);`

数据存取：

- `push(elem);`
- `pop();` // 从栈顶移除第一个元素
- `top();` // 返回栈顶元素

大小操作：

- `empty();`
- `size();`

## queue容器

### 常用方法

构造函数：

- `queue<T> que;`
- `queue(const queue &que);`

赋值操作

- `queue& operator=(const queue &que);`

数据存取：

- `push(elem);`
- `pop();` // 从对头移除第一个元素
- `back();` // 返回最后一个元素
- `front();` // 返回第一个元素

大小操作：

- `empty();`
- `size();`

## list容器

### 基本概念

- STL中链表(list) 是一个`双向循环链表`。
- 由于链表的存储方法并不是连续的内存空间，因此链表list中的迭代器只支持前移和后移，属于**双向迭代器**。
- 插入和删除操作都不会造成原有list迭代器的失效，这在vector中不成立。

### 构造函数

- `list<T> lst;`
- `list(beg, end);`
- `list(n, elem);`
- `list(const list &lst);`

### 赋值和交换

- `assign(beg, end);`
- `assign(n, elem);`
- `list& operator=(const list &lst);`
- `swap(lst)`

### 大小操作

- `size();`
- `empty();`
- `resize(num);`
- `resize(num, elem);`

### 插入和删除

- `push_back(elem);` //在容器尾部加入一个元素
- `pop_back();`//删除容器中最后一个元素. 
- `push_front(elem);`//在容器开头插入一个元素
- `pop_front();` //从容器开头移除第一个元素
- `insert(pos,elem);` //在pos位置插elem元素的拷贝，返回新数据的位置。.
- `insert(pos,n,elem);`//在pos位置插入n个elem数据，无返回值。
- `insert(pos,beg,end);`/l在pos位置插入[beg,end)区间的数据，无返回值。
- `clear();`//移除容器的所有数据
- `erase(beg,end);`//删除[beg,end)区间的数据，返回下一个数据的位置。
- `erase(pos);` //删除pos位置的数据，返回下一个数据的位置。
- `remove(elem);` //删除容器中所有与elem值匹配的元素。

### 数据存取

- `front();` // 返回第一个元素
- `back();` // 返回最后一个元素

### 反转和排序

- `reverse();` //反转
- `sort();` // 排序

## set/multiset 容器

### 基本概念

- 所有元素在插入时会自动排序
- `set/multiset` 属于关联式容器，底层结构都是用**二叉树**实现。



`set/multiset` 的区别：

- set不允许容器中有重复的元素
- multiset允许容器中有重复的元素

### 查找和统计

- `find(key);` // 查找key是否存在，如果存在，返回该键的元素的迭代器；若不存在，返回set.end()。
- `count(key);` // 统计key的元素个数

### pair对组创建

- 成对出现的数据，利用对组可以返回两个数据

**两种创建方式：**

- `pair<type, type> p(value1, value2);`
- `pair<type, type> = make_pair(value1, value2);`

```c++
void method1() {
    pair<string, int> p1("Jack", 20);
    cout << p1.first << " " << p1.second << endl;

    pair<string, int> p2 = make_pair("Rose", 18);
    cout << p2.first << " " << p2.second << endl;
}
```

### set容器排序

- 利用**仿函数**改变set的默认排序规则
- 自定义数据类型需要指定排序规则

```c++
class MyCompare {
public:
    bool operator()(int num1, int num2) {
        return num1 > num2;
    }
};

int main() {

    /** 默认排序 */
    set<int> set1;
    set1.insert(10);
    set1.insert(30);
    set1.insert(20);
    set1.insert(15);
    for (set<int>::iterator it = set1.begin(); it != set1.end(); ++it) {
        cout << *it << " ";
    }
    cout << endl;	/** 10 15 20 30 */

    /** 自定义排序 */
    set<int, MyCompare> set2;
    set2.insert(10);
    set2.insert(30);
    set2.insert(20);
    set2.insert(15);
    for (set<int, MyCompare>::iterator it = set2.begin(); it != set2.end(); ++it) {
        cout << *it << " ";
    }
    cout << endl;	/** 30 20 15 10 */
    return 0;
}
```

## map/multimap容器

- map/multimap容器属于关联式容器，底层结构是用二叉树实现

`map/multimap` 的区别：

- map不允许容器中有重复的key值元素
- multimap允许容器中有重复的key值元素

### map插入和删除

- `insert(elem);`
- `clear();`
- `erase(pos);`
- `erase(beg, end);`
- `erase(key);`

### 查找和统计

- `find(key);` // 查找key是否存在，如果存在，返回该键的元素的迭代器；若不存在，返回map.end()。
- `count(key);` // 统计key的元素个数

### map容器排序

- 利用**仿函数**改变set的默认排序规则
- 自定义数据类型需要指定排序规则

## STL-函数对象

### 函数对象

#### 函数对象概念

- 重载**函数调用操作符**的类，其对象常称为**函数对象**
- **函数对象**使用重载的 `()` 时，行为类似函数调用，也叫**仿函数**

函数对象（仿函数）是一个类，不是一个函数。

#### 函数对象使用

**特点：**

- 函数对象在使用时，可以像普通函数那样调用，可以有参数，可以有返回值

  ```c++
  class MyAdd {
  public:
      int operator()(int num1, int num2) {
          return num1 + num2;
      }
  };
  
  int main() {
      MyAdd myAdd;
      cout << myAdd(10, 20) << endl;
      return 0;
  }
  ```

- 函数对象超出普通函数的概念，函数对象可以有自己的状态

  ```c++
  class MyPrint {
  public:
      int count;
      MyPrint() {
          this->count = 0;
      }
      void operator()(string str) {
          this->count++;
          cout << str << endl;
      }
  };
  
  int main() {
      MyPrint myPrint;
      myPrint("hello world");
      myPrint("hello world");
      myPrint("hello world");
      cout << "方法调用了" << myPrint.count << "次" << endl;
      return 0;
  }
  ```

  

- 函数对象可以作为参数传递

  ```c++
  class MyPrint {
  public:
      void operator()(string str) {
          cout << str << endl;
      }
  };
  
  void method(MyPrint myPrint, string str) {
      myPrint(str);
  }
  
  int main() {
      MyPrint myPrint;
      method(myPrint, "hello c++");
      return 0;
  }
  ```

### 谓词

#### 谓词概念

- 返回bool类型的仿函数称为**谓词**
- 如果`operator()`接收一个参数，称为一元谓词
- 如果`operator()`接收一个参数，称为一元谓词

#### 一元谓词

```c++
class GreaterFive {
public:
    bool operator()(int num) {
        return num > 5;
    }
};

int main() {
    vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    vector<int>::iterator it = find_if(nums.begin(), nums.end(), GreaterFive());
    if (it != nums.end()) {
        cout << "存在大于5的值" << endl;
    } else {
        cout << "没有大于5的值" << endl;
    }
    return 0;
}
```

#### 二元谓词

```c++
class MyCompare {
public:
    bool operator()(int num1, int num2) {
        return num1 > num2;
    }
};

int main() {
    vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    sort(nums.begin(), nums.end(), MyCompare());
    for(vector<int>::iterator ite = nums.begin(); ite != nums.end(); ++ite) {
        cout << *ite << " ";
    }
    cout << endl;
    return 0;
}

```

### 内建函数对象

STL内建了一些函数对象。

**分类：**

- 算术仿函数
- 关系仿函数
- 逻辑仿函数

**用法：**

- 这些仿函数所产生的的对象，用法和一般函数完全相同
- 使用内建函数对象，需要引入头文件`#include<functional>`

#### 算术仿函数

**功能描述：**

- 实现四则远算
- 其中 `negate` 是一元运算，其他都是二元运算

**仿函数原型：**

- `template<class T> T plus<T>`  // 加法仿函数
- `template<class T> T minus<T>`  // 减法仿函数
- `template<class T> T multiplies<T>`  // 乘法仿函数
- `template<class T> T divides<T>` // 除法仿函数
- `template<class T> T modulus<T>` // 取模仿函数
- `template<class T> T negate<T>` // 取反仿函数

```c++
#include <iostream>
#include <functional>
using namespace std;

int main() {
   plus<int> p;
   cout << p(10, 20) << endl;

   negate<int> ne;
   cout << ne(10) << endl;

    return 0;
}
```

#### 关系仿函数

**仿函数原型：**

- `template<class T> bool equal_to<T>` //等于
- `template<class T> bool not_equal<T>` //不等于
- `template<class T> bool greater<T>` //大于
- `template<class T> bool greater_equal<T>` //大于等于
- `template<class T> bool less<T>` //小于
- `template<class T> bool less_equal<T>` //小于等于

#### 逻辑仿函数

**仿函数原型：**

- `template<class T> bool logical_and<T>` //逻辑与
- `template<class T> bool logical_or<T>` //逻辑或
- `template<class T> bool logical_not<T>` //逻辑非

## STL常用算法

### 概述

- 算法主要是由头文件`<algorithm>` `<functional>` `<numeric>` 组成
- `<algorithm>`  是所有STL头文件中最大的一个，范围涉及到比较、交换、查找、遍历、复制、修改等
- `<numeric>`  只包括了几个在序列上面进行简单数学运算的模版函数
- `<functional>` 定义了一些模板类，用以声明函数对象

### 常用遍历算法

#### for_each 算法

三种使用方法：

```c++
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

void printItem(int item) {
    cout << item << " ";
}

class PrintVector {
public:
    void operator()(int num) {
        cout << num << " ";
    }
};

int main() {

    vector<int> nums;
    nums.push_back(10);
    nums.push_back(20);
    nums.push_back(30);
    nums.push_back(40);
    /* forEach 遍历方法*/
    for_each(nums.begin(), nums.end(), [](int value){cout << value << " ";});
    cout << endl;
    for_each(nums.begin(), nums.end(), printItem);
    cout << endl;
    for_each(nums.begin(), nums.end(), PrintVector());
    cout << endl;

    return 0;
}
```

#### transform

**功能描述：**

- 搬运容器到另一个容器中

**函数原型：**

- `transform(iterator beg1, iterator end1, iterator beg2, _func);`

### 常用查找算法

- `find` //查找元素
- `find_if` //按条件查找元素
- `adjacent_find` //查找相邻重复元素
- `binary_search` //二分查找法
- `count` //统计元素个数
- `count_if` //按条件统计元素个数

#### find

- 查找指定元素，找到返回指定元素的迭代器，找不到返回结束迭代器
- `find(iterator beg, iterator end, value);`

```c++
int main() {
    vector<int> nums;
    nums.push_back(10);
    nums.push_back(20);
    nums.push_back(30);
    nums.push_back(40);
    vector<int>::iterator  ite = find(nums.begin(), nums.end(), 5);
    if (ite != nums.end()) {
        cout << "存在该元素" << endl;
    } else {
        cout << "不存在该元素" << endl;
    }
    return 0;
}

```

> 如果操作自定义数据类型，需要重载 `==` 操作符。

#### find_if

- 按条件查找元素
- `find_if(iterator beg, iterator end, _Pred);` // _Pred 为函数或谓词（返回bool类型的仿函数）

```c++
class GreaterFive {
public:
    bool operator()(int num) {
        return num > 5;
    }
};

int main() {
    vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    vector<int>::iterator it = find_if(nums.begin(), nums.end(), GreaterFive());
    if (it != nums.end()) {
        cout << "存在大于5的值" << endl;
    } else {
        cout << "没有大于5的值" << endl;
    }
    return 0;
}
```

#### adjacent_find

- 查找相邻重复元素，返回相邻元素的第一个位置的迭代器，如果不存在，返回结束迭代器
- `adjacent_find(iterator beg, iterator end)` 

#### binary_search

- 查找指定元素是否存在
- `bool binary_search(iterator beg, iterator end, value);`

#### count

- 统计元素个数
- `count(iterator beg, iterator end, value);`

> 如果操作自定义数据类型，需要重载 `==` 操作符。

#### count_if

- 按条件统计元素个数
- `count_if(iterator beg, iterator end, _Pred);` // _Pred 谓词



### 常用排序算法

#### sort

- 对容器内元素进行排序
- `sort(iterator beg, iterator end, _Pred)`

#### random_shuffle

- 洗牌，指定范围内的元素随机调整次序
- `random_shuffle(iterator beg, iterator end)`

#### merge

- 容器元素合并，并存储到另一个容器中
- `merge(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest)`
- 合并时两个容器必须是**有序的**，合并后目标容器仍然是有序的。

#### reverse

- 反转指定范围的元素
- `reverse(iterator beg, iterator end)`

### 常用拷贝和替换算法

#### copy

- 容器内指定范围的元素拷贝到另一个容器中
- `copy(iterator beg, iterator end, iterator dest);`

#### replace

- 将容器内指定范围的旧元素修改成新元素
- `replace(iterator beg, iterator end, oldValue, newValue)`

#### replace_if

- 将容器内指定范围且满足条件的旧元素修改成新元素
- `replace_if(iterator beg, iterator end, _Pred, newValue)`

#### swap

- 互换两个容器的元素
- `swap(container c1, container c2);`

### 常用算术生成算法

算数生成算法属于小型算法，使用时包含的头文件为`#include <numeric>`

#### accumulate

- 计算容器元素累计总和
- `accumulate(iterator beg, iterator end, value);`

```c++
#include <iostream>
#include <vector>
#include <numeric>
int main() {
    vector<int> nums;
    for (int i = 0; i < 100; ++i) {
        nums.push_back(i);
    }
    // 第3个参数为起始累加值
    int sum = accumulate(nums.begin(), nums.end(), 0);
    cout << sum << endl;
    return 0;
}
```

#### fill

- 向容器中添加元素
- `fill(iterator beg, iterator end, value);`

```c++
int main() {
    vector<int> nums(100);
    fill(nums.begin(), nums.end(), 200);
    for_each(nums.begin(), nums.end(), [](int num) {cout << num << " ";});
    return 0;
}
```

### 常用集合算法

#### set_intersection

- 求两个容器的交集
- `set_intersection(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest)`
- 两个集合必须是**有序序列**

#### set_union

- 求两个容器的并集
- `set_union(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest)`
- 两个集合必须是**有序序列**

#### set_difference

- 求两个容器的差集
- `set_difference(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest)`
- 两个集合必须是**有序序列**

# Cmake

## add_library

该指令是将指定的源文件生成链接文件，然后添加到工程中。

```cmake
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [source1] [source2] [...])
```

- `<name>` 表示库文件名称，该库文件会根据命令里列出的源文件来创建；
- `STATIC`、`SHARED`和`MODULE`的作用是指定生成的库文件的类型。STATIC库是目标文件的归档文件，在链接其它目标的时候使用。SHARED库会被动态链接（动态链接库），在运行时会被加载。MODULE库是一种不会被链接到其它目标中的插件，但是可能会在运行时使用dlopen-系列的函数。
- source1、source2分别表示各个源文件；

## aux_source_directory

该指令会查找指定目录下的所有源文件，然后把结果存入指定变量名。

```cmake
aux_source_directory(<dir> <variable>)
```

例如：

```cmake
# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)

# 指定生成目标
add_executable(Demo ${DIR_SRCS})
```

## link_directories

该指令是指定要链接的库文件的路径，有时候不一定需要该指令。

## target_link_libraries

该指令是把目标文件和库文件进行链接。

```cmake
target_link_libraries(<target> [item1] [item2] [...]
                      [[debug|optimized|general] <item>] ...)
```

- `<target>` 是指通过`add_executable` 和 `add_library` 指令生成的已创建的目标文件；
- `[item]` 表示库文件没有后缀的名字；



































































