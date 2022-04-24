# C++ const 梳理

整理一下自己在学习和实际使用过程中掌握的对C++ 中`const`关键字的使用的相关知识。

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
  * 关于何时将其保存在符号表中，何时将其保存在内存中，后面会进行详细说明



## 使用方法

### 1.声明普通常量











## 参考资料

* https://wenku.baidu.com/view/53a778b4d9ef5ef7ba0d4a7302768e9951e76e2a.html?qq-pf-to=pcqq.c2c
* https://blog.csdn.net/lovekatherine/article/details/1644971
* https://blog.csdn.net/Tonny0832/article/details/12558493?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-0.pc_relevant_antiscanv2&spm=1001.2101.3001.4242.1&utm_relevant_index=3
* https://blog.csdn.net/hziee_/article/details/1750733
* https://blog.csdn.net/silently_frog/article/details/96737764

