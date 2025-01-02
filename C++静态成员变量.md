# C++静态成员变量

1. `static`  关键字：

   * 用于函数定义或在代码块之外的变量声明时，修改的是链接属性，从 `external` 修改为 `internal` .
   * 用于代码块内部的变量声明时，修改的时存储类型，从自动变量修改为静态变量。

2. 静态成员变量的使用：

   > The `static` keyword is only used with the declaration of a static member, inside the class definition, but not with the definition of that static member:
   > ```cpp
   > class X { static int n; }; // declaration (uses 'static')
   > int X::n = 1;              // definition (does not use 'static')
   > ```

   所以， `static` 关键字是用于声明的，而非定义。

3. 声明和定义：

   > *Declarations* are how names are introduced (or re-introduced) into the C++ program. Not all declarations actually declare anything, and each kind of entity is declared differently. [Definitions](https://en.cppreference.com/w/cpp/language/definition) are declarations that are sufficient to use the entity identified by the name.

   简单理解，声明是告诉有这个变量，以及这个变量的相关信息；而定义就是指这个变量是可以通过名称来使用了，也就是已经在内存中分配了空间。

   > *Definitions* are [declarations](https://en.cppreference.com/w/cpp/language/declarations) that fully define the entity introduced by the declaration. Every declaration is a definition, except for the following:
   >
   > * A function declaration without a function body（函数原型）
   >
   > * Any declaration with an extern [storage class specifier](https://en.cppreference.com/w/cpp/language/storage_duration) or with a [language linkage](https://en.cppreference.com/w/cpp/language/language_linkage) specifier (such as extern "C") without an initializer
   >
   > * Declaration of a non-inline(since C++17) [static data member](https://en.cppreference.com/w/cpp/language/static) inside a class definition
   >
   > * (deprecated) Namespace scope declaration of a static data member that was defined within the class with the [`constexpr`](https://en.cppreference.com/w/cpp/language/constexpr) specifier:
   >   ```cpp
   >   struct S
   >   {
   >       static constexpr int x = 42; // implicitly inline, defines S::x
   >   };
   >   constexpr int S::x; // declares S::x, not a redefinition
   >   ```
   >
   > * Declaration of a class name (by [forward declaration](https://en.cppreference.com/w/cpp/language/class#Forward_declaration) or by the use of the elaborated type specifier in another declaration)
   >
   > * An [opaque declaration](https://en.cppreference.com/w/cpp/language/enum) of an enumeration:
   >   ```cpp
   >   enum Color : int; // declares, but does not define Color
   >   ```
   >
   > * Declaration of a [template parameter](https://en.cppreference.com/w/cpp/language/template_parameters)
   >
   > * A parameter declaration in a function declaration that isn't a definition
   >
   > * A [typedef](https://en.cppreference.com/w/cpp/language/typedef) declaration
   >
   > * An [alias-declaration](https://en.cppreference.com/w/cpp/language/type_alias):
   >   ```cpp
   >   using S2 = S; // declares, but does not define S2 (S may be incomplete)
   >   ```
   >
   > * A [using-declaration](https://en.cppreference.com/w/cpp/language/using_declaration)
   >
   > * ...

4. **one defination rule**(odr):

   > Only one definition of any variable, function, class type, enumeration type[, concept](https://en.cppreference.com/w/cpp/language/constraints)(since C++20) or template is allowed in any one translation unit (some of these may have multiple declarations, but only one definition is allowed).
   >
   > One and only one definition of every non-[inline](https://en.cppreference.com/w/cpp/language/inline) function or variable that is *odr-used* (see below) is required to appear in the entire program (including any standard and user-defined libraries).
   >
   > For an inline function or inline variable(since C++17), a definition is required in every translation unit where it is *odr-used* ﻿.
   >
   > An object is odr-used if its value is read (unless it is a compile time constant) or written, its address is taken, or a reference is bound to it,
   >
   > A reference is odr-used if it is used and its referent is not known at compile time,
   >
   > A function is odr-used if a function call to it is made or its address is taken.
   >
   > If an entity is odr-used, its definition must exist somewhere in the program; a violation of that is usually a link-time error.

5. 总结：
   对于类中的所有成员变量，都是声明，只有在创建对象的时候才会去开辟地址空间，也就是定义。在类中使用  `static` 关键字来声明一个变量，只是修改了这个变量的存储类型，也就是由自动变量变为静态变量，那么编译器就会在适当的存储区为其开辟地址空间。
   对于类类型而言，静态成员变量、静态成员函数属于类本身，而非对象，所以静态成员函数没有隐含的 `this` 指针，而静态成员变量则需要用户自己提供定义，考虑以下情况：

   ```cpp
   //test.h
   struct X{
       static int a; //声明
   };
   int X::a = 10; //定义
   //test.cpp
   #include "test.h"
   #include <iostream>
   int main(){
       std::cout << X::a << std::endl;
       return 0;
   }
   ```

   此时是正确情况，若没有定义，即：
   ```cpp
   //test.h
   struct X{
       static int a; //声明
   };
   //int X::a = 10; //定义
   //test.cpp
   #include "test.h"
   #include <iostream>
   int main(){
       std::cout << X::a << std::endl;
       return 0;
   }
   ```

   出现错误，找不到 `X::a` 的定义：
   ![image-20250102134035445](C:\Users\26259\AppData\Roaming\Typora\typora-user-images\image-20250102134035445.png)

   现在我们假设还有一个头文件引用了 `test.h`，即：
   ```cpp
   //test.h
   struct X{
       static int a; //声明
   };
   int X::a = 10; //定义
   //error.h
   #include "test.h"
   //test.cpp
   #include "test.h"
   #include <iostream>
   #include "error.h"
   int main(){
       std::cout << X::a << std::endl;
       return 0;
   }
   ```

   因为 `include` 就是简单的复制粘贴，所以肯定会出现重定义问题，结果如下：
   ![image-20250102134544706](C:\Users\26259\AppData\Roaming\Typora\typora-user-images\image-20250102134544706.png)

   所以**对于静态成员变量，最好是在哪个文件中使用，就在那个文件中进行定义**。但是也有一些情况可以直接在类中进行定义:

   * 使用 `const` 关键字，但是必须是整型（const 类型，只读，所以不是odr-used且满足一些条件所以可以有多个定义）

   ```cpp
   //test.h
   class X{
   public:
       static constexpr int a = 10; //定义
   };
   //test.cpp
   #include "test.h"
   #include <iostream>
   int main(){
       std::cout << X::a << std::endl;
       return 0;
   }
   ```

   > If a static data member of integral or enumeration type is declared const (and not volatile), it can be initialized with an [initializer](https://en.cppreference.com/w/cpp/language/initialization) in which every expression is a [constant expression](https://en.cppreference.com/w/cpp/language/constexpr), right inside the class definition

   * 使用 `inline` 关键字(For an inline function or inline variable(since C++17), a definition is required in every translation unit where it is *odr-used* ﻿.)

   ```cpp
   //test.h
   class X{
   public:
       static inline int a = 10; //定义
   };
   //test.cpp
   #include "test.h"
   #include <iostream>
   int main(){
       std::cout << X::a << std::endl;
       return 0;
   }
   ```

   > A static data member may be declared [`inline`](https://en.cppreference.com/w/cpp/language/inline). An inline static data member can be defined in the class definition and may specify an initializer. It does not need an out-of-class definition:
   >
   > ```cpp
   > struct X
   > {
   >     inline static int fully_usable = 1; // No out-of-class definition required, ODR-usable
   >     inline static const std::string class_name{"X"}; // Likewise
   >  
   >     static const int non_addressable = 1; // C.f. non-inline constants, usable
   >                                           // for its value, but not ODR-usable
   >     // static const std::string class_name{"X"}; // Non-integral declaration of this
   >                                                  // form is disallowed entirely
   > };
   > ```

