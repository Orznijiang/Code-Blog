# C++ const 梳理

整理一下自己在学习和实际使用过程中整理的C++ 中`const`关键字的相关知识。

## 简单介绍

`const`是C++的一个关键字，用于对类型进行修饰，说明该类型是一个常类型，不能够被更改，因此一般需要在首次声明时进行初始化。

## 作用及优点

* 定义简单常量

  * example：`const int TEST = 10;`

    * 声明该常量后，后面的代码不能够修改`TEST`的值

  * 宏定义也能够声明常量：`#define TEST 10`，与宏定义相比，`const`常量具有一定的优势

    * `const`常量有数据类型，编译器可以在其声明时对其进行类型安全检查

    * 宏定义只是进行字符替换，无类型安全检查，且无法预先检查到字符替换后的错误

    * `const`常量从汇编的角度看，只是给出了对应的内存地址，而宏定义给出的是立即数。因此，`const`常量在程序运行过程中只有一份拷贝，而宏定义在每次被使用到时都会产生一份拷贝

      ```
      #define PI 3.14 //宏定义
      const double pi = 3.14; //const常量声明，此时并没有为其分配内存（当使用到时才分配）
      ...
      double a = PI; //宏替换，分配一次内存
      double A = PI; //宏替换，再次分配内存
      double b = pi; //此时为pi分配内存
      double B = pi; //不再分配新的内存，const常量只分配一次内存
      ```

* 在函数调用中防止发生不被期望的修改

  ```
  void fun(const int& num){
  	num++; //wrong
  }

* 提高效率
  * 编译器通常不为普通的`const`常量分配存储空间，而是将其保存在符号表中，成为编译期间的常量。这样做的优势是避免了内存的读写，使得效率很高
  * 关于何时将其保存在符号表中，何时将其保存在内存中，后面会进行说明



## 使用方法

### 1.声明普通常量

```
TYPE const name = value;
const TYPE name = value;
```

上面的两种定义形式都是正确的，且作用相同。它表示`const`关键字修饰的类型为`TYPE`的变量`name`的值`value`是不可变的。

### 2.声明指针和引用相关的常量

```
const int * name = &value;
int const * name = &value;
int * const name = &value;
```

当加入指针或者引用后，情况就变成了3种：

* 这里，第一种情况和第二种情况的意义相同。也就是说，无论`const`关键字放在`int`的前面或者后面，都表示这个`int`值不可变，即这个指针指向的值`value`无法被更改

* 而当`const`关键字放在`*`的后面时，表示的意义就不同了。这里表示这个指针本身的值不可被更改，即这个指针指向的地址不可被修改来重新指向别的`int`值

* 引用的情况要更特殊一点，实际上，只存在常量引用，而不存在引用常量，因为引用本身就是变量的别名，是不可更改的。对于上面的语句，引用仅存在前两种情况，即：

  ```
  const int name = 1;
  const int & name1 = name;
  int const & name2 = name;
  ```

  且两种情况表达的含义相同

当然，一条语句可以同时使用多个`const`关键字，如下：

```
const int const * const name = value;
```

这条语句表示`int`值和指针指向的地址都不可变。当前，前面两个`const`关键字的效果是一样的，可以省略一个。

### 3.在函数中使用

#### 修饰形参

* 修饰普通形参

  * 表示该形参在函数内不可变（无意义）

    ```
    void fun(const int num);
    ```

    由于`num`本身只不过是一个拷贝，即使更改了也不会对实参造成影响，更多的意义是为了提醒编程者在写函数的时候不修改这个变量，但实际上意义不大。虽然语法上没有问题，但不建议使用。

* 修饰指针相关形参

  * 表示指针所指内容不可变

    ```
    void fun(const int * num);
    ```

    由于指针直接指向外部的`int`变量，若不希望这个值在函数中被意外修改，则可以用`const`关键字修饰。

  * 表示指针本身不可变（无意义）

    ```
    void fun(int * const num);
    ```

    同普通形参一样，这里指针指向的地址也不过是一个拷贝，修不修改都不会对外部造成影响，因此意义不大

* 修饰引用相关形参

  * 表示被引用对象的值不可被更改

    ```
    void fun(const & int num);
    ```

    在防止被引用对象的值在函数中被更改的同时，通过直接传递地址减少了一次拷贝，增加了效率。对于上面的`int`来说可能印象不大，但对于类等拷贝代价较大的实参较为有用。

#### 修饰返回值

通常情况下，不建议使用`const`关键字修饰的返回值类型。它限制了你不能对函数的返回值进行修改。

* 对于普通变量类型：没有意义。因为函数调用返回时进行一次赋值拷贝，并不会影响函数中的值，相当于白白得到了被限制无法改变的返回值

  ```
  int const fun();

* 对于指针与引用相关的返回值：使用情况较少

  * 无法对返回值进行赋值
  * 当使用返回值为另一个变量赋值时，`const`关键字无意义
  * 多用于操作符的重载函数等不希望返回值被修改的情况
  
  ```
  const int * fun();



### 4.在类中使用

#### 修饰成员变量

* 修饰类的成员变量，表示成员常量，不能被修改

* 成员常量只能够在初始化列表中进行赋值操作

  ```
  class Test{
  	//...
  	const int num;
  	//...
  	Test(int x):num(x){
  		//...
  	}
  }



#### 修饰成员函数

* 修饰类的成员函数，表示这个函数不能改变对象的成员变量，也不能调用类中任何的非`const`成员函数（因为非`const`成员函数可能改变成员变量）

* 对于`const`关键字修饰的类对象、指针或引用，只能够调用类的`const`成员函数。因此，`const`关键字修饰的成员函数的作用就是限制`const`关键字修饰的对象的使用

  ```
  class Test{
  	//...
  	void fun() const; // 添加在函数的末尾
  	//...
  }



#### 修饰类对象、对象指针或对象引用

* 表示该对象（指针指向的对象或引用的对象）为常量，对象中的任何成员不能够被修改
* 该对象不能够调用类中的任何非`const`成员函数（因为非`const`成员函数可能改变成员变量）



## 作用范围

* 当将`const`关键字用于声明某个常量时，该标识符自动具备**internal linkage**属性，即只对相同文件内的函数可见，对其他文件中的函数是不可见的。关于内部链接，可以查阅最下面的参考资料

  ```
  // a.cpp
  // ...
  const int num = 0;
  // ...
  ```

  ```
  // b.cpp
  // ...
  const int num = 1;
  // ...
  ```

  上面的两个`cpp`文件声明了同名的`int`类型常量，但是常量的值不同。若将这两个文件放在一起编译，并不会产生命名冲突，因为这两个常量的作用范围仅在当前的`cpp`文件内（内部链接），外部的文件一般情况下看不到另外的常量，即使它们拥有相同的常量名。

* 稍微复杂的情况：

  ```
  // a.cpp
  // ...
  const char * name = "hi";
  // ...
  ```

  ```
  // b.cpp
  // ...
  const char * name = "hi";
  // ...
  ```

  此时，若将两个文件一起编译，则会出现重复定义的错误。根据上面讲的`const`关键字的修饰对象，此时`const`关键字修饰的是char，表示`"hi"`这个字符串常量。而`name`指针则被认为是非常量，默认情况下，具有**external linkage**属性，即可以被其它源文件使用，在整个程序内有效，且全局只允许有一个，因此导致了重定义。



## 内存分配

* 一般情况下，`const`常量位于符号表中，不会为其分配内存空间
* 当使用常量时，从符号表中取值
* 当进行需要得到`const`常量地址的操作时，才会为其分配内存空间（且只分配一次）
* 用`extern`关键字修饰，成为全局常量时，会为其分配空间









## 参考资料

* https://wenku.baidu.com/view/53a778b4d9ef5ef7ba0d4a7302768e9951e76e2a.html?qq-pf-to=pcqq.c2c
* https://blog.csdn.net/lovekatherine/article/details/1644971
* https://blog.csdn.net/hziee_/article/details/1750733
* https://blog.csdn.net/silently_frog/article/details/96737764
* https://blog.csdn.net/mznewfacer/article/details/6896946?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0.pc_relevant_default&spm=1001.2101.3001.4242.1&utm_relevant_index=3
* https://blog.csdn.net/qq_35599308/article/details/86192311?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2.pc_relevant_paycolumn_v3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2.pc_relevant_paycolumn_v3&utm_relevant_index=5
* https://blog.csdn.net/d137578736/article/details/79033983?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.pc_relevant_aa&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.pc_relevant_aa&utm_relevant_index=1
* 内部链接与外部链接：https://blog.csdn.net/Tonny0832/article/details/12558493?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-0.pc_relevant_antiscanv2&spm=1001.2101.3001.4242.1&utm_relevant_index=3

