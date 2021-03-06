### 1.1 程序段

- 若使用 Lua 语言独立解释器，运行程序的方式为在命令行运行：

  ```lua
  lua 文件名
  如: lua hello.lua
  ```

- 除了将源码保存为文件，还可以在交互模式下运行独立解释器，在控制台输入 lua 直接进入交互模式：

  ```lua
  lua
  ```

  或者执行一个文件再进入交互模式，这对于调试很有用：

  ```lua
  lua -i 文件名
  ```

  交互模式中输入的每一条命令都会在按下回车键立即执行，Lua 语言解释器一般会把输入的每一行当作完整的程序块或表达式来解释执行。若解释器发现某一行不完整则会等待直到程序块或表达式被输入完整后再进行解释执行。

  可以通过 EOF 控制符（Windows），或调用操作系统库的 exit 函数退出交互模式：

  ```lua
  os.exit()
  ```

  交互模式中可以通过函数 dofile 立即执行一个文件，这在开发阶段可以边编辑边测试：

  ```lua
  dofile("文件名")
  ```

  Lua 5.3 版本开始，可以直接在交互模式下输入表达式，Lua 语言会输出表达式的值：

  ```lua
  math.pi / 4
  -- 对于某些函数调用不想输入返回的结果就在后面加上分号使其成为有效的命令但是非有效表达式
  io.flush();
  ```

  Lua 5.3 之前需要在表达式前加上 = ：

  ```lua
  = math.pi / 4
  ```

- Lua 语言执行的每一段代码（一个文件或交互模式下的一行）称为一个程序段，即一组命令或表达式组成的序列。其在大小上没有限制，Lua 语言的解释器可以支持非常大的程序段

### 1.2 一些词法规范

- Lua 语言中的标识符由任意字母、数字、下划线组成，但不能以数字开头。”下划线+大写字母“ 组成的标识符通常被 Lua 用作特殊用途，应避免将其用作其他用途。通常会将 “下划线+小写字符” 用作哑变量。

- Lua 语言的保留字：

  ```lua
  and			break		do			else		elseif		end			false		goto		for			function
  if			in			local		nil			not			or			repeat		return		then		true
  until		while
  ```

- Lua 语言使用两个连续的连字符表示单行注释的开始（--）。只用两个连续的连字符加两对连续左方括号表示长注释或多行注释的开始，直到两个连续的右括号为止（--[[ ]]）。

  一个常用技巧使用：

  ```lua
  --[[
  代码段
  --]]
  ```

   来注释代码段，当要启动代码段时，只需要在左边注释前面加上一个连字符就可以了，该连字符将代码段上下变成两个独立的单行注释：

  ```lua
  ---[[
  代码段
  --]]
  ```

- Lua 语言中连续语句之间的分隔符不是必须的，若需要可使用分号进行分隔。

- Lua语言中表达式之间的换行也和其他空格符作用一样，但换行可读性更好

### 1.3 全局变量

- Lua 语言中，全局变量无须声明即可使用，使用未初始化的全局变量会得到 nil 而不会出错。当将 nil 赋值给全局变量时，Lua 会回收该全局变量，即不会区分未初始化变量和全局变量

### 1.4 类型和值

- Lua 是一种动态类型语言，其没有类型定义，每个变量都带有自身的类型信息，任何变量都可以包含任何类型的值，一般情况下在一个变量用作不同类型会导致代码可读性不佳。

- Lua 语言有 8 种基本类型：nil、boolean、number、string、userdata、function、thread、table。使用函数 type 可以获得一个变量对应的类型名称，其返回一个字符串。

  userdata 类型允许把任意的 C 语言数据保存在 Lua 语言变量中，该类型除了赋值和相等性测试之外没有其他预定义的操作，其用来表示由应用或 C 语言编写的库所创建的新类型

#### 1.4.1 nil

- nil 是一种只有一个 nil 值的类型，主要作用是与其他值进行区分，Lua 语言使用它来表示无效值

#### 1.4.2 Boolean

- 具有 true 和 false 两个值，但在 Lua 语言中该类型并非是用于条件测试的唯一方式，任何值都可以表示条件，除 false 和 nil 外的其他类型的所有值都表示真。

- Lua 支持的逻辑运算符

  - and：若第一个操作数为真返回第二个操作数，否则返回第一个操作数
  - or：若第一个操作数为真则返回第一个操作数，否则返回第二个操作数
  - not：若操作数为真返回 false，否则返回 true

  and 和 or 都遵循短路求值原则

- 一些好用的写法：

  ```lua
  -- 等价 if not x then x = v end
  x = x or v
  -- 当 b 不为假时等价 a ? b : c
  a and b or c
  ```

### 1.5 独立解释器

- 独立解释器是一个可以直接使用 Lua 语言的小程序，由于源文件名为 lua.c 又称 lua.c，由于可执行文件为 lua 又称 lua。

- 若源代码第一行以井号（#）开头，那么解释器在加载该文件时会忽略这一行，这个特征主要是为了方便在 POSIX 系统中将 Lua 作为一种脚本解释器来使用。

- lua 命令的完整参数如：

  ```lua
  lua [operations] [script [args]]
  ```

  - -e：允许直接在命令行输入代码

    ```lua
    -- POSIX 系统需要使用双引号将代码包围起来，以防止 Shell 错误的解析括号
    lua -e print(1)
    ```

  - -l：加载库

    ```lua
    lua -llib
    ```

  - -i：在运行完其他命令行参数后进入交互模式

  解释器在处理参数前会查找 LUA_INIT_版本号（如LUA_INI_5_3）的环境变量，若没有则查找 LUA_INIT 环境变量。若其中任意一个存在，并其内容为 @filename，则解释器会运行相应文件，若它们存在但内容不以 @ 开头，则认为其包含代码，会进行解释执行，通过这个可以预先加载程序包、修改路径、自定义函数、修改函数名、删除函数等等。

  可以通过全局变量 arg 获取解释器传入的参数。编译器在运行代码前会创建一个 arg 的表，存储了所有命令行参数，索引 0 为脚本名，1 为第一个参数，而参数之前的选项都位于负数索引上面。Lua 语言也支持可变长参数，可以通过可变长参数表达式获取，... 表示传递给脚本的所有参数。