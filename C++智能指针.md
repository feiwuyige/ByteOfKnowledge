# C++ 智能指针

1. 浅拷贝和深拷贝：

   * 浅拷贝：源对象与拷贝对象共用同一份实体
   * 深拷贝：源对象与拷贝对象是两个相互独立的实体

2. 智能指针的构造方法：

   * 使用 `make_shared` 函数模板进行构造，参数为要构造对象的构造函数所需要的参数，如：
     ``` cpp
     //make_shared<类型>(构造函数参数列表)
     auto p = make_shared<int>(5);
     ```

   * 通过 `new` 生成的内置指针初始化生成  `shared_ptr`，模板类，其构造函数是 `explicit` 的，即禁止进行隐式转化，所以我们必须使用直接初始化来初始化一个 智能指针：
     ```cpp
     auto p = shared_ptr<std::string> (new std::string("hello, world!")); //正确
     
     shard_ptr<int> p = new int(5); //错误，拷贝初始化，会发生隐式转化
     ```

     **如果通过内置指针来生成智能指针，一定要记得不要去手动回收内置指针，一旦我们将一个内置指针绑定到一个智能指针以后，内存管理的任务就交给了智能指针，我们应当使用智能指针来访问相关内存， 而不再使用内置指针**：

     ```cpp
     void process(shared_ptr<int> psint)
     {
         cout << "psint data is " << *psint << endl;
     }
     
     int main()
     {
         int *p = new int(5);
         process(shared_ptr<int>(p));
         //危险，p已经被释放，会造成崩溃或者逻辑错误
         cout << "p data is " << *p << endl;
         return 0;
     }
     ```

     ![image-20250103100109220](img\image-20250103100109220.png)

     因为`p`构造为`shared_ptr`，那么它的回收就交给了`shared_ptr`，而`shared_ptr`是`process`的形参，形参在`process`运行结束会释放，那么 `psint 和 p`  也被回收，之后再访问`p`会产生逻辑错误，所以打印了一个非法内存的数值。

   * 使用拷贝构造函数，即使用一个智能指针来赋值另一个智能指针，此时两个智能指针会共享一个底层内置指针，从而两个智能指针的引用计数都会＋1：
     ```cpp
     shared_ptr<std::string> p1 = shared_ptr<std::string> (new std::string("hello, world!"));
     shared_ptr<std::string> p2(p1);
     ```

3. 在构造智能指针的时候，可以自定义自己的删除方法来替换智能指针自己的 `delete` 操作。

4. 使用 `get`  方法，可以得到智能指针底层的内置指针对象，但要注意，使用 `get` 返回这个内置指针以后，我们不能手动删除该指针，否则会出现错误：
   ```cpp
   void bad_use_sharedptr()
   {
       shared_ptr<int> p(new int(5));
       //通过p获取内置指针q
       //注意q此时被p绑定，不要手动delete q
       int *q = p.get();
       {
           //两个独立的shared_ptr m和p都绑定q
           auto m = shared_ptr<int>(q);
       }
   
       //上述}结束则m被回收，其绑定的q也被回收
       //此时使用q是非法操作，崩溃或者逻辑错误
       cout << "q data is " << *q << endl;
   }
   ```

   ![image-20250103100758960](img\image-20250103100758960.png)

   **永远不要用 `get` 方法去初始化另一个智能指针，或给另一个智能指针赋值**。

   ```cpp
   std::shared_ptr<int> sp1 = std::make_shared<int>(10);
   std::shared_ptr<int> sp2(sp1.get()); // 错误，此时sp1和sp2是两个独立的，他们有自己各自的引用计数
   //当引用计数为0时，sp1 和 sp2 都会去释放内存，造成重复释放的问题。 
   
   void bad_use_sharedptr()
   {
       std::shared_ptr<int> sp1 = std::make_shared<int>(10);
       std::shared_ptr<int> sp2(sp1.get());
       std::shared_ptr<int> sp3 = sp2;
       std::cout << sp1.use_count() << std::endl;
       std::cout << sp2.use_count() << std::endl;
   }
   ```

   ![image-20250103101946300](img\image-20250103101946300.png)

5. `reset` 方法让智能指针重新绑定一块新的内存，即更改内置指针。

6. 有时候我们智能指针绑定的内置指针不是 `new` 分配的，那么我们就要手动定义删除器，否则默认调用 `delete` 运算符就会出错。





ps. 内容主要参考[恋恋风辰](llfc.club)