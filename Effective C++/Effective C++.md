- 术语：

  - 声明式，告诉编译器某个东西的名称和类型，但略去细节

    ```c++
    extern int x;
    ```

  - 签名式，一个函数的参数和返回类型，该签名等同于函数的类型

  - 定义式，提供编译器一些声明式所遗漏的细节

    ```c++
    int x;
    ```

  - 初始化，给对象初始值的过程

    ```c++
    int x = 1;
    ```

### 1 让自己习惯 C++

#### 01. 视 C++ 为一个语言联邦

- C++ 已经是多重泛型编程语言，同时支持过程形式、面向对象形式、函数形式、泛型形式、元编程形式
- 其主要次语言：
  - C。块、语句、预处理器、内置数据类型、数组、指针都来自 C
  - Object-Oriented C++。class、封装、继承、多态、virtual 函数等等，该部分是对面向对象之古典守则在 C++ 上的直接实施。
  - Template C++。C++ 泛型编程部分
  - STL。一个 template 程序库，对容器、迭代器、算法、函数对象的规约有紧密配合和协调

#### 02. 尽量以 const，enum，inline 替换 #define

- #define 的缺点：

  - 不被视为语言的一部分，所以它定义的名称有可能没进入记号表中，当你运用该常量获得一个编译错误信息时，或在记号式调试器中，可能告诉你的是一个对应的具体值而不是名称。
  - 预处理器直接将宏替换成对应值，可能导致目标码出现多份宏变量
  - 无法利用 #define 创建一个 class 专属常量，一旦宏被定义，它在其后的编译过程中，只要没被 #undef 都有效

  用常量进行替换肯定不会出现上述问题。

  - 使用宏实现函数，在表达式中进行调用会出现问题，因为是直接进行字符串替换，若没有加括号，或者对传递参数有问题，则会导致一些不想要的结果。

    ```c++
    #define MAX(a,b) a > b ? a : b
    MAX(1,2) + 1;		// 等于 a > b ? a : b+1
    MAX(++a,b)			// 等于 ++a > b ? ++a : b
    ```

    这个可以使用 inline template 函数来进行替换。

- const需要注意的情况：

  - 定义常量指针时，由于常量定义经常放在头文件中，所以需要把常量所指向的对象也定义为常量

  - class 专属常量。为了将常量的作用域限制在 class 内，需要让他成为 class 的一个成员，而为确保此常量至多只有一份实体，就必须让它称为一个 static 成员。直接在类内初始化的是声明式，通常 C++ 要求对使用的任何东西提供一个定义式，若是 class 专属常量又是 static 且为整数类型，则需特殊处理，若要对它们取地址、引用绑定（ODR 的规则），或编译器坚持要看到一个定义式，就必须另外提供定义式：（gcc 需要，但 vs 不需要）

    ```c++
    const int A::a;
    ```

    旧编译器不支持在类内的声明式上获得初值，且 in-class 初值设定也只允许对整数常量进行，若不支持类内给定初值则将初值放在定义式。

    若不允许在类内赋初值又想在类内使用该初值：

    ```c++
    class A{
    private:
    	static const int a;
    	int arr[a];		// 错误
    }
    ```

    可以使用 enum hack 做法

    ```c++
    class A{
    private:
    	enum{ a = 1};
    	int arr[a];
    }
    ```

    enum hack 不能被引用或被取地址，这方面和 #define 类似。

#### 03. 尽可能使用 const

- STL 迭代器的作用像 T* 指针，声明迭代器为 const 和声明指针为 const 一样，若想指向的值是不可以改变的，则可以使用 const_iterator
- 令函数返回一个常量值可以防止别人的错误造成意外：如因为写错或判断相等时只写了一个 =，这会对返回的变量进行赋值。

**const 成员函数**

- const 成员函数使 class 接口比较容易被理解，它可以得知哪个函数可以改动对象内容哪个函数不行，且使操作 const 对象成为可能，这对编写高效代码是一个关键。改善 C++ 程序效率的一个根本方法是传输 const 对象的引用，但此技术的前提是有 const 成员函数可用来处理取得并经修饰而成的 const 对象

- 成员函数如果是 const，有两个流行的概念：

  - bitwise constness（physical constness），成员函数只有在不更改对象的任何成员变量（static 除外）时才可以说是 const，也就是不更改对象内的任何一个 bit，编译器只需要寻找成员变量的复制动作即可，这正是 C++ 对常量性的定义，所以 const 成员函数不可以更改对象内的任何非 static 成员变量。

    但一些函数不具备 const 的性质但能通过编译器的测试，当成员函数中包含了一个指针时，只有指针属于成员的所有物，所以指针指向的对象是可以改变的

  - logical constness，一个 const 成员函数可以修改它所处理的对象呃逆的某些 bits，但只有在客户端侦测不出的情况下才得如此，如

    ```c++
    class Text{
    public:
    	size_t length() const{
    		textLength = strlen(pText);		// 错误，const 函数体内不能赋值
    		return textLength;
    	}
    private:
    	char* pText;
    	size_t textLength;
    }
    ```

    利用 mutable 释放掉非 static 成员变量的 bitwise constness 约束即可：

    ```c++
    class Text{
    public:
    	size_t length() const{
    		textLength = strlen(pText);
    		return textLength;
    	}
    private:
    	char* pText;
    	mutable size_t textLength;
    }
    ```

**在 const 和 non-const 成员函数中避免重复**

- mutable 不能解决所有 const 相关问题，如 const 成员函数和相应的非 const 成员函数，它们有很多逻辑重复的可以重用，所以非 const 成员函数版本调用 const 成员版本是一个避免代码重复的安全方法，其中为了避免无穷递归，必须指出是 const 版本的，所以需要将 *this 转为 const 类型（使用 static_cast\<const\>）然后再去除 const（const_cast<>）

  用 const 成员函数调用对应的非 const 成员函数是不好的，因为在承诺过不会改变的函数中调用了没有承诺的函数，

#### 04. 确定对象被使用前已先被初始化

- 若使用 C part of C++ 且初始化可能招致运行期成本，则不保证发生初始化，一旦进入 non-C parts of C++，规则就发生变化。处理方法是永远在使用对象前将其初始化：对于无任何成员的内置类型，必须手动进行赋值，对于其他对象，则在每个构造函数中都将对象的每个成员初始化。

  C++ 规定对象的成员变量的初始化动作发生在进入构造函数本体之前（内置类型不一定为真），在构造函数内的都不是初始化，非内置类型在构造函数体前自动调用 default 构造函数。一个比较好的写法是使用 member initialization list 替代函数体内复制操作，它避免了调用 default 构造函数，而是调用对应的带参构造函数。对于内置类型，初始化和复制成本相同，但为了一致性最好通过成员初始值列来初始化。某些情况下，若成员变量是 const 或 references，它们就一定需要初值而不能被赋值。

- C++ 有固定的成员初始化顺序，base classes 总是早于其 derived classes 初始化，而 class 的成员变量总是以其声明次序被初始化，即使它们在初始化列表中以不同的次序出现，为了避免迷惑，最好在成员初始化列表中以声明的顺序进行初始化。

- static 对象，包括 global 对象，namespace 作用域内的对象，在 classes、函数内、file 作用域内被声明为 static 的对象。函数内的 static 对象被称为 local static 对象，其他的 static 对象称为 non-local static 对象，程序结束时 static 对象会自动调用析构函数销毁。

  若某编译单元内的某个 non-local static 对象初始化动作使用了另一个编译单元内的某个 non-local static 对象，则用到的这个对象可能未被初始化，因为 C++ 对定义于不同编译单元内的 non-local static 对象的初始化次序无明确定义，主要原因是不可能决定正确的初始化次序。

  解决该问题的方法是，将每个 non-local static 对象放入函数内，让函数返回它们的一个 reference，用户调用该函数即可（也就是 Singleton 模式）。原因是因为 C++ 保证，函数内的 local static 对象会在该函数被调用期间首次遇上该对象定义式时被初始化。这样做还有个好处是，如果没有调用过该函数则不会引发构造和析构的成本。

### 2 构造/析构/赋值运算

#### 05. 了解 C++ 默认编写并调用哪些函数

- 若没有声明，编译器会为 class 声明一个 copy 构造函数、一个 copy assignment 操作符和一个析构函数。若没有声明任何构造函数，编译器会声明一个 default 构造函数，这些函数都是 public 且 inline 的，唯有当这些函数被调用时才会被编译器创建出来。

  default 构造函数和析构函数负责调用 non-static 和 base classes 的构造函数和析构函数。

  copy 构造函数和 copy assignment 操作符只是单纯的将来源对象的每一个 non-static 成员变量拷贝到目标对象。

- C++ 并不允许 “改变 reference 指向不同” 对象，也不允许更改 const 成员，所以若成员内含有这两种成员，则不会自动生成 copy 和 copy assignment。若某个 base classes 将 copy assignment 操作符为 private，编译器也会拒绝构建这两个。

#### 06. 若不想使用编译器自动生成的函数，就该明确拒绝

- 若不想让编译自动生成 copy 构造函数和操作符，可以将它们声明为 private 的，从而阻止编译器创建其专属版本，但是这对于成员函数和 friend 函数还是可以访问的。解决方法是将成员变量声明为 private 且不是实现它们（C++ 的 iostream 程序库也是这么做的），也可以专门使用不实现的基类，让子类继承它们即可，但这可能会导致多重继承，在多重继承中有时会组织 empty base class optimization。

#### 07. 为多态基类声明 virtual 析构函数

- C++ 明确指出，当 derived class 的对象由一个 base class 指针被删除，则该 base class 带有一个 non-virtual 析构函数，其结果未有定义——实际执行时通常发生的是对象的 derived 成分没被销毁，这会导致资源泄漏、败坏数据结构、在调试器上浪费许多时间。

  解决方法是给 base class 增加一个 virtual 析构函数，任何 class 只要带有 virtual 函数都几乎确定应该有一个析构函数。若 class 不含 virtual 函数，通常表示它不意图做一个 base class，此时令其析构函数为 virtual 往往是个馊主意，因为带有虚函数会为每个类增加一个虚表指针，对于大量使用的对象来说这个消耗是比较大的，而且 C++ 的该对象也不能和其他如 C 语言等没有继承的语言进行移植，除非明确补偿虚表指针。

- 当需要一个将一个类声明为抽象类，但其他函数不能声明为纯虚函数的时候，可以声明 pure virtual 析构函数，但必须为该函数在外面提供一份定义，因为子类需要调用 base classes 的虚析构函数，所以需要这么做。

#### 08. 别让异常逃离析构函数

- C++ 不禁止析构函数吐出异常，但不鼓励这么做，程序可能过早的结束或出现不明确的行为

- 若析构函数必须执行一个动作，该动作可能会在失败时抛出异常（如管理链接的类），一个合理的想法是在该类的析构函数中调用 close 函数，通过 try、catch 语句块将它围起来 ，若在析构函数中导致异常将传播该异常，也就是允许它离开析构函数，但这会让异常难以驾驭。

  解决方法：

  - 若 close 抛出异常就结束程序，若程序遭遇该异常后无法继续执行，该方法就是一个合理的选项：

    ```c++
    try{
    	close();
    }catch(){
    	std::abort();
    }
    ```

  - 不对异常做出反应。一般而言该方法是不好的，因为它隐藏了某些动作失败的重要信息，但有时候不理睬也比草率结束程序或不明确行为带来的风险好。

  - 一个较好的策略是再设计一个 public 的 close 函数，可以让用户可以拥有一个发现和处理异常的机会，然后在析构函数中判断是否已经调用过 close 函数（通过加一个 bool 成员），若没有再调用，这时候调用失败还是从上面的两种方案中进行选择。

#### 09. 绝不在构造和析构过程中调用 virtual 函数

- 在构造函数中调用 virtual 函数，实际上是调用的自己的本身的那个函数。可以从几个方面解释：

  - 由于 base class 构造函数的执行早于 derived class 构造函数，当 base class 构造函数执行时 derived class 的成员变量尚未初始化，若此期间调用的 virtual 函数下降至 derived class 层阶，虚函数基本上都会使用那些 local 成员变量，而那些变量此时尚未初始化，这会导致不明确的行为。
  - base class 构造期间，对象类型是 base class 而不是 derived class，不止 virtual 函数会被编译器解析至 base class，若使用运行期类型函数（如 dynamic 和 typeid）也会把对象视为 base class 类型。这样的对待是合理的，因为 derived class 成员变量此时是未定义值，最安全的做法就是当它们不存在。

- 析构函数中调用 virtual 函数也是一样。一旦 derived class 析构函数开始运行，对象内的 derived class 成员变量便呈现未定义值，所以 C++ 视它们仿佛不再存在，进入 base class 析构函数后对象就成为一个 base class 对象，而 C++ 的任何部分包括 virtual 函数、dynamic_cast 等等也是这样。

- 若有多个构造函数，每个都需要执行某些相同工作，那么避免代码重复的一个优秀做法是把共同的初始化代码放进一个初始化函数中，然后这些构造函数调用该初始化函数。

- 若需要在继承体系中，对于每个类型的对象被创建都会有相应的方法来调用

  可以使用在 base class 内声明 non-virtual 的函数，在构造函数中调用该函数，然后让 derived class 将对应的初始化信息通过参数传进 base class 的构造函数。derived class 在初值列表内使用辅助函数给与 base class 构造函数参数，这比较方便和可读，且将该辅助函数声明为 static 就保证不会指向开始对象内没有初始化的成员变量：

  ```c++
  class Base{
  public:
  	Base(const std::string& logInfo){
  		Log(logInfo);
  	}
  	void Log(const std::string& logInfo);
  }
  class Derived : public Base{
      Derived(para)
          : Base(createLogString(para))
      {
              
      }
  private:
      static std::string createLogString(para);
  }
  ```

#### 10. 令 operator= 返回一个 reference to *this

- 关于赋值可以写为连锁形式：

  ```c++
  x = y = z = 15;
  ```

  为了实现这个，复制操作符必须返回一个 reference 指向操作符左侧的实参（对于自定义类就是 *this），这是为 class 实现复制操作符时应遵循的协议，它不仅适用于该标准赋值形式，还适用于所有赋值相关运算（如 +=，-=）。这只是个协议并无强制性，但该协议被所有内置类型和标准程序库提供的类型共同遵守，所以除非有一个标新立异的好理由，否则还是从众比较好。

#### 11. 在 operator= 中处理自我赋值

- “自我赋值” 发生在对象被赋值给自己时，这种会在使用 “别名” 的情况下无意中，即使用多个方法指向同一个对象：

  - 在多层循环中使用不同的下标，然后用下标进行赋值
  - 使用指针对指向的对象赋值，在同一个继承体系中，base class 的指针更容易和子类的指针指向同一个对象
  - 使用 using 和宏等

- 有时自我赋值中包含了指针的成员，此时需要对当前指针成员进行 delete，然后再 new 一个对应指针的副本，但这就会导致持有了一个被释放了的指针

  - 传统解决方法是证同测试，即在 delete 前进行判断是否和当前对象所指向的一样，但是这在异常方面存在问题，因为是先 delete 后 new，如在 new 时出现异常，则最终还是会持有一个被删除的指针

  - 在用 new 复制指针所指向的对象前不删除指针，即转换了一下 new 和 delete 的顺序，此时如在 new 时出现异常，则还是会持有以前的指针。若是关心效率问题也可以将证同测试放在前面，但是得先了解自我赋值发生的频率，因为它让代码变的更大一些，且导入一个新的控制流分支，这两者都会降低执行速度，Prefetching、caching、pipelining 等指令效率都会因此降低。

  - copy and swap 技术（和异常安全性有很大关系）：

    ```c++
    A A::operator=(const A& rhs){
    	A temp(rhs);	// 制作 rhs 的副本
    	swap(temp);		// 将 *this 和副本交换
    	return *this;
    }
    ```

    因为 calss 的 copy assignment 操作符可能被声明为以 by value 方式接受实参，而 by value 方式传递会造成一个副本。它降低了可读性，但是将 copy 动作从函数体内转移到函数构造阶段可以让编译器有时生成更高效的代码。

  因为 operator= 具备异常安全性往往会自动获得自我赋值安全的回报，所以很多人选择忽略赋值安全转而去实现异常安全性。

#### 12. 复制对象时勿忘其每一个成分

-  若自己实现 copying 函数，如果有些成员没有赋值，编译器不会给予你相关提示。当编写 copying 函数时，需要确保复制了所有 local 成员变量，调用所有 base classes 内适当的 copying 函数

- 令 copy assignment 操作符调用 copy 构造函数是不合理的，这像在构造一个已经存在的对象。

  令 copy 构造函数调用 copy assignment 操作符同样无意义，这就像在未初始化的对象上做只对已初始化对象有意义的事情

  若发现 copy 构造函数和 copy assignment 操作符有相近的代码，消除重复代码的做法是简历一个新的成员给两者调用，这样的函数往往是 private 且常被命名为 init。

### 3 资源管理

#### 13. 以对象管理资源

- 很多资源是 heap based，创建出来会持有该资源的指针，并在使用完后进行销毁，但很容易在这个中间触发 return、goto、break、异常等导致没有执行 delete 语句，所以可以将资源放进对象内，依赖 C++ 的析构函数自动调用的机制确保资源被释放。

- 以对象管理资源的两个关键想法：

  - 获得资源后立即放进管理对象内。以对象管理资源的观念常被称为资源取得时机便是初始化时机（Resource Acquisition Is Initialization，RAII），因为几乎总是在获得一个资源后同时用它初始化某个管理对象，也有时候用来赋值给管理对象，但都是在获得时就进入管理对象中
  - 管理对象运用析构函数确保资源被释放。不论控制流如何离开区块，一旦对象被销毁其析构函数自然会被自动调用，于是资源被释放

- auto_ptr（unique_ptr 的前身） 被销毁时会自动删除它的所指之物，为了避免这个问题，通过 copy 构造安徽省农户或 copy assignment 操作符复制它们就会变为null，而复制所得的指针将取得资源的唯一拥有权。

  auto_ptr 的替代方案是引用计数型智能指针（reference-counting smart pointer，RCSP），它持续追踪共有多少对象指向某个资源，并在没有指针指向的时候自动删除该资源，但它无法打破环状引用

  这两个指针都在内部做 delete 而不是 delete[] 动作

#### 14. 在资源管理类中小心 copying 行为

- 当一个 RAII 对象被复制时，常选择以下可能：
  - 禁止复制，许多时候允许 RAII 对象复制并不合理
  - 对底层资源使用引用计数法，有时候希望保有资源直到它的最后一个对象被销毁
  - 复制底部资源。有时候可以针对一份资源拥有其任意数量的副本，而需要资源管理类的唯一理由是不需要某个副本时确保它被释放，在该情况下复制资源管理对象应该同时也复制其所包裹的资源，即进行深度拷贝
  - 转移底部资源的拥有权。某些罕见场合可能希望确保永远只有一个 RAII 对象指向一个未加工资源，即使被复制也是如此，此时资源的拥有权会从复制物转移到目标物（unique_ptr）

#### 15. 在资源管理类中提供对原始资源的访问

- 有时候需要使用一个函数将 RAII class 对象转换为其内含的原始资源，对于智能指针，其重载了指针取值操作符也允许隐式转换成底部原始指针。对于 RAII 对象可以通过两个方法：
  - 显示转换函数，一般命名为 get，其返回底部持有的资源类型，可能对于客户程序员来说，每次需要使用时都要调用一次比较繁琐
  - 隐式转换函数，这使得客户调用 C API 时比较轻松，但是其增加了出错的可能性，会在不想要转换的时候进行转换

#### 16. 成对使用 new 和delete 时要采用相同形式

- 当使用 new 时，首先会通过 operator new 函数将内存分配出来，然后针对该内存有一个或多个构造函数被调用

  当使用 delete 时首先会有一个或多个析构函数被调用，然后再通过 operator delete 函数释放内存

- 单一对象的内存布局一般不同于数组的内存布局，数组通常还包括数组大小的记录，以便 delete 知道需要调用多少次析构函数，单一对象的内存则没有这个记录。编译器不一定非要这么实现，但很多编译器确实这样做的

  当对一个 new int 的指针使用 delete [] 形式，假设内存布局如上，则 delete 会读取一部分内存并将其解释为数组大小然后多次调用析构函数

  当对一个 new int[] 的指针使用 delete 形式，其可能导致太少的析构函数被调用

- 该规则对于 typedef 也比较重要，其如果是一个数组需要说清楚，否则使用 delete 时可能就不清楚使用哪一个 delete 形式。为避免这种问题，最好不要对数组形式做 typedef 形式的动作

#### 17. 以独立语句将 new 对象置入智能指针

- C++ 在函数调用、算术运算式等中并未指定操作数的调用顺序，若在这种情况下对智能指针进行如下类似操作：

  ```c++
  void fun(shared_ptr<A> ptr(new A()), operate());
  ```

  由于没有指定操作数的调用顺序，所以很可能调用 new A()，然后再调用 operate()，此时若调用该函数出现异常，那么 new 的对象就会遗失，此时并没有进入智能指针内。

  为避免该问题，应该将 new 和智能指针的赋值放在一个语句中，而不是在函数调用等不确定操作数调用顺序的地方。

### 4 设计与声明

#### 18. 让接口更容易被正确使用，不易被误用

-  理想上，若客户企图使用某个接口而却没有获得他预期的行为，这个代码就不该通过编译，若通过了编译则它的行为就是客户想要的
- 预防错误的几种方式：
  - 许多客户端错误可以导入新类型来预防。如时间类中需要年月日三个成员，但是若直接使用 int，那么三个数字可能输入错误，所以可以通过分别声明三个类然后传递三个对象进去。这三个类还能在里面对年月日限制值，一般不要用 enum 因为它可以转换成 int，不具有类型安全性，可以换为类内静态函数，让其返回相应年月日的对象。
  - 限制类型内什么事情可以做，什么事情不能做。常见的是限制是加上 const。
  - 除非有好理由，否则尽量让定义的 type 的行为和内置 type 一致
  - 任何接口如果要求客户必须记得做某事就有不正确使用的倾向，因为客户可能会忘记该事情。比如通过接口返回一个指针要求客户去进行 delete，可以通过返回一个智能指针替代
- shared_ptr 还有个特别好的性质是，它会自动使用它的每个指针专属的删除器，这可以消除另一个潜在的 “cross-DLL problem” 错误，该错误发生在对象在 DLL 中被 new 创建，却在另一个 DLL 内被 delete 销毁，智能指针的缺省删除器是来自它诞生所在的 DLL 中的 delete，所以不会出现该问题
- Boost 的 shared_ptr 是原始指针的两倍大，以动态分配内存作为专属数据的删除器，以 virtual 形式调用删除器，并在以多线程程序修改引用次数时承受线程同步化的额外开销，总之它比原始指针大且慢，而且使用辅助动态内存，在许多应用程序中这些额外的执行成本并不显著，但降低客户端错误是非常明显的。

#### 19. 设计 class 犹如设计 type

- C++ 和其他 OPP 语言一样，当定义一个新的 class 也就定义了一个新的 type。一个好的 type 有自然的语法，直观的语义，以及一个到多个高效实现品。
- 设计高效 class 要考虑的问题：
  - 新 type 应该如何被创建和销毁。这影响到 class 的构造函数和析构函数以及内存分配函数和释放函数，前提是打算撰写它们。
  - 对象的初始化和对象的赋值有什么样的差别。这决定了构造函数和赋值操作符的行为以及它们的差异，主要是赋值和初始化是两个不同的函数调用。
  - 新 type 的对象如果背 passed by value 意味着什么。copy 构造函数用来定义一个 type 的pass-by-value 如何实现。
  - 什么是新 type 的合法值。对 class 的成员而言通常只有某些数值集是有效的，这些数值集决定了你的 class 必须维护的约束条件，也决定了成员函数，特别是构造函数、复制操作符、set 函数必须进行的错误检查工作，也影响函数抛出的异常、以及极少被使用的函数异常明细。
  - 新 type 是否需要配合某个继承图系。若继承自某些已有的 class，就会受到这些 class 设计的束缚，特别是函数是 virtual 或 non-virtual 的影响。若这个 class 也可以被继承，那影响所声明的函数，特别是析构函数是否为 virtual。
  - 新 type 需要什么样的转换。若允许类型 T1 被隐式转换为 T2，就必须在 T1 内写一个类型转换函数，或在 T2 内写一个 non-explicit-one-argument 构造函数，若只允许 explicit 构造函数存在，就得写出专门负责执行转换的函数，且不能为类型转换操作符或 non-explicit-one-argument。
  - 什么样的操作符和函数对新 type 而言是合理的。这决定为你的 class 声明哪些函数，其中某些是 member 函数，某些不是。
  - 什么样的标准函数应该驳回。这些函数必须声明为 private。
  - 谁该读取新 type 成员。可以决定成员访问权限，也会帮助决定哪一个 class、function 应该是 friend，且将它们嵌套至另一个之内是否合理。
  - 什么是新 type 的未声明接口，它对效率、异常安全性以及资源运用（如多任务锁定和动态内存）提供何种保证。这些保证为 class 实现代码加上相应的约束条件。
  - 新 type 有多么一般化。或许并非定义一个新的 type 而是一个 type 家族，此时就不该定义 class 而是一个新的 class template。
  - 真的需要一个新 type 吗。若执行定义新的 derived  class 以便为既有的 class 添加机能，那么单纯定义一个或多个 non-member 函数或 templates 更能达到目标。

#### 20. 宁以 pass-by-reference-to-const 替换 pass-by-value

- 当一个对象比较大时，由于里面的所有成员变量都需要进行构造和析构，那么当函数传参以 pass-by-value 时会造成调用多个成员的构造和析构函数。

  pass-by-reference-to-const 效率就高的多，它没有任何构造函数和析构函数的调用，因为没有任何新对象被创建，且这个 const 是比价重要的，它向用户保证不会对传入的参数做出任何改变。

- 以 by reference 方式传递参数也可以避免对象切割问题。当一个 derived class 对象以 by value 方式传递，并且形参是一个 base class 类型，那么 base class 的 copy 构造函数会被调用，从而使 derived class 所有自身的特性都被切割掉了，只剩 base class 对象。

- C++ 编译器的底层，reference 往往以指针实现出来的，因此 pass by reference 通常意味着传递指针，因此若对象属于内置类型，pass by value 往往比 pass by reference 的效率高些，这个也适用于 STL 的迭代器和函数对象，因为习惯上它们被设计为 pass by value，迭代器和函数对象的实践者有责任看看它们是否高效且不受切割问题的影响。

- 小型的用户自定义类型不一定是 pass-by-value 的优良候选人。

  - 对象小并不意味着其 copy 构造函数不昂贵，比如包含指针的，复制时需要将指针和所指的对象都要复制。且即使小型对象拥有并不昂贵的 copy 构造函数，某些编译器对待内置类型和用户自定义的类型的态度也可能不同，即使两者的底层表述相同。
  - 作为一个用户自定义类型，其大小很容易有变化，一个 type 目前虽然小，但将来也许会变大，其内不识闲可能改变，甚至改用另一个 C++ 编译器都有可能改变 type 的大小。

- 一般而言，可以合理假设 pass-by-value 并不昂贵的对象就是内置类型和 STL 的迭代器和函数对象

#### 21. 必须返回对象时，别妄想返回其 reference

- 考虑函数返回一个引用。函数可以在 stack 空间或 heap 空间创建新对象。

  若是定义一个 local 对象，则是在 stack 空间，此时返回引用，那么 local 对象会在函数退出前销毁，此时会引用一个已销毁的对象，而且本来是为了避免调用构造函数，但还是会有一个构造函数的调用：

  ```c++
  const Rational& operator*(const Rational& lhs, const Rational& rhs){
  	Retional result(lhs.n * rhs.n, lhs.d * rhs.d);
  	return result;
  }
  ```

  若是用 new 在 heap 中创建一个新对象并用 reference 指向它，此时还是要使用一个构造函数，因为分配所得的内存将用一个适当的构造函数完成初始化操作，而且此时应该谁来负责完成指针的 delete 操作

  ```c++
  const Rational* operator*(const Raional& lhs, const Rational& rhs){
  	Rational* result = new Rational(lhs.n * rhs.n, lhs.d * rhs.d);
  	return *result;
  }
  
  Rational w, x, y, z;
  w = x * y * z;		// 调用了两次 operator*，但是没有办法对这些进行 delete 调用
  ```

  在函数内声明一个 static 对象然后再返回，可以在除了第一次以外没有任何构造函数的调用，但是它会引入多线程安全的问题，而且会在客户判断相等时出现问题：

  ```c++
  if((a * b) == (c * d))		// 由于 operator* 返回的是 static 对象的引用，所以这个式子永远是返回 true
  ```

- 一个必须返回新对象的函数的正确写法就是让函数返回一个新对象，当然必须承受该函数返回值的构造成本和析构成本，但长远来看也只是为了获得正确行为而付出的一些小小代价。

#### 22. 将成员变量声明为 private

- 将成员变量声明为 private 的理由：
  - 语法一致性。如成员变量不是 public，客户唯一能够访问对象的方法就是成员函数，若 public 内的每个东西都是函数，客户就不需要在打算访问 class 成员时是否该使用圆括号。
  - 使用函数可以对成员变量的精确控制。若令成员变量为 public，每个都可以读写，但以函数取得或设定该值，就可以实现出不准访问、只读访问、读写访问、甚至是只写访问。
  - 封装，若通过函数访问该成员碧昂量，日后可以更改或替换这个成员变量，而 class 客户一点也不会知道 class 的内部实现已经改变。
- protected 成员变量和 public 成员变量论点相同，语法一致性和精确控制访问也适用。对于封装角度来说也是一样的，假如拥有一个 protected 成员变量，而最终取消了它，俺么所有使用它的 derived classes 都会被破坏
- 从封装来说只有两种访问权限，private（提供封装） 和其他（不提供封装）。

#### 23. 宁以 non-member、non-friend 替换 member 函数

- 封装，若某些东西被封装它就不再可见，越多东西被封装就有越少人可以看到它，而越少人看到它就有越大的谈薪去变化它，因为我们的改变仅仅直接影响看到改变的那些人事物。可以通过访问该数据的函数数量对封装性进行一种粗糙的量测。
- 能够访问 private 成员变量的函数只有 class 的 member 函数加上 friend 函数而已，若你要在 member 函数（不止可以访问 class 内的 private 数据，也可以访问 private 函数、enum、typedef 等）和一个 non-member non-friend 函数（无法访问上述任何东西）之间做抉择，且两者提供相同机能，那么导致较大封装性的 non-member non-friend 函数，因为它并不增加能够访问 class 内的 private 成分的函数数量，friend 函数对 class private 成员的访问权力和 member 函数相同，因此两者对封装的冲击力道也相同
- 若只因在意封装性而让函数成为 class 的 non-member 并不意味它不可以是另一个 class 的 member，它可以作为某个工具类的 static member 函数，只要它不是以前类型的一部分或者它的friend 就不会影响 private 成员的封装性。在 C++ 比较自然的做法是这些和类有关系的 non-member 函数位于和该类同样的 namespace 内。namespace 和 class 不同，前者可以跨越多个源码文件而后者不能，将所有便利函数放在多个头文件内但隶属于同一个命名空间，意味客户可以轻松扩展这一组便利函数，需要做的就是添加更多的 non-member non-friend 函数到此命名空间内，这 class 无法提供的另一个性质，因为 class 定义式对客户而言除了派生以外是不能扩展的。

#### 24. 若所有参数皆需类型转换，请为此采用 non-member 函数

- 比如想实现一个乘法的时候，乘法需要满足交换律，若是 member 函数则只有位于参数列的才能进行隐式类型转换，对于 this 对象这个隐含的参数，不能进行隐式转换，所以需要将其变为 non-member 函数。
- member 函数的反面是 non-member 函数，而不是 friend 函数，因为 non-member 函数可以通过 public member 函数来实现。所以无论何时可以避免 friend 函数就该避免，就如同真实世界一样朋友带来的麻烦往往多余其价值。当然有时候 friend 也有其合理性，但也不能够只因函数不该为 member 就让其成为 friend。

#### 25. 考虑写出一个不抛异常的 swap 函数

- swap 将两对象的值彼此赋予对象，缺省情况下 swap 动作可由标准程序库提供的 swap 算法完成，其实现使用的三变量的交换方法。它对于用指针指向一个对象的类型，其常见表现形式是所谓 “pimpl 手法”（pointer to implementation），一旦要置换这种对象，本来只需要置换两个指针，但对于缺省的 swap 算法并不知道，它还会把底层的数据都进行复制。

- 通常我们不被允许改变 std 命名空间内的任何东西，但被允许为标准的 template 制造特化版本，使其能专属自定义 class。可以为 class 提供 public 的 swap 接口，然后用特化的 swap 调用 class 的 swap，该做法与 STL 容器有一致性，所有的 STL 容器都提供了 public swap 成员函数和 std::swap 特化版本来调用 public。

  ```c++
  class A{
  public:
  	void swap(A& other){
  		using std::swap;
  		swap(a, other.a);
  	}
  }
  
  namespace std{
  	template<>
  	void swap<A>(A& a1, A& a2){
  		a1.swap(a2);
  	}
  }
  ```

  若原本的类都是 template，那么由于不能直接对函数进行偏特化，所以通常的做法是为它添加一个重载版本，但 std 中客户可以全特化 template 但不能添加新的 template 进去，std 的内容完全由 C++ 标准委员会决定，他们禁止膨胀那些已经声明好的东西，虽然这样做基本上可以编译和执行，但是行为没有明确的定义

  ```c++
  namespace std{
  	template<typename T>
  	void swap<A<T>>(A<T>& a1, A<T>& a2);		// 错误，函数不能偏特化
  	
  	template<typename T>
  	void swap(A<T>& a1, A<T>* a2);				// 不合法，不能向 std 添加新的template
  }
  ```

  可以通过声明一个 non-member swap 让其调用 member swap，但是不将该 swap 声明为 std::swap 的特化版本或重载版本。

  ```c++
  namespace Test{
  	template<typename T>
  	class A{}
  	
  	tempalte<typename T>
  	void swap(A<T>& a1, A<T>* a2);
  }
  ```

  若这个版本没有在 namespace 当中，则调用不会有问题，但不要在 global 命名空间中塞满各种名字，所以若想要 class 版本的 swap 在尽可能多的情况下调用，则在调用 swap 的地方使用 using std::swap 使 std 的 swap 也进行曝光，这样当调用时没有合适的模板 T 存在就可以调用 std 的 swap。

- 成员版 swap 绝不可抛出异常，因为 swap 的一个最好的应用是帮助 class 和 class template 提供强烈的异常安全保障，这一约束只限制成员版，不能施行于非成员版，因为 swap 的缺省版本是以 copy 构造函数和 copy assignment 操作符为基础，一般情况下两者都允许抛出异常。因此对于一个自定版本的 swap 要提供高效和不抛出异常，一般来说高效率的 swap 几乎总是基于内置类型的操作，而内置类型上的操作绝不会抛出异常。

### 5 实现

#### 26. 尽可能延后变量定义式的出现时间

- 只要定义了一个变量而其带有一个构造函数或析构函数，俺么当程序的控制流到达这个变量定义式时，就需要构造成本，当变量离开作用于时就需要析构成本，即使该变量最终没有使用。
- 不止应该延后变量的定义，甚至应该延后该定义直到能够给定初始值为止，这样不仅能够避免构造和析构非必要对象，还可以避免无意义的 default 构造行为
- 对象需要在循环内部使用时，若 class 的一个赋值成本低于一组构造 + 析构成本，那么在循环外进行定义要高效（1 个构造和析构 + n 个赋值），但是该做法也会赋予变量更大的作用域，对维护性等会造成一点影响，所以除非该代码对效率比较敏感，其他情况可以不使用这个。若一个赋值成本高于构造 + 析构成本，那么在循环内定义要高效（n 个构造和析构）

#### 27. 尽量少做转型动作

- C++ 规则的设计目标之一是保证 ”类型错误“ 绝不可能发生，理论上程序可以很 ”干净“ 的通过编译，就表示它并不企图在任何对象身上执行任何不安全、无意义、愚蠢荒谬的操作

- 转型语法通常有三种不同的形式：

  - C 风格

    ```c++
    (T)expression;
    ```

  - 函数风格

    ```c++
    T(expression)
    ```

  这两种方式并无差别，只是小括号的位置不同

  - C++ 的始终新式转型：

    - const_cast。通常用来将对象引用的常量性移除，也是唯一有此能力的 C++ 转型操作符。
    - dynamic_cast。主要用来执行安全向下转型，也就是用来决定某对象是否归属继承体系中的某个类型，是唯一无法用旧式语法执行的动作，也是唯一可能耗费重大运行成本的转型动作。
    - reinterpret_cast。低层次的转型，实际动作和结果可能取决于编译器，所以其不可移植。
    - static_cast。强制隐式转换。

    旧式转型依然合法，但新式转型较受欢迎，因为他们很容易在代码中被辨识出来，因此可以更容易找到类型系统在哪个位置被破坏，且各类转型动作的范围更窄，编译器就更容易发现错误转型的运用。

- 任何一个类型转换，不论是显式还是隐式，往往令编译器编译出运行期间的代码。

- 建立一个 base class 指针指向一个 derived class 对象，有时候这两个指针值并不相同，该情况下会在运行期施加一个偏移量到 Derived* 身上以取得正确的 Base*。也就是说单一对象可能拥有一个以上的地址，若在 C++ 中使用多重继承，该情况经常发生，即使在单一继承也可能发生，这意味着通常应避免假设对象如何在 C++ 布局，更不应该以此基础执行任何转型动作。对象的布局方式和它们的地址计算方式随编译器的不同而不同，即知道对象如何布局而设计的转型在某一平台上行得通，但在其他平台上并不一定行得通。

- 转型可能写出不符合预期的代码，如对对象转型后调用相应的函数，而该函数改变了对象内部的值，但是由于转型时是返回的副本，所以实际上并没有更改到对应对象的值

  ```c++
  static_cast<BaseClass>(deriveObj).Change()		// Change方法更改的是返回的临时值
  ```

  解决方法是不使用转型操作，若在子类的函数中可以直接使用 BaseClass::Change() 调用父类对应的代码。

- dynamic_cast 的许多实现版本执行速度相当慢。如一个很普遍的实现版本基于 class 名称的字符串进行比较，若在第 n 层的单继承体系内的某个对象上执行 dynamic_cast ，那么就会执行 n 次 strcmp 调用来比较名称。

- dynamic_cast 通常是想在一个 derived class 对象身上执行 derived class 操作函数，但目前只有一个指向 base 的 reference，只能靠它们处理对象。有两个一般性做法可以避免这个问题：

  - 使用容器并在其中存储直接的指向 derived class 对象的指针，便消除了通过 base class 接口处理对象的需要。该做法无法在同一个容器存储指向所有可能的一个基类的所有派生类。
  - 在 base class 内提供 virtual 函数做你想对各个派生类想做的事

- 优良的 C++ 代码很少使用转型，应该尽可能隔离转型动作，若要使用转型，通常是把它隐藏在某个函数内，函数的接口会保护调用者不受函数内部任何肮脏龌龊的动作影响。

#### 28. 避免返回 handles 指向对象内部成分

- 成员变量的封装性最多只等于返回其 reference 的函数的访问级别，因为外部可以读取该成员变量，且若 const 成员函数传出一个 reference，而这个数据又是存储在其他类或结构体对象中的（若是自己类中的成员则不行），那么函数的调用者可以修改该数据。比如 A 结构体中有个 public 成员 a，B 类持有 A 结构体的指针，并在 const 成员函数中返回指向 A 中成员 a 的引用，那么是可以更改该数据的。对于该问题只要返回 const reference 就可以解决，但是封装性的话得看是否是想让客户看到而放松读取的权限，若是则也可以理解，但是这还会带来 dangling handle，即 handle 所指的对象可能在某地调用析构函数被释放了，导致指向一个无效的指针。
- References、指针和迭代器都是 handles，而返回一个带百多对象内部数据的 handle 就有降低对象封装性的风险，通常认为对象的内部指的是它的成员变量，但是不为 public 的成员函数也是内部的一部分，以为这不应该令成员函数返回一个指针指向访问级别更低的成员函数，若这么做，后者的实际访问级别会提升至前者。

#### 29. 为 “异常安全” 而努力是值得的

- 当异常被抛出时，带有异常安全性的函数会：

  - 不泄漏任何资源。
  - 不允许数据出问题。如维护一个 count，当异常出现时对象没有创建成功，但是 count 还是累加了

- 异常安全函数提供三个保证之一：

  - 基本承诺。若异常被抛出，程序内的任何事物仍然保持在有效状态下，没有任何对象或数据结构会因此破坏，所有对象都处于一种前后一致的状态，但是无法保证程序现在的情况是什么样的，只保证处于合法状态。
  - 强烈保证。异常被抛出，程序状态不改变，调用这样的函数若成功就是完全成功，失败程序就恢复到之前的状态
  - 不抛掷保证。承诺不抛出异常，因为它们总是能完成它们承诺的功能，作用于内置类型身上的所有操作都提供 nothrow 保证。

  除非面对不具备异常安全性的传统代码，否则应该提供上述三种保证之一。

- 若有

  ```c++
  int func() throw();		// 空白异常
  ```

  这并不代表 func 不会抛出异常，而是若该函数抛出异常将是严重的错误，会有意想不到的函数调用（通过 set_unexpected 函数指定的）。实际上函数的声明式并不能告诉一个函数是否是正确的、可移植的、高效的，也不能告诉它是否提供任何异常安全性保证，所有这些性质都根据实现决定。

- 很难在 C part of C++ 领域中调用任何一个可能抛出异常的函数，所以只能在可能的情况下提供 nothrow 保证，但对大部分函数而言，基本上是在基本保证和强烈保证之间做选择

- copy and swap 策略：为打算修改的对象原件做出一个副本，然后在副本上做出一切必要的修改，若有任何修改动作抛出异常，原对象仍然保持未改变状态，等所有改变都成功了再将修改过的那个副本和原对象在一个不抛出异常的操作中 swap。实现通常是将所有隶属对象的数据放入另一个对象，类中只存储对应的指针，交换时也只需要交换指针即可，该方法常称为 pimpl idiom。

  该策略是让对象状态从全无到全有改变的一个好方法，但它并不保证整个函数有强烈的异常安全性。

- 若函数只操作局部性（如对象内的数据）的状态，便相对容易的能提供强烈保证，但当函数对非局部性数据（如数据库的数据）有连带影响时，提供强烈保证就困难的多。

- 当提供强烈保证不切实际时，就必须提供基本保证

- 一个软件系统要么具备异常安全性要么就不具备，没有所谓的局部异常安全性，因为其他地方只要调用不具备异常安全性的函数就可能导致资源泄露或数据败坏

#### 30. 透彻了解 inlining 的里里外外

- Inline 函数比宏好的多，可以调用它们又不需要承受函数调用导致的额外开销，且编译器最优化机制通常被设计用来浓缩那些不含函数调用的代码，所以 inline 某个函数，或许编译器就有能力对该函数调用的地方执行最优化，大多数编译器不会对一个非 inline 函数调用动作执行最优化。
- inline 函数的观念是将对该函数的每一个调用都用函数本体替代，这样可能增加目标码的大小，在一台内存有限的机器上，过度的 inlining 会造成程序太大，即使拥有虚内存，inline 造成的代码碰撞也会导致额外的换页行为，降低指令高速缓存的命中率。换个角度，若 inline 函数的本体很小，编译器所产生的的目标码可能比函数调用所产出的更小，这样也会获得较高的指令高速缓存命中率。
- inline 只是对编译器的一个申请，不是强制命令它可以显式或隐式的提出：
  - 隐式：将函数定义于 class 定义式内，这样的函数通常是成员函数，也可以是 friend 函数，都会被声明为 inline
  - 显式：在函数定义式前面加上关键字 inline
- inline 函数通常一定被置于头文件内，因为大多数的 build environment 在编译过程中进行 inlining，而为了将一个函数调用替换为函数本体编译器必须知道函数定义是什么，C/C++ 中是在编译期完成。某些 build environment 可以在链接时完成 inlining，少量 build environment 可以在运行期完成 inlining（如 .NET CLI 的托管环境）。
- Template 通常也被置于头文件，因为它们一旦被调用，编译器就需要对其进行具现化，需要知道它的定义，这个也是看 build environment。但是 Template 的具现化与 inlining 无关，所以你认为一个 template 的所有具现化版本都应该 inlined 那么就将该 template 声明为 inline，但如果不是所有版本都需要，那么就避免声明为 inline。
- 大部分编译器拒绝将太过复杂的函数 inline，如带有函数和递归，而所有对 virtual 函数的调用也不会 inline，除非是最平淡无奇的，因为 virtual 函数是在运行期间才确定调用哪个函数，inline 是在执行前将函数进行替换。这说明 inline 函数是否真的 inline 取决于 build environment 和 编译器，现在大多数编译器提供了一个诊断，若它们无法将要求的函数 inline 会抛出一个警告。
- 编译器通常不对通过函数指针而进行的调用实施 inline，因为 inline 函数没有本体，编译器没有能力提出一个指针指向并不存在的函数，所以要先生成一个非 inline 本体才行。所以 inline 函数调用是否被 inline 还取决于调用的方式。编译器也有可能使用指针，如有些编译器生成构造函数和析构函数时会生成一个非 inline 的本体，用一个指针指向它们，这样就可以在 array 里面自动进行构造和析构。
- 构造和析构函数不适合 inline。C++ 对于对象的创建和销毁做出了很多保证，使用 new 和 delete 时会自动初始化和析构，子类对象的 base class 的构造函数和析构会调用，成员会自动初始化，但只是保证了结果，过程都是依赖编译器的实现，所以一定有地方保证了这些东西的发生，有些时候这些代码就放在了构造和析构函数内。
- inline 函数无法随着程序库的升级而升级，  所以一旦程序库的设计者决定改变 inline 函数，所有用到该 inline 的地方都需要重新编译
- 大部分调试器对于 inline 函数束手无策，因为不知道怎么在一个并不存在的函数设立断点。

- 一开始先不要将任何函数声明为 inline，或至少限定那些必须成为 inline 的，到以后再决定是否对一个函数 inline

#### 31. 将文件间的编译依存关系降至最低

- 若是在定义类型 A 的头文件中 include 其所需要的其他类型头文件，就会在这些头文件中形成编译依存关系，若这些文件中有任何一个被改变，或者这些文件所依赖的其他头文件被改变，那么每一个使用类型 A 或者类型中含有 A 的都必须重新编译。

- 可以通过前置声明使用的类型来替代 include 对应的头文件，对于标准库不应该使用前置声明，因为标准库的声明比较复杂，且标准头文件不太可能成为编译瓶颈，特别是你的 build environment 允许使用预编译头文件，若解析标准头文件真的是一个问题，那么就需要改变接口的设计，避免使用标准程序库中引发导致问题的 include 的那一部分。

  C++ 将接口和成员都声明在头文件中，而不是只声明接口的原因是，当使用者使用时需要知道一个对象占用多少内存，这个问题在 Java 等语言上并不存在，因为那种语言只需要一个指针指向该对象。所以也可以将成员实现为指向对象的指针，这样一个类的实现就分为了定义类的头文件提供了接口，其成员的所使用的类型所定义的头文件提供了实现，这样的设计常被称为 pimpl idiom（pimpl 是 pointer to implementation）

  这两个结合起来就是接口与实现分离，这个分离的关键在于用声明的依赖性替换定义的依赖性，这就是将编译依赖性最小化的本质，让头文件尽可能的自我满足，万一做不到就让它依赖其他文件类型的声明式：

  - 若使用 object reference 或 object pointer 可以完成任务，就不要使用 object，可以只靠一个类型声明式就定义出指向该类型的 reference 和 pointer，若定义某类型的object 就需要用到该类型的定义式。

  - 尽量使用 class 的声明式替换 class 的定义式，当声明一个函数而用到某个 class 时，并不需要该 class 的定义，即使函数以 by value 方式传递该类型的参数。不包含对应的头文件其实是让最后使用的文件去包含对应的头文件，这样做是因为所有使用该类型的文件很少使用该类型的所有函数，所以可以将不是真正需要的那部分从具体使用该类型的文件中去掉，减少了这部分的依赖。如 100 个文件使用了类型 A，但只有 50 个文件使用了里面的有关 B 类型的成员，那么就节省了 50 个函数的依赖。

  - 为声明式和定义式提供不同的头文件，这需要两个文件，一个用于声明，一个用于定义，这些文件必须保持一致性，若有个声明式被改变了，这两个文件都得改变，因此程序库的客户应该总是 include 一个声明文件而非前置声明若干个函数，程序库的作者也应该提供这两个文件。只含声明式的头文件命名应以 fwd 结尾，如标准库头文件 \<iosfwd\>，它内含了 iostream 各组件的声明式，其定义分布在若干个不同的头文件内，包括 <sstream\>、\<streambuf\>、\<fstream\>、\<iostream\>

    <iosfwd\> 也彰显了本条款适用于 template 也适用于 non-template，有某些 build environment 允许 template 定义式放在非头文件中，这样就可以将只含声明式的头文件提供给 template，<iosfwd\> 就是如此。C++ 也提供关键字 export，允许 template 声明式和定义式分割于不同的文件内，但是现在支持该关键字的编译器非常少。

- 使用 pimpl idom 的 class 往往被称为 Handle class，这样的 class 完成实际的工作有几个方法

  - 将它们的所有函数转交给相应的实现类并由后者完成实际工作，而它们只是调用一下这个函数就行

  - 让 handle class 成为一种特殊的 abstract base class，它的目的是详细描述 derived class 的接口，因此它通常不带有成员变量，也没有构造函数，只有 virtual 析构函数和一组 pure virtual 函数，这样客户就只能使用 handle class 的 reference 来编写引用程序，它们自身不能实例化出对象，所以需要调用 factory 函数或者 virtual 构造函数来创建 derived 对象，它们返回指针，指向动态分配所得的对象，这样的函数又会在 base class 中声明为 static。

    成员必须通过 reference 取得对象的数据，那么每一次访问都增加一层间接性，每一个对象消耗的内存数量必须增加一个 pointer 大小，且需要进行动态分配，所以将遭受动态分配和释放带来的额外开销，以及 bad_alloc（内存不足）异常的可能性。然后由于每一个函数都是 virtual 那么每次函数调用都需要付出一个间接跳跃的成本，且其派生的对象都必须含有一个 vpr

### 6 继承与面向对象设计

#### 32. 确定你的 public 继承塑模除 is-a 关系

- 以 C++ 进行面向对象编程，最重要的一个规则是 public 继承意味着 is-a 的关系，若令 class D 以 public 形式继承 class B ，那么 B 对象使用的任何地方，D 对象一样可以使用，但若是需要 D 对象的地方，B 对象不一定能使用。
- 有时候直觉会误导人，企鹅是一种鸟，鸟会飞，但事实上企鹅不可以飞，所以有一些鸟不能飞。世界上并不存在一个适用于所有软件的完美设计，所谓的最佳设计取决于系统希望做什么事，包括现在与未来。有几种思想可以处理企鹅和鸟：
  - 若鸟不在乎是否能飞行，可以使用双继承体系来实现
  - 为企鹅重新定义 fly 函数，令其产生一个运行时错误，但是好的接口可以防止无效的代码通过编译，所以应该在编译器拒绝企鹅飞行而不只是在运行时检测

#### 33. 避免遮掩继承而来的名称

- base class 内的函数会被 derived class 内中同名的函数遮掩掉，即使 base class 和 derived class 内的函数有不同的参数类型也适用，且不论函数是否是 virtual。从另一个角度看 base class 对应的函数不再被继承。该行为的理由是防止在程序库或应用框架内建立新的 derived class时附带的从疏远的 base class    继承重载函数，但通常是想要继承重载函数，且如果正在使用 public 继承而不继承重载函数，就是违反了 base 和 derived class 之间的 is-a 关系。
- 若继承 base class 并加上重载函数，且又希望重新定义或覆盖一部分，那么必须使用 using 声明为每个名称被遮掩的函数声明一次。这个在 public 继承下是不会发生的，若在 private 继承下可能是有意义的，比如只想继承一个名称的一个函数，但是使用 using 指令会继承所有的重载函数，此时可以使用一个转交函数，在其中调用父类的对应版本即可，这种转交函数的另一个用途是为那些不支持 using 声明式的老旧编译器开辟一条新路，将继承而来的名称汇入 derived class 作用域内

#### 34. 区分接口继承和实现继承

- public 继承由两部分组成：函数接口继承和函数实现继承
- 作为 class 设计者，有时候会希望 derived class 只继承成员函数的接口，有时候同时继承函数的接口和实现并希望能重写实现，有时候希望继承函数的接口和实现但不允许重写任何东西：
  - 成员函数的接口总是会被继承
  - pure virtual 函数必须被继承了它们的具体 class 重新声明，且在抽象 class 中通常没有定义，所以声明一个 pure class 函数的目的是让 derived class 只继承函数接口
  - 声明 impure virtual 函数的目的，是让 derived class 继承该函数的接口和缺省实现。允许 impure virtual 函数同时指定函数声明和函数缺省行为有时可能造成危险。如在继承体系中新增一个 derived class，但子类并不需要该缺省行为而需要自己的行为，但是却忘记重写该行为，解决该问题可以通过将对应的函数声明为 pure virtual，然后提供一个 protected 的代表缺省行为的函数，需要缺省行为的就自己在子类调用默认行为。有些人反对以不同的函数分别提供接口和缺省实现，他们关心因过度雷同的函数名称而引起的 class 命名空间污染问题，可以利用 pure virtual 函数必须在 derived class 中重新声明，但也可以拥有自己的实现这一事实，所以可以在 base class 中声明 pure virtual 函数，并在类外实现实现一个缺省行为，然后子类的实现中通过 base::函数 调用缺省行为，这种方法使名字一致但是却丧失了使缺省行为与 pure virtual 函数访问级别不一样的能力。
  - 若成员函数是个 non-virtual 函数，意味着它不打算在 dervied class 中有不同的行为，所以使子类继承函数的接口及一份强制性实现
- 遵循上面的准则可以避免经验不足的 class 设计者常犯的两个错误：
  - 将所有函数声明为 non-virutal，这时 derived class 没有空间进行特化工作，尤其是 non-virtual 析构函数会带来问题。当然对于一个不想成为 base class 的 class，其中的函数都是 non-virtual 是合理的。
  - 将所有成员函数声明为 virtual，有时候是正确的，如 interface，但某些函数就是不该在 derived class 中被重新定义，对于这些函数应该声明为 non-virtual

#### 35. 考虑 virtual 函数以外的其他选择

**使用 Non-Vritual Interface 手法实现 Template Method 模式**

- 该流派主张 virtual 函数应该几乎总是 private，建议较好的设计是保留 public virtual 函数为 public，但是让它成为 non-virtual，并调用一个 private virtual 函数进行实际工作，其成为 non-virtual interface（NVI）手法，是所谓的 Template Method（与 C++ template 无关）设计模式的一个独特表现形式。

  NVI 的优点是可以在调用 virtual 函数之前和之后做一些工作，这可以保证在一个 virtual 函数调用前做好准备工作，调用后清理场景。可能令人疑惑的是它涉及在 derived class 内重新定义 private virtual 函数，重新定义若干个 derived class 并不调用的函数，但在这里并不矛盾，重新定义 virtual 函数表示某些事如何被完成，调用 virtual 函数则表示它何时被完成，这些事情是独立互不相干的，所以可以允许 derived class 重新定义 virtual 函数，但 base class 保留诉说函数何时被调用的权利。

  某些继承体系要求 derived class 在 virtual 函数的实现内必须调用其 base class 的对应兄弟，而为了让这样调用合法该 virtual 函数就必须是 protected 的。

**使用 Function Pointers 实现 Strategy 模式**

- Strategy 设计模式的一种简单应用可以使 class 中持有一个函数指针，然后使用 non-member 函数定义 Strategy，运行时通过切换指针指向的函数调整策略，不同对象也可以拥有不同策略。但如果需要 class 的 non-public 成员进行计算，non-member 函数就不适用了，实际上只要是将 class 内的某个技能替换为 class 外的某个等价技能，都可能存在该问题
- 唯一能解决以 non-member 函数访问 class 的 non-public 成分方法就是弱化 class 的封装，御用函数替换 virtual 函数的优点是否可以弥补缺点，需要根据每个设计情况的不同而决定

**使用 function 类完成 Strategy 模式**

- 使用函数指针约束了需要的函数必须满足其签名，若使用 function 对象就可以保存所有近似的可调用对象，对调用对象的限制要求更低。特别是当使用 bind 函数之后，使可调用对象更灵活。

**古典的 Strategy 模式**

- 传统的 Strategy 做法会将 Strategy 函数做成一个分离的继承体系中的 virtual 成员函数，需要策略的类通过组合方式保存策略对象指针，然后更改该指针指向的策略继承体系的不同类型对象就可以更改策略

#### 36. 绝不重新定义继承而来的 non-virtual 函数

- 如果 derived class 真的有必要实现和 base class 不同 non-virtual 函数，那么每个 is-a 的关系就不成立，就不应该 public 继承 base class，且若必须以 public 继承 base class，那么也无法体现出不变性凌驾于特异性，该函数应该声明为 virtual 的，所以在任何情况下都不盖重新定义一个继承而来的 non-virtual 函数

#### 37. 绝不重新定义继承而来的缺省参数值

- virtual 函数是动态绑定的，而缺省参数值是静态绑定的。静态绑定又名前期绑定（early binding），动态绑定又名后期绑定（late binding）。对象的静态类型是在声明时所采用的类型，动态类型是目前所指向的对象类型。

  如果调用定义在 derived class 内的 virtual 函数，该函数在 base class 中有缺省参数值，那么会使用 base class 中的缺省参数。

- C++ 以该方式运作的原因是运行期效率，若缺省参数值是动态绑定，那么编译器就必须有某种方法在运行期为 virtual 函数决定适当的参数缺省值，会比在编译器决定的机制更慢更复杂。

- 若想要在继承体系中的 class 为 virtual 函数提供不同的缺省参数值，可以使用替代设计。如 NVI 手法，另 base class 内的一个 public non-virtual 函数调用 private virtual 函数，后者可以被 derived class 重新定义，所以可以让 non-virtual 函数提供缺省参数值，而 private virtual 函数做真正的工作。

#### 38. 通过复合塑模除 has-a 或根据某物实现出

- 复合是类型之间，其中一个类型的对象内含另一种类型的对象。该术语有很多同义词，包括 layering（分层）、containment（内含）、aggregation（聚合）、embedding（内嵌）
- 复合意味着 has-a 或 is-implemented-in-terms-of。若程序中的对象相当于所塑造世界中的某些事物，这样的对象就属于应用域部分，发生于这些对象间的复合表现 has-a 的关系，若对象存粹是为了实现细节上面的人工制品，这样的对象就属于软件的实现域，发生于这些对象间的复合表现 is-implemented-in-terms-of 关系（如想通过 list 实现出 stack）。

#### 39. 明智而谨慎的使用 private 继承

- private 继承意味着 implemented-in-terms-of，若让 class D 以 private 形式继承 class B，那用意就是为了采用 class B 内已经具备的某些特性，而不是因为它们在概念上有什么关联，private 继承意味着只有实现部分被继承，接口部分被省略，它只是一种实现技术

- 尽可能使用复合，必要时才使用 private 继承，主要是在 protected 成员和 virtual 函数牵扯进来的时候，还有在 empty class 的特殊情况下

- 如果使用 private 继承：

  - 若想设计子类使其不能重新定义父类中的 virtual 函数这是不可能的事情，因为子类可以重新定义即使不能调用它。
  - 这样做可能使两者的编译依赖增加，在子类中必须要看到父类的定义，那么就需要在头文件中 #include 父类对应的头文件。如果使用组合可以使用指针，这样就只需要一个声明式就可以

- 在有个特殊情况下可能 private 继承比组合好，就是空白类，这种类不带有任何数据，也没有 virtual 函数，它理论上不存在任何空间，但是在技术上，C++ 裁定所有独立对象都需要有非零的大小，所以大多数编译器中空白类的对象的 sizeof 为 1，C++ 会默默安插一个 char 到空对象内，但是对于 struct 和 class 的对齐可能导致组合空白类不止有一个 char 的大小，但如果是继承的话那空白基类的大小就为 0，这就是所谓的 EBO（empty base optimization），EBO 一般只在单继承下才可行，规定 C++ 对象布局规则那些人通常表示 EBO 无法施行于拥有多个 base 的 derived class。

  empty class 中往往内含 typedefs、enums、static 成员变量、non-virtual 函数，STL 中有很多技术用途的 empty class，包括 unary_funcion、binary_function

- 在并不存在 is-a 关系的两个类之间，其中一个想访问另外一个的 protected 成员或需要重新定义其中一个或多个 virtual 函数，private 继承可能就有用。一个混合 public 继承和复合的设计可以替换这种行为，尽管它比较复杂。

#### 40. 明智而谨慎的使用多重继承

- 当使用多重继承，程序可能从一个以上的 base class 继承相同的名称（函数、typedef 等），这会导致歧义。为了解决这个歧义，必须指定要调用的是哪一个 base class 中的函数

- 任何时候若有一个继承体系，而其中的某个 base class 和某个 derived class 之间有一条以上相通的路线，就必须决定是否打算让 base class 内的成员变量经由每一条路径被复制，C++ 支持这两种方法，但是默认的做法是执行复制。若不想执行复制，就必须让被多次继承的那个 base class 称为一个 virtual base class，那就必须令所有继承该 base class 的使用 virtual 继承：

  ```c++
  class A{};
  class B: virtual public A{};
  class C: virtual public A{};
  class D: public A, public B{};
  ```

  从正确的行为上看，public 继承应该总是 virtual 的，但是编译器为了防止继承的变量重复需要做额外工作，所以使用 virtual 继承的 class 产生的对象往往比没使用的跟打，访问 virtual base class 的成员变量也比没使用的更慢，里面的细节因编译器而异，但是得为 virtual 继承付出代价。

- virtual base 的初始化责任是由继承体系中的最低层 calss 负责，即 class 若派生自 virtual base 而需要初始化，必须知道其 virtual base，不论距离它多远，且当一个新的 derived class 加入继承体系中还必须承担其 virtual base 的初始化责任。

- 非必要不使用 virtual base，平常请使用 non-virtual 继承，若要使用 virtual base class，尽可能避免在其中放置数据，这样就不要担心这些 class 身上的初始化和赋值所带来的奇怪的事情了（C# 和 Java 中的 Interface 就是差不多原理）。

### 7 模板与泛型编程

#### 41. 了解隐式接口和编译器多态

- 面向对象编程世界总是以显示接口和运行期多态解决问题。Template 泛型编程世界和面向对象有根本上的不同，在该世界中显式接口和运行期多态仍然存在，但重要性贬低，反而隐式接口和编译器多态重要性提高。

  运行期多态是哪一个 virtual 函数被绑定，编译器多态是哪一个重载函数被调用

  显式接口由函数的签名式（函数名、参数类型、返回类型）构成，隐式接口不基于函数签名，而是由有效表达式组成，即 template 中使用该类型的地方应该满足什么条件。隐式接口和显示接口都在编译期完成检查，无法在 template 中使用不支持 template 隐式接口的对象。

#### 42. 了解 typename 的双重意义

- C++ 并不总是把 class  和 typename 等价，有时一定得使用 typename。

- template 内出现的名称若依赖于某个 template 参数称为从属名称（如 T t），若从属名称在 class 内呈嵌套状称之为嵌套从属名称（如 T::xxx t），若并不依赖任何 template 参数的名称，这样的名称称为非从属名称

- 嵌套从属名称有可能导致解析困难，对于 T::x* t，若 T 类型正好有静态成员 x，那么该表达式可能就意味着乘法，若 x 代表的是一个类型那么这个表达式就意味着指针。C++ 中，若解析器在 template 中遇到一个嵌套从属名称，它会假设该名称不是类型，除非在嵌套从属名称前添加关键字 typename，如 typename T::x* t，所以一般性的规则就是在所有嵌套从属名称前加上关键字 typename，且 typename 也只用来验证嵌套从属名称，其他名称前不该有它。但是在成员初始化列表以及继承 base class list 处不能使用 typename：

  ```c++
  class Derived : public typename Base<T>::XXX{	// 错误
  public:
  	Derived(int x) : typename Base<T>::XXX(x){	// 错误
  	
  	}
  }
  ```

- typename 相关规则在不同编译器上有不同的实践，某些编译器接受的代码原本该有 typename 却遗漏了，原本不该有 typename 却出现了，还有少数编译器拒绝 typename，所以 typename 和嵌套从属类型名称之间的作用，可能会在移植性上面造成影响

#### 43. 学习处理模板化基类内的名称

- 对于 template 模板参数，在编码时是不会知道该 template 的具体类型，所以也无法使用它的 public 成员
- 为了使用特定模板参数的 public 成员，可以使用 template<> 指明全特化，即没有参数可以变化，这样就可以使用指定类型的 public 成员
- 当 template class 用作基类的时候，子类是无法直接使用父类的 public 成员，因为 base class 有可能被特化，而特化版本可能不提供一般性 template 相同的接口，因此往往拒绝在 templatized base class 中寻找继承而来的名称，解决这个有三个方法：
  - 在 base class 函数调用动作之前加上 this->
  - 使用 using 声明式来讲一个 base class 名称带入 derived class 作用域内、
  - 明确使用 base class::成员名 调用对应成员，但如果调用的是 virtual 函数，该解法会关掉 virtual 函数的动态绑定。

#### 44. 将参数无关的代码抽离 template

- 使用 template 可能导致代码膨胀，其二进制带着重复或几乎重复的代码、数据。在 non-template 代码中重复十分明确，可以观察到两个函数或两个 class 间有重复，但在 template 代码中是比较隐晦的，因为只存在一份 template 源码。

- 对于有些 non-type 的模板参数，可以换成函数的参数来进行替换，如向量的乘法，向量是 n*1 的规格，如果对向量大小使用模板参数，然后不同规格的向量作为不同的类，这样就会可能导致生成 n 个乘法，此时可以将模板参数换成函数的参数，然后作为一个 base class 让所有向量继承。还有种方法是在模板类中存储一个指向向量类型的指针，因为向量内部可能就存储了对应的大小。

  后者可能比模板参数的性能更差，因为在模板参数中大小是一个编译期常量，因此可以使用常量的广传达达到最优化，但是从另一个角度看不使用模板参数的版本只拥有单一版本的乘法，这可以减少执行文件大小，因此降低程序的 working set（进程所使用的内存页）大小，并强化指令高速缓存区内的引用集中化，这些都可能使程序执行的更快速。哪一个占据主导地位只能通过尝试观察平台的行为以及面对代表性数据的行为

  再就是对象的大小，因为将乘法搬进 base class 内可能会根据实现增加每一个对象的大小，如 base class 有指向具体类的指针。

- type parameters 也会导致膨胀上，如很多平台上 int 和 long 有相同的表述，某些链接器会合并完全相同的函数实现码，但有些不会，后者意味着某些 template 可能会具象化为两个相同的版本。同样，在大多数平台上，所有指针类型都有相同的二进制表述，因此凡 template 持有指针者，往往应该对每一个程序函数使用唯一一份底层实现，这意味着某些成员函数操作 T\*，那么可以替换成 void\* 来完成实际工作

#### 45. 运用成员函数模板接受所有兼容类型

- 同一个 template 的不同具现体之间并不存在与生俱来的雇有关系，如 base class 和 derived class 具现化某个 template，产生的两个具现体并不带有 base-derived 关系。

**Templates 和泛型编程**

- 如现在想写一个智能指针类，现在想写一个构造函数接受另一个智能指针完成拷贝，但是不可能为每一个类型的指针都写一个构造函数，所以应该是为它写一个构造模板，也就是 member template，这个函数有时称之为泛化 copy 构造函数。当想让该 copy 函数提供更好的行为时，如可以通过一个 Derived 智能指针拷贝给一个 Base 智能指针，但是并不想一个通过一个 Base 智能指针拷贝给一个 Derived 智能指针，此时若可以提供一个 get 成员函数返回一个原始指针副本，那么就可以在该函数中使用成员初值列表来初始化，实现转换的约束行为

  ```c++
  template<typename T>
  class SmartPtr{
  public:
  	template<typename U>
  	SmartPtr(const SmartPtr<U>& ohter)
  		: heldPtr(other.get()) { }
  	T* get() const { return heldPtr; }
  private:
  T* heldPtr;
  }
  ```

- member template 并不改变语言规则，而语言规则中包括程序需要一个 copy 构造函数而没有声明它，编译器会自己生成一个。所以在 class 中声明泛化 copy 构造函数并不会阻止编译器生成自己的 copy 构造函数，所以若要控制 copy 的方方面面，那么需要同时声明泛化 copy 构造函数和正常的 copy 构造函数。

#### 46. 需要类型转换时请为模板定义非成员函数

- template 实参推导过程中从不会将隐式类型转换函数纳入考虑。函数调用过程确实能使用转换，但首先要知道转换函数的存在，而这必须要先为相关的 function template 推导出参数类型，然而 template 实参推导过程中并不考虑采纳通过构造函数发生的隐式类型转换。

  ```c++
  template<typename T>
  class Rational{
  public:
      // 包含从数值类型转换为 Rational 的隐式转换构造函数
  ...
  }
  
  template<typename T>
  const Rational<T> operator*(const Rational<T>& lhs, const Rational<T> rhs);
  
  void main(){
      Rational<int> r(1, 1);
      Rational<int> t = r * 1;		// 失败，因为不会进行隐式转换
  }
  ```

  template class 内的 friend 声明式可以指定某个特定函数，意味可以将想要使用隐式转换函数的那个模板函数成为 friend，此时就可以不依赖 template 的实参推导：

  ```c++
  template<typename T>
  class Rational{
  public:
  ...
  friend const Rational operator*(const Rational& lhs, const Rational& rhs);			//会链接失败
  }
  
  template<typename T>
  const Rational<T> operator*(const Rational<T>& lhs, const Rational<T> rhs);
  ```

  编译器知道调用的是对应的函数，所以通过了编译，但是函数只被声明在 Rational 内，并没有定义出来，所以最简单的方式就是将函数本体整合到声明式中

  ```c++
  template<typename T>
  class Rational{
  public:
  ...
  friend const Rational operator*(const Rational& lhs, const Rational& rhs){
      return Rational(lhs.numerator() * rhs.numerator(),  lhs.denominator() * rhs.denominator());
  }
  }
  ```

  这里虽然使用 friend，但和传统 friend 用于访问 class 的 non-public 成分的用途不同。为了让类型转换发生在实参上，需要一个 non-member 函数，为了让这个函数被自动具现化，需要将它声明在 class 内部，而在 class 内部声明 non-member 函数的唯一方法就是让它成为一个 friend。但是该做法还是会让 friend 函数成为 inline，若函数比较复杂不适合 inline，解决方法是让 friend 函数调用辅助函数

  ```c++
  template<typename T>
  class Rational{
  public:
  ...
  friend const Rational operator*(const Rational& lhs, const Rational& rhs){
      return doMultiplay(lhs, rhs);
  }
  }
  
  template<typename T>
  const Rational<T> doMultiply(const Rational<T>& lhs, const Rational<T>* rhs){
  	return Rational(lhs.numerator() * rhs.numerator(),  lhs.denominator() * rhs.denominator());
  }
  ```

  作为一个 template，doMultiply 不支持混合式乘法，但是它并不需要，因为在 fiend 的函数中已经支持了类型转换需要的东西。

- 在 class template 中，template class 名称可以被用来作为 template 和其参数的简称，所以只要在对应的 template class 中，可以用 class 代替 class\<T\> 

#### 47. 请使用 traits class 表现类型信息

- 迭代器 struct 之间的关系是有效的 is-a 关系：

  ```c++
  struct input_iterator_tag {};
  struct output_iterator_tag {};
  struct forward_iterator_tag : public input_iterator_tag {};
  struct bidirectional_iterator_tag : public forward_iterator_tag {};
  struct random_access_iterator_tag : public bidirectional_iterator_tag {};
  ```

- STL 中有一些工具性 template，其中有一个 advance 用来将某个迭代器移动给定距离，观点上只是做 iter += d 动作，但是只有 random access 迭代器才支持 += 操作，对于其他迭代器只能反复进行 d 次 ++ 或 --。

  为了判断 iter 是否是 random access 迭代器，就必须要知道 iter 的类型是否为 random access。traits 可以允许在编译期间取得某些类型信息，它是一个技术，也是 C++ 程序员共同遵守的协议，该技术的要求之一是对内置类型和用户自定义类型的表现必须一样好。traits 能够施行于内置类型意味着类型内的嵌套信息要出具了，因为无法将信息嵌套于原始指针内，因此类型的 traits 信息必须位于类型自身之外，标准技术是把它放进一个 template 及其一个或多个特化版本中，这样的 template 在标准程序库中有若干个，其中针对迭代器的命名为 iterator_traits

  ```c++
  template<typename IterT>
  struct iterator_traits;
  ```

  iterator_traits 的运作方式是针对每一个类型 IterT 在 struct iterator_traits\<IterT\> 内一定声明某个 typedef 名为 iterator_category，其用来确定 IterT 的迭代器分类，如

  ```c++
  template<...>
  class deque{
  public:
  	class iterator{
  	public:
  		typedef random_access_terator_tag iterator_category;
  		...
  	};
  	...
  };
  
  template<...>
  class list{
  public:
  	class iterator{
  	public:
  		typedef bidirectional_iterator_tag iterator_category;
  		...
  	}
  	...
  };
  
  template<typename IterT>
  struct iterator_traits{
  	typedef typename IterT::iterator iterator_category;
  	...
  };
  ```

  这在用户自定义类型上行得通，但对于指针不行，因为指针不可能嵌套 typedef，为了支持指针迭代器 iterator_traits 特别针对指针类型提供一个偏特化版本

  ```c++
  template<typename IterT>
  struct iterator_traits<IterT*>
  {
  	typedef random_access_terator_tag iterator_category;
  	...
  };
  ```

  此时根据给定的类型名称确定类型

  ```c++
  if(typeid(typename std::iterator_traits<IterT>::iterator_category) == typeid(std::random_access_iterator_tag))	// 这会导致编译问题，且 IterT 在编译器就可知了，何必等到运行期确定
  ```

  导致编译期错误是因为，当传入一个非 random access 迭代器时，会生成一个对应迭代器的版本，但对于非 random access 并不存在 += 运算符，所以报错

  所以为了调用针对不同类型实现不同步骤：

  ```c++
  template<typename IterT, typename DistT>
  void doAdvance(IterT& iter, DistT d, std::random_access_terator_tag)		// 用于 random access
  {
  	iter += d;
  }
  
  template<typename IterT, typename DistT>
  void doAdvance(IterT& iter, DistT d, std::bidirectional_iterator_tag)		// 用于 bidirectional access
  {
  	if(d >= 0) { while(d--) iter++; }
  	else { while(d++) iter--; }
  }
  
  template<typename IterT, typename DistT>
  void doAdvance(IterT& iter, DistT d, std::input_iterator_tag)		// 用于 input access
  {
  	if(d < 0){
  		throw std::out_of_range("");
  	}
  	while(d--) iter++;
  }
  
  template<typename IterT, typename DistT>
  void advance(IterT& iter, DistT d){
      doAdvance(iter, d, typename std::iterator_traits<IterT>::iterator_category());
  }
  ```

- 实现一个 traits class 的步骤：

  - 确认若干希望将来可取得的类型相关信息
  - 为该信息选择一个名称
  - 提供一个 template 和一组特化版本内涵希望支持的类型相关信息。

  使用一个 traits class：

  - 建立一组重载函数或函数模板，彼此之间的差异只在于各自的 traits 参数，令每个函数实现码与其接受的 traits 信息相应和。
  - 简历一个控制函数或函数模板，调用上述的函数并传递 traits class 所提供的信息

- 习惯上 traits 总是被实现为 struct 但是往往被称为 traits classes
- traits 广泛应用于标准程序库，除了 iterator_category 还提供另四份迭代器相关信息，还有 char_traits 用来保存字符类型的相关信息以及 numeric_limits 用来保存数值类型的相关信息
- TR1 导入许多新的 traits class 用以提供类型信息，包括 is_fundamental\<T\>，is_array\<T\> 以及 is_base_of\<T1,T2\>

#### 48. 认识 template 元编程

- Template metaprogramming（TMP，模板元编程）是编写 template-based C++ 程序并执行于编译期的过程，它是以 C++ 写成、执行于 C++ 编译器内的程序，一旦 TMP 程序结束执行，从 template 具现出来的若干 C++ 源码和往常一样被编译。

  TMP 有两个作用：

  - 让某些事情变的容易
  - 由于 TMP 执行于 C++ 编译器，因此可将工作从运行期转移到编译器，这可使有些在运行期才能侦测到的错误提前到编译期，也可能使 C++ 程序在每一方面都更高效：较小的可执行文件、较短的运行期、较少的内存需求。但将工作从运行期转移至编译器的另一个结果是，编译时间变长了。

- TMP 是一种函数式语言，并没有真正的循环构建，所以循环由递归完成，且它的递归不是正常种类，不涉及递归函数调用，而是递归模板具现化
- TMP 可以达成的一些目标：
  - 确保量度单位正确。如将一个质量变量赋值给一个速度变量是错误的，但将一个距离变量除以一个时间变量并将结果赋值给一个速度变量则正确。使用 TMP 可知在编译期确保程序所有量度单位的组合都正确，无论计算多复杂。
  - 优化矩阵运算。使用高级、与 TMP 相关的 template 技术，即所谓的 expression templates 就可能消除临时对象并合并循环，于是 TMP 软件使用较少的内存执行速度又有戏剧性提升。
  - 可以生成客户定制的设计模式实现品，如 Strategy、Observer、Visitor 等可以多种方式实现出来，运用所谓的 policy-based design 的 TMP-based 技术，可能产生一些 template 用来表述独立的设计选项，然后可以任意结合它们导致模式实现品带着客户定制的行为。该技术已被用来实现出智能指针的行为政策，用以在在编译期间生成数以百计不同的智能指针类型，它已经超越了编译工艺设计领域，更广义的称为 generative programming（殖生式编程）的一个基础。

### 8 定制 new 和delete

#### 49. 了解 new-handler 的行为