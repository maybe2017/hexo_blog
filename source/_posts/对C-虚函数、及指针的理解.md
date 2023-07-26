---
title: 对C++虚函数、及指针的理解
date: 2023-07-26 11:14:44
comments: true
layout: post
toc: true
tags:
  - C/C++
categories: 开发语言基础
---

### 一、不同类型指针强转、解引用的问题

1. 下面的示例，基类 Base 先注释掉虚析构，原因后续说明，子类 Child 创建的对象中，只拥有两个 int 变量、及一个虚函数表指针；

   基类 Base：

   ```c++
   class Base{
   private:
       int i;
   public:
       Base(){
           std::cout << "Base构造被调用" << std::endl;
       }
       // 纯虚析构(强制子类重写析构) virtual ~Base() = 0;
   //    virtual ~Base(){
   //        std::cout << "Base虚析构被调用" << std::endl;
   //    };

       // 虚函数
       virtual void print() {
           std::cout << "Base print被调用" << std::endl;
       }

       // 纯虚函数
       virtual void fun() = 0;

       // final 不允许子类重写
       virtual void funcOfFinal() final {
           std::cout << "Base funcOfFinal被调用" << std::endl;
       };
   };
   ```

   子类 Child2：

   ```java
   class Child2 : public Base {
   private:
       int self;
   public:
       Child2(){
           std::cout << "Child2构造被调用" << std::endl;
       }
   //    ~Child2(){
   //        std::cout << "Child2析构被调用" << std::endl;
   //    }
       virtual void print() override {
           std::cout << "Child2 print被调用" << std::endl;
       }
       virtual void fun() {
           std::cout << "Child2 fun被调用" << std::endl;
       }
   };
   ```

   主函数：

   ```c++
   // 测试通过对象拿到虚函数表指针、通过表指针去调用虚函数
   void testGetVfptrAndInvoke() {
     std::cout << "-------测试通过对象拿到虚函数表指针、通过表指针去调用虚函数--------" << std::endl;
     Child2 a;
     // 将(&a)强制类型转换为(u64 *)，来把从&a开始的8个字节当作一个整体，这8个字节就是vfptr存储的值
     u64 *addrOfA = (u64 *)&a;
     // 然后对其进行解引用，就相当于取出这8个字节中的数据，8个字节编码后的值就是虚函数表的首地址 (等同u64 *arr = (u64*)*addrOfA;)
     long long  vaule = *addrOfA;
     // 将数值转为地址(指针)类型，此时就拿到了 指向虚函数表 的指针
     u64* vfptr = (u64*)vaule;

     // 用指针进行虚函数的调用
     std::cout << "-------虚函数调用开始--------" << std::endl;
     std::cout << "--> 调用vfptr[0]: ";
     auto f0 = (VIRTUAL_FUNC)vfptr[0];
     f0();

     std::cout << "--> 调用vfptr[1]: ";
     auto f1 = (VIRTUAL_FUNC)vfptr[1];
     f1();

     std::cout << "--> 调用vfptr[2]: ";
     auto f2 = (VIRTUAL_FUNC)vfptr[2];
     f2();
     std::cout << "-------虚函数调用结束--------" << std::endl;
   }


   int main() {
     testGetVfptrAndInvoke();
   }
   ```

   结果输出：

   ```c++
   /Users/mayu/Documents/workspace/c_codes/cmake-build-debug/InheritanceUse
   -------测试通过对象拿到虚函数表指针、通过表指针去调用虚函数--------
   Base构造被调用
   Child2构造被调用
   -------虚函数调用开始--------
   --> 调用vfptr[0]: Child2 print被调用
   --> 调用vfptr[1]: Child2 fun被调用
   --> 调用vfptr[2]: Base funcOfFinal被调用
   -------虚函数调用结束--------
   Child2析构被调用

   ```

2. 起初不能理解 main 函数中对 addrOfA 指针解引用后，即\*addrOfA 到底是个什么东西？后面经过查阅资料，考虑以下性质：

   - **单继承时，虚函数表指针通常存储在类对象“内存布局”的最前面**；
   - 虚函数表实质上是一个`函数指针`的数组；
   - 一个有虚函数（`无论是继承得到的虚函数还是自身有的`）的类，该类的所有对象，都共用一份虚函数表**；**
   - 每个对象有一套（`这里用套而不用个，是因为多重继承时，可能有多个指针组成的一套`）“虚函数表指针”，指向该虚函数表。
   - 需要注意的一点是，<span style="color:red;">父类的构造与析构子类虽然规定不能继承，但是虚析构也认为是虚函数，父类如果有虚析构，那么该函数也会出现在子类的虚函数表中（第一个位置），且满足子类重写覆盖函数地址的规则</span>。

   对 a 对象内存分布画了个简图，如下：

![c++获取虚函数表地址](https://p.ipic.vip/g5y8nr.png)

3. 解引用的本质就是编译器根据指针所指的类型，然后从指针所指向的内存连续取 N 个字节，然后将这 N 个字节按照指针的类型去解释。比如 int \*型指针，那么这里 N 就是 4，然后按照 int 的编码方式去解释数字；（这里也就解释了上面代码示例中，疑惑为什么用 A 类型的指针解引用后，得到的是 a 对象，而不是一个 long long 的值）。
4. 输出结果中，只有子类析构被调用，是因为父类没有虚析构，一旦打开 Base 类注释，放开父类的虚析构，那么正常情况下，子类析构执行结束前，就会默认调用父类析构。<span style="color:red;">之所以注释掉父类虚析构，是因为一旦放开，那么在子类的虚函数表中，第一个元素的位置就存储了父类析构函数的地址（子类暂不覆写）</span>，那么一旦这个位置的虚析构函数被调用后，后续对虚函数表中其他索引位置调用时，程序运行时就会报异常（原因暂未研究）。

### 二、一些基础的理解

1. 指针和指针变量的理解：指针就是一个地址，一个常量；指针变量，就是存放内存地址的变量，存放的地址可以变；只是我们平时总是将两者混着称呼。

2. 变量名：本质是变量地址的符号化。可以认为编译器会自动维护一个映射，将我们程序中的变量名转换为变量所对应的地址，然后再对这个地址去进行读写。

3. 对指针解引用时，它产生的结果也是既可以作为左值也可以作为右值；作为左值表示的是<span style="color:red;">指针中存储的地址对应的那块内存空间</span>，作为右值时，表示的是<span style="color:red;">指针中存储的地址的那块空间中的内容</span>。

4. 对指针解引用会得到所指的对象，如果给解引用的结果赋值（作为左值使用），就是给指针所指的对象赋值。

5. void* 类型的指针是不能解引用和进行加减操作的，因为 void* 类型的指针指向 void 类型，void 类型是没有意义的；void \* 类型的指针编译器是不知道它到底指向的是 int、double、或者是一个结构体，所以编译器没法对 void 型指针解引用。。

6. https://developer.huawei.com/consumer/cn/forum/topic/0204413352795160424
