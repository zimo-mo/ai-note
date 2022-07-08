# The Rust Programming Language

## 1. Common Concepts

### 1.1 变量及可变性

在rust中，变量默认为不可变。当一个值绑定（bound to）到非可变变量后，就不能更改该值。如下所示：

```rust
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

`cargo run`将会提示如下错。不能给不可变变量`x`进行第二次赋值。

```bash
Compiling rust-book v0.1.0 (/home/finance/project/rust-book)
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:6:5
  |
4 |     let x = 5;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
5 |     println!("The value of x is: {}", x);
6 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable

For more information about this error, try `rustc --explain E0384`.
error: could not compile `rust-book` due to previous error
```

如果需要在其他地方修改变量的值，需要在变量名前加`mut`声明。

```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```



**Constants**

和不可变量类似，常量值绑定到常量名之后不允许更改。但是也有如下区别：

-   常量声明采用`const`替代`let`，但不能使用`mut`关键字修饰，并且必须加类型注释

    eg: `const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;`

-   常量可以在任何作用域声明

-   常量值计算只能是常量表达式（非运行时计算结果）



**Shadowing**

在rust编程中，可以声明一个和前面出现相同名称的变量，第一个变量会被该变量遮罩（shadowwed）。

```rust
fn main() {
    let x = 5;
    let x = x + 1;
    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }
    println!("The value of x is: {}", x);
}
```

以上代码中，首先5被绑定到`x`，接下来`x + 1 = 6`被再次绑定到`x`，第一次`x`变量被遮罩；在内置作用域（inner scope），`let x`再次遮罩`x`，并绑定新值`12`。当内置作用域结束，内置作用域的遮罩也结束。`x`重新回到`x=6`。

shadowing与声明为`mut`的变量有如下区别： 

-   shadowing必须使用`let`关键字

-   shadowing使用相同的变量名，但是类型可以不同

    

### 1.2 数据类型（Data Types）

rust作为一种静态类型语言，意味着在编译期就必须知道每个变量的类型。通常编译器会进行类型推导，但是在推到类型不确定的时候，必须加上类型注解。比如：

```rust
#![allow(unused)]
fn main() {
	let guess: u32 = "42".parse().expect("Not a number!");
}
```



#### 标量类型（Scalar Types）

一个标量类型表示一个值。Rust有四种主要的标量类型：`integer,floating-point,numbers,Booleans,and characters.`

**Integer Types**

| Length  | Signed | Unsigned |
| ------- | ------ | -------- |
| 8-bit   | i8     | u8       |
| 16-bit  | i16    | u16      |
| 32-bit  | i32    | u32      |
| 64-bit  | i64    | u64      |
| 128-bit | i128   | u128     |
| arch    | isize  | usize    |

符号整数范围：$[-2^{n-1},2^{n-1} - 1]$.

无符号整数存储范围：$[0, 2^n -1]$.

其中`isize`和`usize`依赖于程序所运行的cpu指令架构，在64-bit architecture上其大小为64bits，在32-bit architecture上其大小为32bits。



#### Floating-Point Types

rust定义两种float点类型： `f32`和`f64`。系统默认`f64`，因为在现代cpu上与`f32`的速度相当，但是精度更高。



#### Boolean Type

`bool`类型有两种可能的值，`true`和`false`，其大小为一个byte.

```rust
fn main() {
    let t = true;
    let f: bool = false; // with explicit type annotation
}
```



#### Character Type

为了区别于字符串，字符类型采用单引号。

```rust
fn main() {
    let c = 'z';
    let z = 'ℤ';
    let heart_eyed_cat = '😻';
}
```

rust字符（char）类型大小为4bytes，表示一个Unicode标量（Unicode Scalar Value）。Unicode标量范围为`U+0000`to`U+D7FF`, `U+E000`to `U+10FFFF`.



### 1.3 复合类型（Compound Types）

符合类型将多种类型的值聚合到一个类型中。rust内置两种主要的复合类型：`tuples`和`arrays`。

####  Tuple Type

`tuples`定义如下，

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

`tuples`一旦申明后，其长度不能变更。



## 2. 所有权（Ownership）

### 2.1 什么是所有权（owership）

所有权(owership)是Rust管理内存的一系列规则。每种编程语言都有管理内存的方式，比如java、go等采用垃圾回收机制不断去判断是否存在不使用的内存；另外一些语言比如`c/c++`需要显示申请和释放内存；`rust`采用第三种方式，通过所有权机制来管理内存。如果程序违反任何所有权的规则，将编译不通过。但是所有权的特性并不会影响到程序的运行时间。



**Owership Rules**

-   Each value in Rust has a variable that’s called its *owner*.
-   There can only be one owner at a time.
-   When the owner goes out of scope, the value will be dropped.



**Variable Scope**

