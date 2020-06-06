# RustTips

Rust 和其它语言最大的特点，就是它的所有权系统，及在此基础之上构建的编程模型。

### 1.绑定
重要：首先必须强调下，准确地说Rust中并没有变量这一概念，而应该称为**标识符**，**目标资源**(内存，存放value)绑定到这个标识符。

### 2.move
在Rust中，和“绑定”概念相辅相成的另一个机制就是**“转移move所有权”**，意思是，可以把资源的所有权(ownership)从一个**绑定**转移（move）成另一个**绑定**。

##### let move
这个操作同样通过let关键字完成，和绑定不同的是，=两边的左值和右值均为两个标识符：
```
语法：
    let 标识符A = 标识符B;  // 把“B”绑定资源的所有权转移给“A”
```
move前后的内存示意如下：
```
Before move:
a <=> 内存(地址：A，内容："xyz")
After move:
a
b <=> 内存(地址：A，内容："xyz")
```
被move的变量不可以继续被使用。

##### 参数 move
除了通过let关键字外，还有一种move，是直接作为方法或者函数的参数，导致发生了move
代码示例：
```
fn main() {
    let mut a = String::from("life cycle test");

    move_test(a);

    println!("{:?}",a);
}

fn move_test(s: String) {
    println!("{:?}",s)
}
```
编译过程中将会抛出以下的错误：
```
error[E0382]: borrow of moved value: `a`
  --> src/main.rs:18:21
   |
14 |     let mut a = String::from("life cycle test");
   |         ----- move occurs because `a` has type `std::string::String`, which does not implement the `Copy` trait
15 | 
16 |     move_test(a);
   |               - value moved here
17 | 
18 |     println!("{:?}",a);
   |                     ^ value borrowed here after move
```

### 3.方法　函数
rust 中方法，函数，trait和　go中的方法，函数，interface有点相似。
##### go中的方法和函数
```
package main

import "fmt"

type Person struct {
	Name string
}

func main() {
	person := Person{
		Name: "person",
	}

	person.printName()

	ptr := &person

	ptr.printfName()

	ptr.printName()

	person.printfName()

	printPerson(person)
}

func (c Person)printName(){
	fmt.Println(c.Name)
}

func (c *Person) printfName(){
	fmt.Printf("%s\n",c.Name)
}

func printPerson(person Person){
	fmt.Println(person)
}
```
运行结果:
```
person
person
person
person
{person}
```
##### rust中的方法和参数
```
#[derive(Debug)]
pub struct MBtree {
    pub name: String,
    pub age: i64,
    pub number: i64,
}

impl MBtree {
    pub fn new(name: String, age: i64, number: i64) -> MBtree {
        MBtree { name, age, number }
    }

    pub fn print_name(self) -> () {
        println!("{:?}", self.name)
    }

    pub fn print_age(&self) -> () {
        println!("{:?}", self.age)
    }

    pub fn print_number(&mut self) -> () {
        println!("{:?}", self.number);
        self.number = 12;
        println!("{:?}", self.number);
    }
}


pub fn print_mbtree_number(mbtree: &mut MBtree) -> () {
    println!("{:?}", mbtree.number)
}

pub fn print_mbtree_age(mbtree: &MBtree) -> () {
    println!("{:?}", mbtree.age)
}

pub fn print_mbtree_name(mbtree: MBtree) -> () {
    println!("{:?}", mbtree.name)
}


fn main() {
    let mut mbtree = MBtree::new(String::from("mbtree"), 21, 121);

    mbtree.print_age();

    println!("{:?}", mbtree.age);

    mbtree.print_number();

    mbtree.print_age();

    print_mbtree_age(&mbtree);

    print_mbtree_number(&mut mbtree);

    print_mbtree_name(mbtree);
}
```
在rust中，参数的传递有三种类型：
- self
- &self
- &mut self
self会消费所有权，相当于move。一旦被移动，该变量被消费，后面无法继续使用。
在print_mbtree_name继续打印，
```
    print_mbtree_name(mbtree);

    println!("{:?}",mbtree.number)
```
会抛出以下的错误：
```
error[E0382]: borrow of moved value: `mbtree`
  --> src/main.rs:59:21
   |
43 |     let mut mbtree = MBtree::new(String::from("mbtree"), 21, 121);
   |         ---------- move occurs because `mbtree` has type `MBtree`, which does not implement the `Copy` trait
...
57 |     print_mbtree_name(mbtree);
   |                       ------ value moved here
58 | 
59 |     println!("{:?}",mbtree.number)
   |                     ^^^^^^^^^^^^^ value borrowed here after move

error: aborting due to previous error
```
### 4.trait
#### go interface
在go中有着interface, 主要有着以下的特点：
- 接口的使用不仅仅针对结构体，自定义类型、变量等等都可以实现接口。
- 如果一个接口没有任何方法，我们称为空接口，由于空接口没有方法，所以任何类型都实现了空接口。
- 要实现一个接口，必须实现该接口里面的所有方法。
- 接口的实现是隐式实现

例子如下：
```
package main

import "fmt"


//定义接口
type Skills interface {
    Running()
    Getname() string
}

type Student struct {
    Name string
    Age int
}


// 实现接口
func (p Student) Getname() string{   //实现Getname方法
    fmt.Println(p.Name )
    return p.Name
}

func (p Student) Running()  {   // 实现 Running方法
    fmt.Printf("%s running",p.Name)
}
func main()  {
    var skill Skills
    var stu1 Student
    stu1.Name = "wd"
    stu1.Age = 22
    skill = stu1
    skill.Running()  //调用接口
}
```
##### rust trait
rust trait有以下的特点：
- 需要显式地被实现
- trait中可以提供默认的方法，但是可以被实现者重载
- trait中的参数传递也有三种类型，self,&self,&mut self。

[一个非常好的例子](https://rustwiki.org/zh-CN/rust-by-example/trait.html)如下:
```
struct Sheep {
    naked: bool,
    name: String,
}

trait Animal {
    fn new(name: String) -> Self;

    // 实例方法签名；这些方法将返回一个字符串。
    fn name(&self) -> String;
    fn noise(&self) -> String;

    // trait 可以提供默认的方法定义。
    fn talk(&self) {
        println!("{} says {}", self.name(), self.noise());
    }

    fn sleep(self: Self)
    where
        Self: std::marker::Sized,

    //rust一共提供了5个重要的标签trait，都被定义在标准库std::marker模块中
    //
    // Sized trait，用来标示编译期可确定大小的类型
    // Unsize trait，目前该trait为实验特性，用于标示动态大小类型
    // Copy trait，用来标示可以安全地按位复制其值的类型。
    // Send trait,用来标示可以跨线程安全通信的类型
    // Sync trait，用来标示可以在线程间安全共享引用的类型。

    //If we dont add `where Self: std::marker::Sized`, It will report the following error:
    //error[E0277]: the size for values of type `Self` cannot be known at compilation time
    //   --> src/main.rs:18:14
    //    |
    // 18 |     fn sleep(self: Self)
    //    |              ^^^^       - help: consider further restricting `Self`: `where Self: std::marker::Sized`
    //    |              |
    //    |              doesn't have a size known at compile-time
    //    |
    //    = help: the trait `std::marker::Sized` is not implemented for `Self`
    //    = note: to learn more, visit <https://doc.rust-lang.org/book/ch19-04-advanced-types.html#dynamically-sized-types-and-the-sized-trait>
    //    = note: all local variables must have a statically known size
    //    = help: unsized locals are gated as an unstable feature
    //
    // error: aborting due to previous error
    {
        println!("{}", self.name())
    }
}

impl Sheep {
    fn is_naked(&self) -> bool {
        self.naked
    }

    fn shear(&mut self) {
        if self.is_naked() {
            // 实现者可以使用它的 trait 方法。
            println!("{} is already naked...", self.name());
        } else {
            println!("{} gets a haircut!", self.name);

            self.naked = true;
        }
    }
}

// 对 `Sheep` 实现 `Animal` trait。
impl Animal for Sheep {
    fn new(name: String) -> Sheep {
        Sheep {
            name: name,
            naked: false,
        }
    }

    fn name(&self) -> String {
        self.name.clone()
    }

    fn noise(&self) -> String {
        if self.is_naked() {
            String::from("baaaaah?")
        } else {
            String::from("baaaaah!")
        }
    }

    // 默认 trait 方法可以重载。
    fn talk(&self) {
        // 例如我们可以增加一些安静的沉思。
        println!("{} pauses briefly... {}", self.name, self.noise());
    }

    fn sleep(self){
        println!("{} {}", self.name(),self.name)
    }
}

fn main() {
    let mut dolly: Sheep = Animal::new(String::from("Dolly"));

    dolly.talk();
    dolly.shear();
    dolly.talk();

    dolly.sleep();

    // dolly moved, If we add the following code,It will report the error:
    //error[E0382]: borrow of moved value: `dolly`
    //    --> src/main.rs:106:5
    //     |
    // 96  |     let mut dolly: Sheep = Animal::new(String::from("Dolly"));
    //     |         --------- move occurs because `dolly` has type `Sheep`, which does not implement the `Copy` trait
    // ...
    // 102 |     dolly.sleep();
    //     |     ----- value moved here
    // ...
    // 106 |     dolly.talk();
    //     |     ^^^^^ value borrowed here after move
    //
    // error: aborting due to previous error

    // dolly.talk();

}
```

### 5.泛型

在go 目前的版本(1.14)中，暂时不支持泛型。

泛型在rust中使用广泛，可以广泛地应用在元祖、枚举、结构体定义，方法，函数，trait中。

##### 结构体和枚举定义中的泛型
```
#[derive(Debug)]
struct Point<T, U> {
    x: T,
    y: U,
}

#[derive(Debug)]
enum Error<T, E> {
    NetWork(T),
    HandShake(E),
}

#[derive(Debug)]
enum Result<T> {
    Waiting,
    Processing(T),
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
    println!("{:?}", both_integer);
    println!("{:?}", both_float);
    println!("{:?}", integer_and_float);

    //consider giving `result1` the explicit type `Result<T>`, where the type parameter `T` is specified
    //编译的时候必须知道类型
    let error1: Error<i32, i32> = Error::NetWork(12);
    let error2: Error<i32, String> = Error::HandShake(String::from("error test"));

    println!("{:?}", error1);
    println!("{:?}", error2);

    //consider giving `result1` the explicit type `Result<T>`, where the type parameter `T` is specified
    //编译的时候必须知道类型
    let result1: Result<String> = Result::Waiting;
    let result2 = Result::Processing(String::from("result test"));
    println!("{:?}", result1);
    println!("{:?}", result2);
}
```
运行结果如下，需要注意的是，编译器需要在编译过程中可以推断出所有的泛型的实际类型，可以具体查看上面的enum。
```
Point { x: 5, y: 10 }
Point { x: 1.0, y: 4.0 }
Point { x: 5, y: 4.0 }
NetWork(12)
HandShake("error test")
Waiting
Processing("result test")
```

##### 覆盖函数，方法，trait的泛型例子。
```
use std::fmt::{Debug, Display};
mod order;

#[derive(Debug)]
struct Point<U, T>
where
    U: PartialOrd + Clone,
{
    x: U,
    y: T,
}

//This is a generics of method test
impl<U, T> Point<U, T>
where
    U: PartialOrd + Clone + Display,
{
    fn getX(&self) -> &U {
        &self.x
    }

    fn getY(&self) -> &T {
        &self.y
    }

    fn compose<S>(&self, s: S)
    where
        S: PartialOrd + Display,
    {
        println!("{} , says {}", s, &self.x)
    }
}

trait Tone {
    fn hello(&self);

    fn world<V>(&self, v: V)
    where
        V: PartialOrd + Clone + Display;
}

impl<U, T> Tone for Point<U, T>
where
    U: PartialOrd + Clone + Display,
    T: PartialOrd + Clone + Display,
{
    fn hello(&self) {
        println!("tone hello x:{},y:{}", self.getX(), self.getY())
    }

    fn world<V>(&self, v: V)
    where
        V: PartialOrd + Clone + Display,
    {
        println!("tone world,x:{},y:{},v:{}", self.getX(), self.getY(), v)
    }
}

trait Howl<V>
where
    V: PartialOrd + Clone + Display,
{
    fn hello(&self);

    fn world(&self, v: V);
}

//Not recommend, we should know the type when we compile the code
impl<U, T, V> Howl<V> for Point<U, T>
where
    U: PartialOrd + Clone + Display,
    T: PartialOrd + Clone + Display,
    V: PartialOrd + Clone + Display,
{
    fn hello(&self) {
        println!("Howl hello x:{},y:{}", self.getX(), self.getY())
    }

    fn world(&self, v: V) {
        println!("Howl world,x:{},y:{},v:{}", self.getX(), self.getY(), v)
    }
}

//This is a generics of function test
fn composer<U, T, S>(point: &Point<U, T>, s: S)
where
    S: PartialOrd + Display,
    U: PartialOrd + Clone + Display,
    T: Display,
{
    println!("{} ,and {} ,and {}", point.getX(), s, point.getY())
}

fn largest<T>(list: &[T]) -> T
where
    T: PartialOrd + Copy,
{
    let mut largest = list[0];
    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn smallest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut smallest = list[0];
    for &item in list.iter() {
        if item < smallest {
            smallest = item;
        }
    }
    smallest
}

fn main() {
    let mut point = Point {
        x: 5,
        y: String::from("point test"),
    };
    println!("{:?}", point.getX());
    println!("{:?}", point.getY());

    Point::compose(&point, String::from("compose test"));

    composer(&point, String::from("composer test"));

    println!("-------------generics tone test--------------");
    Tone::world(&point, String::from("tone test"));
    Tone::hello(&point);
    println!("-------------generics howl test--------------");
    Howl::world(&point, String::from("tone test"));
    //trait objects without an explicit `dyn` are deprecated
    (&point as &dyn Howl<String>).hello();

    println!("-------------function howl test--------------");
    let number_list = vec![34, 50, 25, 100, 65];

    let i32_largest = largest(&number_list);
    let i32_smallest = smallest(&number_list);

    println!("The largest number is {}", i32_largest);
    println!("The smallest number is {}", i32_smallest);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let char_largest = largest(&char_list);
    let char_smallest = smallest(&char_list);
    println!("The largest char is {}", char_largest);
    println!("The smallest char is {}", char_smallest);
}

```


### 6.几种常用的trait
##### From和Into
From和Into会消耗目标对象的所有权。消耗旧的对象，得到一个新的对象。
```
use std::convert::From;

#[derive(Debug)]
struct FromNumber {
    value: i32,
}

#[derive(Debug)]
struct IntoNumber {
    value: i32,
}

impl From<IntoNumber> for FromNumber {
    fn from(item: IntoNumber) -> Self {
        FromNumber {
            value: item.value * 2,
        }
    }
}

//The Into trait is simply the reciprocal of the From trait.
//That is, if you have implemented the From trait for your type,
//Into will call it when necessary.
// 在代码中只要实现了from,对应的对象就可以进行调用into。
// into 的代码实现，最后还是会调用from。
// impl<T, U> Into<U> for T
// where
//     U: From<T>,
// {
//     fn into(self) -> U {
//         U::from(self)
//     }
// }

fn main() {
    let into_number = IntoNumber{
        value:2,
    };

    println!("IntoNumber is {:?}",into_number);

    // Try removing the type declaration
    let from_number:FromNumber = into_number.into();
    println!("My number is {:?}", from_number);


    // borrow of moved value: `into_number`
    // println!("IntoNumber is {:?}",into_number);

    let into_number_2 = IntoNumber{
        value:3,
    };

    println!("IntoNumber2 is {:?}",into_number_2);

    let from_number2 = FromNumber::from(into_number_2);
    println!("Fromnumber2 is {:?}", from_number2);

    // borrow of moved value: `into_number_2`
    // println!("IntoNumber is {:?}",into_number_2);
}
```

##### Borrow, BorrowMut
对于Borrow和BorrowMut，我的理解就是提供了一共函数式编程的调用方式。(关于函数式编程可以看java的lamda表达式)。

另外borrow和borrowMut在大多是情况下和`&`与`&mut` 等价，但是在String类型上有一些区别。

- 对于一个类型为 T 的值 foo，如果 T 实现了 Borrow<U>，那么，foo 可执行 .borrow() 操作，即 foo.borrow()。操作的结果，我们得到了一个类型为 &U 的新引用。Borrow 的前后类型之间要求必须有**内部等价性**。不具有这个等价性的两个类型之间，不能实现 Borrow。

- BorrowMut<T> 提供了一个方法 .borrow_mut()。它是 Borrow<T> 的可变（mutable）引用版本。对于一个类型为 T 的值 foo，如果 T 实现了 BorrowMut<U>，那么，foo 可执行 .borrow_mut() 操作，即 foo.borrow_mut()。操作的结果我们得到类型为 &mut U 的一个可变（mutable）引用。注：在转换的过程中，foo 会被可变（mutable）借用。

```
use std::rc::Rc;
use std::cell::RefCell;
use std::collections::HashMap;
use std::borrow::Borrow;

#[derive(Debug)]
pub struct Student {
    age: i32,
    name: String,
}

fn main() {
    let a = String::from("test");

    let b = &a ;

    //error[E0283]: type annotations needed for `std::string::String`
    //   --> src/main.rs:17:15
    //    |
    // 13 |     let a = String::from("test");
    //    |         - consider giving `a` a type
    // ...
    // 17 |     let c = a.borrow();
    //    |               ^^^^^^ cannot infer type for struct `std::string::String`
    //    |
    //    = note: cannot resolve `std::string::String: std::borrow::Borrow<_>`
    //
    // error: aborting due to previous error
    let c = a.borrow();
}
```
上述代码编译过程中，会抛出错误。
把`let c = a.borrow`替换为以下语句，强制指定类型，可以通过：
```
let c:&str  = a.borrow();
```
##### asRef,asMut
AsRef 提供了一个方法 .as_ref()。

对于一个类型为 T 的对象 foo，如果 T 实现了 AsRef<U>，那么，foo 可执行 .as_ref() 操作，即 foo.as_ref()。操作的结果，我们得到了一个类型为 &U 的新引用。
- 与 Into<T> 不同的是，AsRef<T> 只是类型转换，foo 对象本身没有被消耗；
- T: AsRef<U> 中的 T，可以接受 资源拥有者（owned）类型，共享引用（shared referrence）类型 ，可变引用（mutable referrence）类型。
- rust对于内部等价性没有borrow那么严格。

AsMut<T> 提供了一个方法 .as_mut()。它是 AsRef<T> 的可变（mutable）引用版本。

对于一个类型为 T 的对象 foo，如果 T 实现了 AsMut<U>，那么，foo 可执行 .as_mut() 操作，即 foo.as_mut()。操作的结果，我们得到了一个类型为 &mut U 的可变（mutable）引用。注：在转换的过程中，foo 会被可变（mutable）借用。

asRef和asMut的使用，主要是想使用其它类型的方法，但是自己又不提供，所以可以先转换到其它类型的引用上。

asRef和asMut可以自己实现。以[tokio-rs/bytes](https://github.com/tokio-rs/bytes)为例，自定义的类，可以使用&[u8]的方法。
```
impl AsRef<[u8]> for BytesMut {
    #[inline]
    fn as_ref(&self) -> &[u8] {
        self.as_slice()
    }
}

impl AsMut<[u8]> for BytesMut {
    fn as_mut(&mut self) -> &mut [u8] {
        self.as_slice_mut()
    }
}

impl AsRef<[u8]> for Bytes {
    #[inline]
    fn as_ref(&self) -> &[u8] {
        self.as_slice()
    }
}
```
string也可以转换为&PATH的引用，并能够调用&PATH的方法：
```
#[stable(feature = "rust1", since = "1.0.0")]
impl AsRef<Path> for String {
    fn as_ref(&self) -> &Path {
        Path::new(self)
    }
}
```
例子如下：
```
use std::path::Path;

fn main() {
    let a = String::from("/root/test.txt");

    let path:&Path = a.as_ref();
    println!("{:?}",path.file_name().unwrap());
}
```
运行结果：
```
"test.txt"
```

##### ToOwned
ToOwned 为 Clone 的普适版本。它提供了 .to_owned() 方法，用于类型转换。

有些实现了 Clone 的类型 T 可以从引用状态实例 &T 通过 .clone() 方法，生成具有所有权的 T 的实例。但是它只能由 &T 生成 T。而对于其它形式的引用，Clone 就无能为力了。

而 ToOwned trait 能够从任意引用类型实例，生成具有所有权的类型实例。
![Screenshot from 2020-06-06 16-58-16.png](https://upload-images.jianshu.io/upload_images/6907217-5bb603fc82172f50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
例子中的代码，如下：
```
use std::path::Path;
use std::borrow::ToOwned;

fn main() {
    let a = "to_owned";
    let b = a.clone();

    println!("{:?}",a);

    let mut c:String = a.to_owned();
    c.push_str(" test");
    println!("{:?}",c)
}
```
运行结果：
```
"to_owned"
"to_owned test"
```
to_owned一般比较很少用，即使在[tokio-rs/hyper](https://github.com/hyperium/hyper)中都基本没怎么用到。

### 7. 指针
Rust 中的指针大体可以分为以下三种：

- 引用 references
- 智能指针 smart pointers
- 裸指针 raw pointers

#### 引用
就是直接对一个变量执行 &、&mut 操作，永远不会为 null。
借用与引用是一种相辅相成的关系，若B是对A的引用，也可称之为B借用了A。
引用按照数据结构可以分为两类。
- 简单引用。其占用的大小与 usize 一致。
- 切片。切片( &[T] )。 T 为切片内元素的类型。切片类型存有： "地址(ptr) + 长度(len) "两个字段。
#### 智能指针
智能指针的概念起源于C++，智能指针是一类数据结构，他们的表现类似指针，但是拥有额外的元数据和功能。
在Rust中，引用和智能指针的一个的区别是引用是一类只借用数据的指针；相反，在大部分情况下，智能指针拥有他们指向的数据。Rust标准库中不同的智能指针提供了比引用更丰富的功能：
- Box<T>，用于在堆上分配数据。
- Rc<T>，一个引用计数类型，其数据可以有多个所有者。
- Ref<T> 和 RefMut<T>，通过RefCell<T>访问，一个在运行时而不是在编译时执行借用规则的类型。
每一个智能指针的实现都不大一样，这里就不再展开叙述。

##### Box指针

在Rust中，所有值默认都是栈上分配。通过创建Box<T>，可以把值装箱，使它在堆上分配。Box<T>类型是一个智能指针，因为它实现了Dereftrait，它允许Box<T>值被当作引用对待。当Box<T>值离开作用域时，由于它实现了Droptrait，首先删除其指向的堆数据，然后删除自身。
Box<T>是堆上分配的指针类型，称为“装箱”（boxed），其指针本身在栈，指向的数据在堆，在Rust中提供了最简单的堆分配类型。使用Box<T>的情况：

- 递归类型和trait对象。Rust需要在编译时知道一个类型占用多少空间，Box<T>的大小是已知的。
例子如下(merkle.rs中的tree的定义)：
```
/// Binary Tree where leaves hold a stand-alone value.
#[derive(Clone, Debug, PartialEq, Eq, PartialOrd, Ord, Hash)]
pub enum Tree<T> {
    Empty {
        hash: Vec<u8>,
    },

    Leaf {
        hash: Vec<u8>,
        value: T,
    },

    Node {
        hash: Vec<u8>,
        left: Box<Tree<T>>,
        right: Box<Tree<T>>,
    },
}
```
##### Rc和Arc(具体遇到再详看)
对一个资源，同一时刻，有且只有一个所有权拥有者。Rc 和 Arc 使用引用计数的方法，让程序在同一时刻，实现同一资源的多个所有权拥有者，多个拥有者共享资源。

Rc 用于同一线程内部，通过 use std::rc::Rc 来引入。它有以下几个特点：

- 用 Rc 包装起来的类型对象，是 immutable 的，即 不可变的。即你无法修改 Rc<T> 中的 T 对象，只能读；
- 一旦最后一个拥有者消失，则资源会被自动回收，这个生命周期是在编译期就确定下来的；
- Rc 只能用于同一线程内部，不能用于线程之间的对象共享（不能跨线程传递）；
- Rc 实际上是一个指针，它不影响包裹对象的方法调用形式（即不存在先解开包裹再调用值这一说）。

Arc 是原子引用计数，是 Rc 的多线程版本。Arc 通过 std::sync::Arc 引入。它的特点：
- Arc 可跨线程传递，用于跨线程共享一个对象；
- 用 Arc 包裹起来的类型对象，对可变性没有要求；
- 一旦最后一个拥有者消失，则资源会被自动回收，这个生命周期是在编译期就确定下来的；
- Arc 实际上是一个指针，它不影响包裹对象的方法调用形式（即不存在先解开包裹再调用值这一说）；
- Arc 对于多线程的共享状态几乎是必须的（减少复制，提高性能）。


#### Cell和RefCell
前面我们提到，Rust 通过其所有权机制，严格控制拥有和借用关系，来保证程序的安全，并且这种安全是在编译期可计算、可预测的。但是这种严格的控制，有时也会带来灵活性的丧失，有的场景下甚至还满足不了需求。

因此，Rust 标准库中，设计了这样一个系统的组件：Cell, RefCell，它们弥补了 Rust 所有权机制在灵活性上和某些场景下的不足。

具体是因为，它们提供了 内部可变性（相对于标准的 继承可变性 来讲的）。通常，我们要修改一个对象，必须
- 成为它的拥有者，并且声明 mut；
- 或 以 &mut 的形式，借用；
而通过 Cell, RefCell，我们可以在需要的时候，就可以修改里面的对象。而不受编译期静态借用规则束缚。

Cell 有如下特点：
- Cell<T> 只能用于 T 实现了 Copy 的情况；

相对于 Cell 只能包裹实现了 Copy 的类型，RefCell 用于更普遍的情况（其它情况都用 RefCell）。

相对于标准情况的 静态借用，RefCell 实现了 运行时借用，这个借用是临时的。这意味着，编译器对 RefCell 中的内容，不会做静态借用检查，也意味着，**出了什么问题，用户自己负责**。

RefCell 的特点：
- 在不确定一个对象是否实现了 Copy 时，直接选 RefCell；
- 如果被包裹对象，同时被可变借用了两次，则会导致线程崩溃。所以需要用户自行判断；
- RefCell 只能用于线程内部，不能跨线程；
- RefCell 常常与 Rc 配合使用（都是单线程内部使用）；

Rc是不可变的，RefCell不去做静态检查，提供内部可变性。他俩的组合：
```
use std::collections::HashMap;
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let shared_map: Rc<RefCell<_>> = Rc::new(RefCell::new(HashMap::new()));
    shared_map.borrow_mut().insert("africa", 92388);
    shared_map.borrow_mut().insert("kyoto", 11837);
    shared_map.borrow_mut().insert("piccadilly", 11826);
    shared_map.borrow_mut().insert("marbles", 38);
}
```
如无必要，不要频繁使用。

#### 裸指针
类似 C 语言里面的指针，可以为 null ！

**创建裸指针，是 safe 的，读写裸指针，才需要 unsafe ！**

裸指针又可以分为可变、不可变，分别写作 *mut T 和 *const T

这里的星号不是解引用运算符，它是类型名称的一部分。

这里的 T 表示指针指向的具体类型，裸指针本身的的类型大小与 usize 一致。

它允许别名，允许用来写共享所有权的类型，甚至是内存安全的共享内存类型如：Rc<T>和Arc<T>，但是赋予你更多权利的同时意味着你需要担当更多的责任：

- 不能保证指向有效的内存，甚至不能保证是非空的
- 是普通旧式类型，也就是说，它不移动所有权，因此Rust编译器不能保证不出像释放后使用这种bug
- 缺少任何形式的生命周期，不像&，因此编译器不能判断出悬垂指针
- 除了不允许直接通过*const T改变外，没有别名或可变性的保障
裸指针使用范例：
```
struct Student {
    name: String,
    age: i32,
}
fn main() {

    let a = 1;
    let b = &a as *const i32;

    let mut x = 2;
    let y = &mut x as *mut i32;

    unsafe{
        println!("{:?}",*y);
        *y = *y + 5;
        println!("{:?}",*y);
    }
}
```
如无必要，尽可能少使用裸指针，引入不安全因素。

### 8. 模式
在rust中，match有两个主要作用，一是相当于其它语言的switch语句进行模式匹配，另一点是解构数据。
##### 模式匹配
```
enum Direction {
    East,
    West,
    North,
    South,
}

fn main() {
    let dire = Direction::South;
    match dire {
        Direction::East => println!("East"),
        Direction::North | Direction::South => {
            println!("South or North");
        },
        _ => println!("West"),
    };
}
```
这是一个没什么实际意义的程序，但是能清楚的表达出match的用法。看到这里，你肯定能想起一个常见的控制语句——switch。没错，match可以起到和switch相同的作用。不过有几点需要注意：
- match所罗列的匹配，必须穷举出其所有可能。当然，你也可以用 _ 这个符号来代表其余的所有可能性情况，就类似于switch中的default语句。
- match的每一个分支都必须是一个表达式，并且，除非一个分支一定会触发panic，这些分支的所有表达式的最终返回值类型必须相同。可以把match整体视为一个表达式，既然是一个表达式，那么就一定能求得它的结果。

##### 解构数据
match还有一个非常重要的作用就是对现有的数据结构进行解构，轻易的可以拿出其中的数据部分来。
```
enum Action {
    Say(String),
    MoveTo(i32, i32),
    ChangeColorRGB(u16, u16, u16),
}

fn main() {
    let action = Action::Say("Hello Rust".to_string());
    match action {
        Action::Say(s) => {
            println!("{}", s);
        },
        Action::MoveTo(x, y) => {
            println!("point from (0, 0) move to ({}, {})", x, y);
        },
        Action::ChangeColorRGB(r, g, _) => {
            println!("change color into '(r:{}, g:{}, b:0)', 'b' has been ignored",
                r, g,
            );
        }
    }
}
```

##### ref 和ref mut
前面我们了解到，当被模式匹配命中的时候，未实现Copy的类型会被默认的move掉，因此，原owner就不再持有其所有权。

但是有些时候，我们只想要从中拿到一个变量的（可变）引用，而不想将其move出作用域，怎么做呢？答：用ref或者ref mut。
```
fn main() {
    let mut x = String::from("match test");

    // It will show the following error:
    // borrow of moved value: `x`
    // match x{
    //     mr=> println!("{:?}",mr),
    // }
    // println!("{:?}",x);


    println!("{:?}", x);
    match x {
        ref mut mr => {
            println!("mut ref before:{}", mr);
            mr.push_str(" end");
            println!("mut ref end:{}", mr);
        },
    }
    println!("{:?}", x);

    // 当然了……在let表达式里也能用
    // let ref mut mrx = x;
}

```
### 9. FFI
FFI是用来与其它语言交互的接口，在有些语言里面称为语言绑定(language bindings).Java 里面一般称为 JNI(Java Native Interface) 或 JNA(Java Native Access)。

由于现实中很多程序是由不同编程语言写的，必然会涉及到跨语言调用，比如 A 语言写的函数如果想在 B 语言里面调用，这时一般有两种解决方案：一种是将函数做成一个服务，通过进程间通信IPC或网络协议通信RPC,RESTFUL等)；另一种就是直接通过 FFI 调用。前者需要至少两个独立的进程才能实现，而后者直接将其它语言的接口内嵌到本语言中，所以调用效率比前者高。

如果在SGX开发，由于要调用intel-sgx,所以必须了解ffi基本知识。

ffi分为两种，一种是调用其它语言的接口，一种是制作语言接口供其它语言调用。

#### rust调用c语言
rust调用c语言主要分为以下四个过程：
- 引入libc库和c库
- 声明你的ffi函数,声明调用那个c函数
- 调用ffi函数
- 封装unsafe，暴露安全接口
##### 引入libc库
由于cffi的数据类型与rust不完全相同，我们需要引入libc库来表达对应ffi函数中的类型。

在Cargo.toml中添加以下行:
```
[dependencies]
libc = "0.2.9"
```
在你的rs文件中引入库:
```
extern crate libc
```
##### 声明你的ffi函数，声明调用哪个c函数
就像c语言需要#include声明了对应函数的头文件一样，rust中调用ffi也需要对对应函数进行声明。
```
use libc::c_int;
use libc::c_void;
use libc::size_t;

#[link(name = "yourlib")]
extern {
    fn your_func(arg1: c_int, arg2: *mut c_void) -> size_t; // 声明ffi函数
    fn your_func2(arg1: c_int, arg2: *mut c_void) -> size_t;
    static ffi_global: c_int; // 声明ffi全局变量
}
```
声明一个ffi库需要一个标记有#[link(name = "yourlib")]的extern块。name为对应的库(so/dll/dylib/a)的名字。 如：如果你需要snappy库(libsnappy.so/libsnappy.dll/libsnappy.dylib/libsnappy.a), 则对应的name为snappy。 在一个extern块中你可以声明任意多的函数和变量。
##### 调用ffi函数
声明完成后就可以进行调用了。 由于此函数来自外部的c库，所以rust并不能保证该函数的安全性。因此，调用任何一个ffi函数需要一个unsafe块。
```
let result: size_t = unsafe {
    your_func(1 as c_int, Box::into_raw(Box::new(3)) as *mut c_void)
};
```
##### 封装unsafe，暴露安全接口
作为一个库作者，对外暴露不安全接口是一种非常不合格的做法。在做c库的rust binding时，我们做的最多的将是将不安全的c接口封装成一个安全接口。 通常做法是：在一个叫ffi.rs之类的文件中写上所有的extern块用以声明ffi函数。在一个叫wrapper.rs之类的文件中进行包装：
```
// ffi.rs(ffi　声明)
#[link(name = "yourlib")]
extern {
    fn your_func(arg1: c_int, arg2: *mut c_void) -> size_t;
}
```
wrapper如下：
```
// wrapper.rs
fn your_func_wrapper(arg1: i32, arg2: &mut i32) -> isize {
    unsafe { your_func(1 as c_int, Box::into_raw(Box::new(3)) as *mut c_void) } as isize
}
```
更多细节查看：[rust调用ffi函数](https://rustcc.gitbooks.io/rustprimer/content/ffi/calling-ffi-function.html)


#### 其他语言调用rust语言编译的库

为了能让rust的函数通过ffi被调用，需要加上extern "C"对函数进行修饰。

但由于rust支持重载，所以函数名会被编译器进行混淆，就像c++一样。因此当你的函数被编译完毕后，函数名会带上一串表明函数签名的字符串。

比如：fn test() {}会变成_ZN4test20hf06ae59e934e5641haaE. 这样的函数名为ffi调用带来了困难，因此，rust提供了#[no_mangle]属性为函数修饰。 对于带有#[no_mangle]属性的函数，rust编译器不会为它进行函数名混淆。如：

demo:
```

#[derive(Debug)]
struct Foo<T> {
  t: T
}

#[no_mangle]
extern "C" fn new_foo_vec() -> *const c_void {
    Box::into_raw(Box::new(Foo {t: vec![1,2,3]})) as *const c_void
}

```
将上述的代码编译成相应的lib，就可以被其他语言调用。

具体详查：[将Rust编译成库](https://rustcc.gitbooks.io/rustprimer/content/ffi/compiling-rust-to-lib.html)



### 10.多线程编程模型，锁等（在SGX的开发中基本不会用到）
后面再根据需要进行补充。
- 线程(thread::spawn)
- 消息传递(channel　推荐)
- 共享内存
- 同步(主要是原子锁，智能指针Mutex,RWLock)
- 与并发相关的trait。(send和sync)
- 智能指针Arc在并发中的使用。

### 11.作用域
关于借用的规则，在RustPrimer中有做以下的说明：
```
- 同一时刻，最多只有一个可变借用（&mut T），或者2。
- 同一时刻，可有0个或多个不可变借用（&T）但不能有任何可变借用。
- 借用在离开作用域后释放。
- 在可变借用释放前不可访问源变量。
```
下面举两个例子说明其中的疑惑之处：
```
#[derive(Debug)]
pub struct Student {
    age: i32,
    name: String,
}

fn main() {
    let mut student = Student {
        age: 12,
        name: String::from("jojo"),
    };

    let ptr = &mut student;
    ptr.age = 3;
    println!("student ptr: {:?}", ptr);

    let ptr2 = &mut student;
    ptr2.age = 5;
    println!("student ptr2: {:?}", ptr2);

    println!("student struct: {:?}", student);
}
```
上述程序编译运行没有问题，运行结果：
```
student ptr: Student { age: 3, name: "jojo" }
student ptr2: Student { age: 5, name: "jojo" }
student struct: Student { age: 5, name: "jojo" }
```

下面一段程序会编译报错：
```
#[derive(Debug)]
pub struct Student {
    age: i32,
    name: String,
}

fn main() {
    let mut student = Student {
        age: 12,
        name: String::from("jojo"),
    };

    let ptr = &mut student;
    let ptr2 = &mut student;


    ptr.age = 3;
    println!("student ptr: {:?}", ptr);

    ptr2.age = 5;
    println!("student ptr2: {:?}", ptr2);

    println!("student struct: {:?}", student);
}
```
运行结果：
```
error[E0499]: cannot borrow `student` as mutable more than once at a time
  --> src/main.rs:14:16
   |
13 |     let ptr = &mut student;
   |               ------------ first mutable borrow occurs here
14 |     let ptr2 = &mut student;
   |                ^^^^^^^^^^^^ second mutable borrow occurs here
...
17 |     ptr.age = 3;
   |     ----------- first borrow later used here

error: aborting due to previous error

For more information about this error, try `rustc --explain E0499`.
error: could not compile `high_level_trait`.
```
因为一行语句的位置的移动，导致无法编译通过。
对于rust的`作用域`和`所有权规则中的同一刻`，还需要之后再研究一下。


参考文档：
[https://legacy.gitbook.com/book/rustcc/rustprimer](https://legacy.gitbook.com/book/rustcc/rustprimer)
[https://kaisery.gitbooks.io/trpl-zh-cn/content/ch19-03-advanced-traits.html](https://kaisery.gitbooks.io/trpl-zh-cn/content/ch19-03-advanced-traits.html)
