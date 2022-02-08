### 1 类型推导

#### 01. 理解模板类型推导

```c++
template<typename T>
void f(ParamType param);

f(expr);
```

- 在编译期，编译器会通过 expr 推导两个类型：T 的类型和 ParamType 的类型，这两个类型往往不一样，因为 ParamType 经常会包含一些修饰，如 const、引用等。

- T 类型的推导依赖于 expr 的类型和 ParamType 的形式，分三种情况：

  1. ParamType 具有指针或引用，但非万能引用。若 expr 具有引用类型，先将引用部分忽略（& 或 *），然后对 expr 的类型和 ParamType 的类型执行模式匹配来决定 T 的类型：

     ```c++
     template<typename T>
     void f1(T& param)
     
     template<typename T>
     void f2(const T& param)
     
     int x1;
     const int x2;
     const int& x3;
     
     f1(x1);		// T 为 int，param 为 int&
     f1(x2);		// T 为 const int，param 为 const int&
     f1(x3);		// T 为 const int，param 为 const int&
     
     f2(x1);		// T 为 int，param 为 const int&
     f2(x2);		// T 为 int，param 为 const int&
     f2(x3);		// T 为 int，param 为 const int&
     ```

  2. ParamType 是一个万能引用。若 expr 是左值，则 T 和 param 都会被推导为左值引用，若为右值则和上面的规则差不多。

     ```c++
     template<typename T>
     void f(T&& param)
     
     int x1;
     const int x2;
     const int& x3;
     
     f(x1);		// x1 为左值，T 为 int&，param 为 int&
     f(x2);		// x2 为左值，T 为 const int&，param 为 const int&
     f(x3);		// x3 为左值，T 为 const int&，param 为 const int&
     
     f(0);		// 0 为右值，T 为 int，param 为 int&&
     ```

  3. ParamType 非指针也非引用。此时是按值传递，不管传入什么 param 都将是它的一个副本，即一个全新的对象，所以 expr 类型具有引用类型、const、volatile 都会被忽略。

     ```c++
     template<typename T>
     void f(T param)
     
     int x1;
     const int x2;
     const int& x3;
     
     f(x1);		// T 为 int，param 为 int
     f(x2);		// T 为 int，param 为 int
     f(x3);		// T 为 int，param 为 int
     ```

  4. 数组和函数。由于数组形参和函数形参都将会被转换为指针类型，所以一般情况下按照第 1 种情况进行推测。但若把形参声明为数组的引用或函数的引用时，T 会被推导成实际的数组类型或函数类型，数组类型还会带有数组的大小

#### 02. 理解 auto 类型推导

- 模板类型推导和 auto 类型推导可以建立起一一映射的关系，它们之间也雀氏存在双线的算法变换。当某变量采用 auto 来声明时，auto 就类似于模板参数 T，而 auto 变量加上类型修饰则类似于 ParamType，赋值的右边相当于函数调用的实参。

  ```c++
  const int x;
  
  auto y1 = x;		// auto 为 int
  auto& y2 = x;		// auto 为 const int
  const auto* y3 = x;	// auto 为 int
  auto&& y4 = x;		// auto 为 const int&
  ```

- auto推导和模板类型推导唯一不同的是，若使用初始值列表初始化一个 auto 对象时，会被推导为 initializer_list\<T\> 类型的对象，而使用列表初始化来调用一个模板函数会报错：

  ```c++
  auto x = {0};		// auto 为 initializer_list<int> 类型
  
  template<typename T>
  void f(T param)
  
  f({0});				// 编译错误
  ```

- C++ 允许使用 auto 来表示函数的返回值需要推导，且 C++14 的 lambda 表达式也可以在形参声明中使用 auto，但这些 auto 使用的时模板类型推导，所以在无法使用列表来初始化对应形参或为 auto 返回类型返回一个列表：

  ```c++
  auto f(){
  	return {1};		// 编译错误
  }
  
  vector<int> vec;
  auto f = [&vec](const auto& temp) { vec = temp; }
  f({1});				// 编译错误
  ```

#### 03. 理解 decltype

- decltype 返回给定的名字或表达式的确切类型，若是左值表达式则返回 T& 类型，所以在使用 decltype 作为返回值的时候，使用左值表达式就可能返回一个局部变量的引用：

  ```c++
  int x;
  decltype((x)) y = x;		// (x) 是表达式，y 为 int&
  ```

- C++11 中，decltype 主要用途是用来声明返回值类型依赖于形参类型的函数模板，比如容器类型的 [] 操作符，返回 T&

- C++11 开始有返回值类型尾序语法，该方法的好处在于指定返回值类型时可以使用函数形参：

  ```c++
  template<typename T>
  auto f(T t) -> decltype(t){
  	return t;
  }
  ```

- C++11 允许对单表达式 lambda 式的返回值类型实施推导，而 C++ 14 将这个允许范围扩展到了一切 lambda 式和一切函数，包括多表达式的，所以在 C++14 中可以去掉返回值类型尾序语法，而只保留前导 auto。

- 某些类型推导时需要采用 decltype 类型推导规则，在 C++14 中可以通过 decltype(auto) 解决该问题，auto 指定想要实施推导的类型，而推导过程采用 decltype 的规则：

  ```c++
  template<typename T>
  decltype(auto) f(T t){
  	return t;
  }
  ```

#### 04. 掌握查看类型推导结果的方法

- IDE 编辑器：IDE 代码编辑器通常会在鼠标悬停至某个程序实体（变量、形参、函数等）显示该实体的类型。该方法奏效就需要代码处于一种可编译的状态，因为其工作原理是让 C++ 编译器或其前端在 IDE 内执行一次，若该编译器不能在分析代码时得到足够有用的信息，也就无法显示出推导的类型。
- 编译器诊断信息：使编译器显示其推导的类型的一个方法是使该类型导致某些编译错误，错误信息中基本上会提及导致该错误的类型。
- 运行时输出：使用 printf 等输出函数输出运行时获取的类型信息，该方法可以对类型的输出进行完全的控制，但是困难点在于如何为想要的类型创建输出的字符表示。可以通过 typeid(名字).name() 获取对应类型的名字，但它不保证返回任何有意义的内容，只是各个编译器的实现会在该返回内容里面填写，所以不同的编译器获得的结果不一样，且他的结果也并不可靠，他处理类型的方式就类似于向函数模板按值传参，所以引用和 const 都会被省略掉。可以通过其他库（如 Boost）的实现来获得准确的结果

### 2 auto

#### 05. 优先选用 auto，而非显示类型声明

- auto 的优点：

  - 用 auto 声明的变量其类型推导都时来自其初始化的对象，所以必须进行初始化，就不会因为没有初始化导致的问题。

  - 在使用迭代器时，auto 可以用来推导迭代器的值的类型，而不需要使用迭代器中存储的值类型（可能有相应的一些问题）来声明对应的局部变量：

    ```c++
    template<typename It>
    void f(It it){
    	typename std::iterator_traits<It>::value_type temp = *it;
    	auto temp = *it;
    }
    ```

  - 使用 auto 可以表示只有编译器知道的类型，如 lambda 表达式的类型。虽然可以通过 function 模板来指向任何类型的可调用对象，但是 auto 要比它更好：

    - 使用 auto 声明的、存储着一个闭包的变量和该闭包是同一类型的，所以要求的内存也是相同的。function 声明、存储的是一个 function 实例，所以不管给定的签名如何都会占有固定大小的内存，且该大小对于存储的闭包而言并不一定够用，若不够则构造函数会分配堆上内存来存储该闭包，所以 function 对象一般比使用 auto 占用更多的内存
    - 编译器的实现细节一般都会限制内力那，并产生间接函数调用，所以 function 来第哦啊用闭包几乎比通过使用 auto 声明的变量来调用的更慢。
    - 写一个 auto 比写一整个 function 实例类型更简单

  - 可以方便的指定机器相关类型，若对于一个机器相关的 size_t 显示指定为 unsigned，可能导致在 64 位上 size_t 为 64 而 unsigned 为 32，这样会在超过 32 位时导致错误，而使用 auto 则不会。

  - 显示指定类型可能导致既不想要也没想到的隐式类型转换，使用 auto 则不会担心声明变量的类型和初始化表达式的变量类型之间的不匹配问题，从而导致隐式类型转换。

#### 06. 当 auto 推导的类型不符号要求时，使用带显示类型的初始化物习惯用法

- 有些 “隐形” 的代理类和 auto 无法和平共处，这种类的对象往往会设计成仅仅维持单个语句内存在，所以要创建这种类的变量往往就违反了基本库的设计前提。比如 vector\<bool\>::reference 类，当使用 vector<bool\> 的 operator[] 操作就会返回该类型，但是该类型可以隐式转换为 bool 类型，而通过 auto 则会变成对应的代理类型：

  ```c++
  vector<bool> vec = {1, 2, 3};
  
  bool b1 = vec[0];		// vector<bool>::reference 会隐式转换为 bool 类型
  auto b2 = vec[1];		// 类型为 vector<bool>::reference
  ```

  该情况下解决方法不是放弃 auto，因为问题是 auto 没有推导出想要的类型，所以方法是通过一次强制类型转换。

  ```c++
  auto b2 = static_cast<bool>(vec[1]);
  ```

### 3 转向现代 C++

#### 07. 在创建对象时区分 () 和 {}

- 使用初始值的方式包括：小括号、大括号、等号、等号加大括号（C++ 通常和大括号的方式一样处理）：

  ```c++
  int x(0);
  int x{0};
  int x = 0;
  int x = {0};
  ```

  对于基础类型，这几个是一样的，对于自定义类型，等号是赋值操作符，而小括号则是对应的构造函数。大括号是为了解决众多初始化语法带来的困惑，也为了解决这些语法不能覆盖所有初始化场景问题，所以 C++11 引入了大括号进行统一初始化：

  - 大括号可以用于指定容器的初始值，而等号和小括号不行：

    ```c++
    vector<int> x = {0};
    ```

  - 大括号和等号可以为自定义类的成员指定默认初始值，而小括号不行：

    ```c++
    class Widget{
    private:
    	int x{0};		// 正确
    	int y = 0;		// 正确
    	int z(0);		// 错误
    }
    ```

  - 不可复制的对象可以采用大括号和小括号来进行初始化，而等号不行：

    ```c++
    std::atomic<int> ai1{0};		// 正确
    std::atomic<int> ai2(0);		// 正确
    std::atomic<int> ai3 = 0;		// 错误
    ```

  - 大括号禁止内建类型之间进行窄式隐式类型转换（会丢失数据的转换），而小括号和等号可以：

    ```c++
    int x = 1.0;	// 正确
    int y(1.0);		// 正确
    int z{1.0};		// 错误
    ```

  - C++ 规定任何能够解析为声明的都要解析为声明，所以如果使用小括号调用默认构造函数来创建一个对象，则会导致声明一个函数，但是使用大括号则可以避免该问题：

    ```c++
    Widget w1();		// 声明的一个函数
    Widget w2{};		// 调用默认构造函数声明的一个对象
    ```

- 大括号的缺点：若有一个或多个构造函数声明了具备 std::initializer_list 类型的形参，那么采用大括号初始化的会非常强烈的选用带有 std::initializer_list 类型的重载版本：

  ```c++
  class Widget{
  public:
  	Widget(int i, bool b);
  	Widget(std::initializer_list<long double> ld);
  }
  
  Widget w1(10, true);		// 调用第一个构造函数
  Widget w2{10, true};		// 调用第二个构造函数
  ```

  只要能转换成 std::initialzier_list 中的类型，即使是复制或移动构造函数都会被劫持：

  ```c++
  class Widget{
  public:
  	Widget(int i, bool b);
  	Widget(std::initializer_list<long double> ld);
  	operator float() const;		// 可以转换成 float
  }
  
  Widget w1(w);			// 执行复制构造函数
  Widget w2{w};			// 先转换成 float，然后 float 可以转换为 long double，所以执行第二个构造函数，移动构造函数同理
  ```

  若是没办法转换成 std::initialzier_list 中的类型，才会选择使用其他的构造函数。

  若使用一个空大括号来构造一个函数，且该对象支持默认构造函数，又支持带有 std::initializer_list 形参的构造函数，语言规定，应该执行默认构造函数，该空大括号表示没有实参，而不是空的 initializer_list，若的确想调用这个 std::initializer_list 形参的构造函数，可以通过传入空括号：

  ```c++
  Widget w({});		// 调用 std::initializer_list 构造函数
  Widget w{{}};		// 调用 std::initializer_list 构造函数
  ```

- 若是类的作者，需要意识到若构造函数中有一个 std::initializer_list 形参的，那么使用大括号初始化的客户代码就可能导致意向不到的调用，所以应该设计客户不管是大括号还是小括号调用的构造函数是一致的。

  若是客户代码的程序员，那么创建对象时使用大括号还是小括号需要考虑是否会产生意向不到的结果。

  若是开发模板的程序与，创建对象时使用大括号还是小括号时一个困难的问题：

  ```c++
  template<typename T, typename... Ts>
  void f(Ts&&... params){
  	T t1(std::forward<Ts>(params)...);
  	T t2{std::forward<Ts>(params)...};
  }
  
  std::vector<int> v;
  f<std::vector<int>>(10, 20);		// 若模板中声明为大括号则得到包含 2 个元素的 vector，使用小括号得到包含 10 个元素的 vector
  ```

  这也是 std::make_unique 和 std::make_shared 遇到的问题，这些函数使用的是小括号，且以文档的形式告知使用者。

#### 08. 优先选用 nullptr，而非 0 或 NULL

- 字面值常量的类型是 int 而非指针，当 C++ 在只能使用指针的语境出现了 0，也会勉强解释为空指针。从实际效果上说，该结论对于 NULL 也成立，因为标准允许各个实现给予 NULL 非 int 的整型类型。这两个都不具有指针类型，所以会在指针类型和整形之间的进行重载时出现意外，向这样的重载函数传递 0 和 NULL 是不会调用到指针类型的重载版本的：

  ```c++
  void f(int);
  void f(void*);
  
  f(0);			// 调用 f(int)
  f(NULL);		// 调用 f(int) 或者不会通过编译（依照具体实现）
  ```

- nullptr 不具备整形类型，也不具备指针类型，它的实际类型是 std::nullptr_t，且在一个循环定义下，std::nullptr_t 被指定为 nullptr 的类型，std::nullptr_t 可以隐式转换到所有的裸指针类型。

  优点：

  - 在使用 nullptr 调用指针类型形参和整形类型形参的重载函数时是不会调用到整形类型
  - 提高代码的清晰性，特别在使用 auto 时，可以明确看出这是指针类型，如果使用 0 则不太明显
  - 在模板中调用其他函数时，这些函数有指针类型的形参，那么使用 nullptr 就可以调用，但是使用 0 或 NULL 则会出错

#### 09. 优先选用别名声明，而非 typedef

- using 形式的别名形式相较于 typedef 的优点：

  - 别名声明在处理涉及函数指针的类型的时候可能比较容易理解：

    ```c++
    typedef void (*FP)(int, const std::string&);		// 使用 typedef
    using FP = void (*)(int, const std::string&);		// 使用别名声明
    ```

  - 别名声明可以模板化（该情况下叫做别名模板），typedef 只能用嵌套在模板化的 struct 里的 typedef 才能做类似的事情：

    ```c++
    template<typename T>
    using MyAllocList = std::list<T, MyAlloc<T>>;
    
    MyAllocList<Widget> widgetList;				// 客户端代码
    
    template<typename T>
    struct MyAllocList{
    	typedef std::list<T, MyAlloc<T>> type;
    }
    
    MyAllocList<Widget>::type widgetList;		// 客户端代码
    ```

    若还要在模板内使用这个别名创建对应的类型，那么 typedef 创建的名字还要加上 typename 前缀，因为不确定 ::type 该名字是否为类型，也可能是数据成员。而 using 别名声明则不需要：

    ```c++
    template<typename T>
    class Widget{
    private:
    	typename MyAllocList<T>::type list;		// 使用 typedef 声明的别名类型
    	MyAllocList<T> list;					// 使用 using 声明的别名类型
    }
    ```

- C++11 用类型特征（type trait）的形式给程序员提供如移除 const、引用等类型变换的工具，其在 type_traits 头文件提供了一整套模板，其中有几十个类型特征，它们并非都是执行类型转换功能的用途，但其中用做该用途的部分提供了可预测的结构，它们都是以 ::type 结尾，因为它们是用嵌套在 struct 中的 typedef 实现的，这样做有历史原因，但在 C++14 中使用了别名模板，对于每个 ::type 都可以变为 _t：

  ```c++
  std::remove_const<T>::type		// C++11
  std::remove_const_t<T>			// C++14
  ```

  其原理是：

  ```c++
  template<typename T>
  using std::remove_const_t = typename remove_const<T>::type
  ```

#### 10. 优先选用限定作用域的枚举类型，而非不限定作用域的枚举类型

- 限定作用域的枚举类型相较于不限定作用域的枚举类型的优点：

  - C++98 风格的枚举类型中定义的枚举值，它们的名字属于包含着这个枚举类型的作用域，就会导致名字泄露，而 C++ 11 中的限定作用域的枚举类型并不会以该方式泄露名字，其通过 enum class 声明，有时也称为枚举类。

  - 限定作用域的枚举类型的枚举值是强类型的，它不能隐式转换为其他类型，只能通过显示转型。而不限定作用域的枚举类型中的枚举值可以隐式转换到整形类型，并可以从此进一步转换为浮点类型。

  - 限定作用域的枚举类型可以直接进行前置声明，而不限作用域的需要一些额外工作（在 vs 的编译器中不存在该情况，它没有做接下来所说的优化，默认为 int 类型）。因为这种枚举类型会由编译器来选择一个证书类型作为其底层类型，为了节约内存，编译器通常会为它们选择足够表示枚举值的最小底层类型，在某些情况下，编译器会用空间换取时间，它们可能会不选择只具备最小可容尺寸的类型，但是它们也需要具备优化空间的能力，为了使这种设计成为可能，C++98 就只提供了枚举类型定义（列出所有枚举量）的支持，枚举类型声明则不允许，这样编译器就有可以在枚举类型被使用前逐个确定其底层类型选择哪一种。而限定类型的枚举类型的底层类型是已知的，默认是 int，若不何意可以推翻它，不限定作用域的枚举类型也可以同样的方式指定，该情况下就可以使用前置声明了：

    ```
    enum class Status : uint32_t;
    
    enum Status : uint32_t;
    ```

- 不限定作用域的枚举类型在某些情况下可能是有用的，比如当需要引用 C++11 中的 std::tuple 类型的各个域时：

  ```c++
  using UserInfo = std::tuple<std::string, std::string, std::size_t>;		// 名字、电子邮件、声望值
  
  UserInfo userInfo;
  auto val = std::get<1>(userInfo);		// 获取第一个域的值，这样可读性比较差
  
  enum UserInfoFileds{ UserInfoName, UserInfoEmail, UserInfoReputation };
  auto val = std::get<UserInfoEmail>(userInfo);		// 可读性比较强
  ```

  也可以采用限定作用域的枚举类型，但是需要强制类型转换，如果想减少类型转换需要的字数，就需要写个函数，而 get 是一个模板，所以该函数的结果能在编译期得出，就需要使用 constexpr 函数，由于还要配合任何的枚举类型，所以还需要是一个模板，且由于枚举的底层类型不一定相同，就需要泛化返回值，可以通过 std::underlying_type 类型特征取得返回值类型，还得声明为 noexcept 因为它不会生成异常：

  ```c++
  template<typename E>
  constexpr typename std::underlying_type<E>::type ToUType(E enumerator) noexcept{
  	return static_cast<typename std::underlying_type<E>::type>(enumrator);
  }
  ```

#### 11. 优先选用删除函数，而非 private 未定义函数

- C++98 中为了阻止成员函数被调用，采用的做法是将该函数声明为 private 并不去定义它们，声明为 private 就阻止客户去调用它们，而不定义它们是让类的友元和成员函数使用它们时会在链接时失败。

  C++11 使用 “=delete” 将函数标识为删除函数，其无法通过任何方法使用，即使是成员和友元函数。习惯上删除函数被声明为 public，因为当客户代码尝试使用某个成员函数时 C++ 会先校验可访问性，后校验删除装填，所以当客户代码想调用某个 private 删除函数时，有些编译器只会报错该函数为 private。

- 删除函数相对于 private 未定义函数的优点：

  - private 未定义函数在被成员函数和友元函数调用时，在链接时才会报错，而删除函数编译时就会出错。

  - 任何函数都能成为删除函数，但只有成员函数才能成为 private。比如由于任何可以看作数值的类型都能隐式转换到 int，所以对于某些不需要的版本可以通过删除函数剔除掉：

    ```c++
    void f(int number);
    void f(char) = delete;			// 拒绝 char 类型
    void f(bool) = delete;			// 拒绝 bool 类型
    void f(double) = delete;		// 拒绝 double、float 类型
    ```

    由于 float 类型在面临转型到 int 还是 double 时会优先选择 double，所以当删除 double 类型的形参时传入 float 会尝试调用 double 导致错误。

    而且可以阻止不应该进行的模板具现：

    ```c++
    template<typename T>
    void f(T t);
    
    template<>
    void f(int) = delete;
    ```

    而即使是在类中声明的函数模板也能通过 private 未定义函数进行删除，因为模板的特化只能在 namespace 作用域中进行而不是在类作用域。

#### 12. 为意在改写的函数添加 override 声明

- 要让改写函数的动作真的发生，需要满足以下条件：

  - 基类中的函数必须是虚函数
  - 基类和派生类中的函数名必须完全相同（析构函数除外）
  - 基类和派生类中的函数形参类型必须完全相同。
  - 基类和派生类中的函数常量性必须完全相同
  - 基类和派生类中的函数返回值和异常规格必须兼容

  C++11 新增了：

  - 基类和派生类中的函数引用饰词必须完全相同：

    ```c++
    class Widget{
    public:
    	void f() &;			// 仅在 *this 为左值时调用
    	void f() &&;		// 仅在 *this 为右值时调用
    }
    ```

- C++ 提供了 override 声明来显示的标明派生类的函数是为了改写基类版本，且一旦这样， 编译器就会检查所有和改写相关的问题

- C++11 的 override 和 final 是两个语境关键字，特点是语言会保留这些关键字，但是仅在特定的语境下。如 override 仅出现在成员函数声明的末尾才有保留意义，这代表一些遗留代码，其中已经用过 override 这个而名字的话不必因为升级到 C++11 而改名。

