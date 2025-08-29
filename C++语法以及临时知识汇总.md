## Lambda表达式
[中文网络blog](https://www.cnblogs.com/pzhfei/archive/2013/01/14/lambda_expression.html)  
[语法说明](https://zh.cppreference.com/w/cpp/language/lambda)  
要点：  
### 语法  
[ captures ] <tparams>(可选)(C\++20) ( params ) specifiers(可选) exception attr -> ret requires(可选)(C++20) { body }	(1)	  
[ captures ] ( params ) -> ret { body }	(2)	  
[ captures ] ( params ) { body }	(3)	  
[ captures ] { body }	(4)	  

### 关于captures说明
[]        //未定义变量.试图在Lambda内使用任何外部变量都是错误的.  
[x, &y]   //x 按值捕获, y 按引用捕获.  
[&]       //用到的任何外部变量都隐式按引用捕获  
[=]       //用到的任何外部变量都隐式按值捕获  
[&, x]    //x显式地按值捕获. 其它变量按引用捕获  
[=, &z]   //z按引用捕获. 其它变量按值捕获  

+ 值拷贝，在lambda函数创建之时，对capture的变量进行复制。
+ 引用捕获，只捕获对象引用，不复制。
+ 表达式捕获，右值捕获。c++14.
```
    auto add = [v1=1, v2=std::move(important)](int x, int y)    {
        return v1+(*v2)+x+y;
    };
```
+ **lambda函数的参数列表中可以使用auto作为参数类型**

### std::function<>与std::bind()

### 例子
```
// 用于循环迭代中
vector<int> v;  
v.push_back( 1 );  
v.push_back( 2 );  
//...  
for_each( v.begin(), v.end(), [] (int val)  
{  
    cout << val;  
} );  
```
```
AddressBook global_address_book;  
  
vector<string> findAddressesFromOrgs ()  
{  
    return global_address_book.findMatchingAddresses(   
        // we're declaring a lambda here; the [] signals the start  
        [] (const string& addr) { return addr.find( ".org" ) != string::npos; }   
    );  
}  
```
```
    auto func = [] () { cout << "Hello world"; };  
    func(); // now call the function  
```

## 标准库与之前知识不同的点
+ std::map[i] 当key==i的项不存在是会自动创建，之前会直接崩溃。

## std::move [这个的使用还是应该慎重]
1. std::move的优化需要类支持move construct和move operator=
2. std::move调用后，原alue可能空了。具体要看类的实现。
3. [例子](https://blog.csdn.net/swartz_lubel/article/details/59620868)
4. [关联-move constructor and move operator=](https://docs.microsoft.com/en-us/cpp/cpp/move-constructors-and-move-assignment-operators-cpp?redirectedfrom=MSDN&view=msvc-160)

## Structured binding
C++版本：C\++17
```
std::tuple<int, double, std::string> f() {
return std::make_tuple(1, 2.3, "456");
}
auto [x, y, z] = f();
```

## initializer_list
```
#include <initializer_list>
    class MagicFoo {
    public:
        std::vector<int> vec;
        MagicFoo(std::initializer_list<int> list) {
            for (auto it = list.begin(); it != list.end(); ++it) {
                vec.push_back(*it);
            }
        }
    };
    int main() {
        MagicFoo magicFoo = {1, 2, 3, 4, 5};
    }
```
### 定义已std::initializer_list为参数的构造函数因谨慎
因为std::initializer_list为参数的构造函数将屏蔽掉大部分其他的构造函数，当用户使用{}进行对象初始化时。

## std::optional<T>
[介绍](https://blog.csdn.net/haokan123456789/article/details/136099479)

## std::any
是一个可用于任何类型单个值的类型安全的容器。std: any是一种值类型，它能够更改其类型，同时仍然具有类型安全性。也就是说，对象可以保存任意类型的值，但是它们知道当前保存的值是哪种类型。在声明此类型的对象时，不需要指定可能的类型。
小结
std::any a = 1;: 声明一个any类型的容器，容器中的值为int类型的1
a.type(): 得到容器中的值的类型
std::any_cast(a);: 强制类型转换, 转换失败可以捕获到std::bad_any_cast类型的异常
has_value(): 判断容器中是否有值
reset(): 删除容器中的值
std::any_cast(&a): 强制转换得到容器中的值的地址
[介绍](https://www.cnblogs.com/gnivor/p/12793239.html)

## 时间相关
```
// 获取当前系统时间的毫秒数
uint64_t now = std::chrono::duration_cast<std::chrono::milliseconds>(std::chrono::system_clock::now().time_since_epoch()).count();
// 表示时间间隔，毫秒，主要用于sleep或者wait_for之类的函数
std::chrono::milliseconds(minInterval)
// 关于time_point用法
const std::chrono::time_point<std::chrono::system_clock> now =
        std::chrono::system_clock::now();
const std::chrono::time_point<std::chrono::steady_clock> start =
        std::chrono::steady_clock::now();
```

## auto 与 decltype
**注意：auto不可用于函数参数，不可以定义auto数组变量**
```
auto i = 5;     // i as int
auto arr = new auto(10);    //arr as int*
auto auto_aar2[10] = {arr};     // 编译不通过，不可以定义auto数组变量

decltype(x);        // 返回x的类型
auto x = 1;
auto y = 2;
decltype(x+y) z;

std::is_same<decltype(x), decltype(z)>::value == true;
```
### auto的优点
1. 可以避免变量未初始化的问题。auto声明变量必须初始化。
2. 变量定义自动推导类型。
3. 可以直接声明闭包（比如函数）类型的变量
4. 可以编码int等类型截断。

### auto可能出问题的场景
+ auto用于proxy类型可能有问题。如. auto someVar = expression of "invisible" proxy class type;
    > 如何发现proxy class，则要对库的接口更详细的了解

## trailing type inference
实现模板中根据参数类型返回不同的返回值类型
```
// C++11 之前
template<typename R, typename T, typename U>
R add(T x, U y)
{
    return x+y;
}

// C++11 版本 不再需要了解T、U类型的加法运算得出的是什么类型的结果
template<typename T, typename U>
auto add2(T x, U y) -> decltype(x+y)
{
    return x+y;
}

// C++14版本
template<typename T, typename U>
auto add3(T x, U y)
{
    return x+y;
}
```
==decltype(auto)  ???==

## constexpr
**从c++17开始，constexpr可以用于if语句中**
```
template<typename T>
auto printf_type_ifno(const T& t)
{
    if constexpr(std::is_integral<T>::value) {
        return t+1;
    }
    else
    {
        return t+0.1;
    }
}
```

## 迭代语法（简化）
```
    std::vector<int> vec = {1, 3, 5, 7, 9};
    std::cout << "vec: ";
    for(auto iVec : vec)
    {
        std::cout << iVec << " ";
    }
    std::cout << std::endl;
    std::cout << "vec: ";
    for(auto &iVec : vec)
    {
        iVec += 1;
    }
    for(auto iVec : vec)
    {
        std::cout << iVec << " ";
    }
    std::cout << std::endl;
```

## template extern语法
声明编译时不用在此处实例化模板
```
template class std::vector<bool>; // force instantiation
extern template class std::vector<double>; // should not instantiation in current file
```

## using
使用using实现模板的别名，类似typedef。
```
typedef int (*process)(void*);
using NewProcess = int(*)(void*);   // 使用using定义函数指针

template<typename T>
using TrueDarkMagic = MagicType<std::vector<T>, std::string>;   // 使用using定义模板TrueDarkMagic
```

## 模板参数使用默认类型
这样在使用模板时就不需要指定类型
```
template<typename T = int, typename U = int>
auto add_1(T t, U u)
{ 
    return t+u;
}

std::cout << "template default paramater: 2+4=" << add_1(2, 4) << std::endl;
```

## 模板参数个数、类型可变
支持定义参数个数、类型可变化的模板。类似于printf中的可变参数。
#### 如何展开模块参数列表中的参数
1. 递归展开
```
template<typename T0>
void printf1(T0 value) {
    std::cout << value << std::endl;
}
template<typename T, typename... Ts>
void printf1(T value, Ts... args) {
    std::cout << value << std::endl;
    printf1(args...);
}
```
2. 递归并判断sizeof..展开(C++17)
```
template<typename T0, typename... T>
void printf2(T0 t0, T... t) {
    std::cout << t0 << std::endl;
    if constexpr (sizeof...(t) > 0) printf2(t...);
}
```
3. 使用std::initializer_list展开
```
template<typename T, typename... Ts>
auto printf3(T value, Ts... args) {
    std::cout << value << std::endl;
    (void) std::initializer_list<T>{([&args] {
    std::cout << args << std::endl;
}(), value)...};
}
```

#### Fold expression
将模板的可变参数应用于公式，操作符
```
// fold expression C++17 
template<typename ... T>
auto add3(T ... t)
{
    return (t + ...);
}
// fold expression
    std::cout << "add3: " << add3(1,2,3,4,5,6,7,8,9,10) << std::endl;
```

## 非类型（常量）模板参数自动推导
在模板参数列表中使用auto
```
// 此时T是作为常量值对待的
template<auto T>
void print1() 
{
    std::cout << "T: " << T << std::endl;
}

    // Non-type template param c++17
    print1<10>();
    // print1<3.3>();    //编译失败
    // print1<"222">());    //编译失败
```

## 对象--代理构造函数
在一个构造函数中调用同一个类的另一个构造函数
```
//delegate constructor
class DeleConstructor
{
public:
    DeleConstructor()
        : val1(1)
    {

    }

    DeleConstructor(int val2)
        : DeleConstructor()     // delegate the default constructor
        // , m_val2(val2) //加这行编译不通过
    {
        m_val2 = val2;
    }

    int val1 = 0;
    int m_val2 = 0;
};
```
## 对象--继承构造函数 using
```
// 构造函数继承自其基类
class SubClass : public DeleConstructor
{
public:
    using DeleConstructor::DeleConstructor;
};
```
## 对象--override关键字
使用此关键字表面当前方法为重写基类中定义的方法。添加此方法可以在修改了基类的方法声明时，在编译时检测到子类的未对应的修改。否则可能导致错误
```
// 构造函数继承自其基类
class SubClass : public DeleConstructor
{
public:
    using DeleConstructor::DeleConstructor;

    virtual int foo(int v) override
    {
        return v;
    }

    // 此方法编译不通过，因为在基类中没找到对应方法可以override。但是去掉override时可以编译通过
    // virtual int foo(double d) override
    // {
        // return d;
    // }
};
```
## 对象--final关键字
使用final关键字表明类或者类的方法不再可以继承或者重载。
```
struct Base {
    virtual void foo() final;
};
struct SubClass1 final: Base {
}; // legal
struct SubClass2 : SubClass1 {
}; // illegal, SubClass1 has final
struct SubClass3: Base {
    void foo(); // illegal, foo has final
};
```
## 对象--显式声明使用默认函数或者禁止默认函数
c\++中如果用户不定义默认构造函数、赋值构造函数、复制构造函数、new、delete操作符，则编译器将自动为类生成这些函数。C++11支持显式告诉编译器是否需要和使用这些默认函数。
```
// explicit delete default function
class Magic{
public:
    Magic() = default;  // explicit let compiler use default constructor
    Magic& operator=(const Magic& robj) = delete;   // explicti declare refuse constructor
};
```

## 强类型的枚举
使用enum class关键字定义强类型的枚举。不在可以直接将枚举值与int进行比较和转换
```
// Strongly typed enum 
enum class new_enum : unsigned int {
    VALUE_1,
    VALUE_2,
    VALUE_3 = 100,
    VALUE_4 = 100,
};

template<typename T>
std::ostream& operator<<(typename std::enable_if<std::is_enum<T>::value, std::ostream>::type& stream, T e)
{
    return stream << static_cast<typename std::underlying_type<T>::type>(e);
}
```
## 左值lvalue与右值rvalue
* 左值： is the value to the left of the assignment symbol. To be
precise, an lvalue is a persistent object that still exists after an expression (not necessarily an assignment
expression).
* 右值：the value on the right refers to the temporary object that no longer exists
after the expression ends.
* 右值的引用：To get a xvalue, you need to use the declaration of the rvalue reference: T &&, where T is the type.
* **std::move** the std::move method to unconditionally convert lvalue parameters to rvalues.

### 右值引用的问题
```
// rvalue reference
void reference(int& v)
{
    std::cout << "lvalue v=" << v << std::endl;
}
void reference(int&& v)
{
    std::cout << "lvalue v=" << v << std::endl;
}

    reference(1);   //输出lvalue v=1
    int i = 2;
    reference(i);   //输出lvalue v=2
```
#### std::move与std::forward
+ std::move无条件将参数转换为右值，没有任何的运行时代码（性能消耗）
+ std::forward在特定条件下将参数传递给下一个。std::forward只在传递的参数是右值时才会向前传递返回右值。

### 容器--std::array
+ 与std::vector的区别。  
std::array是固定大小的，而std::vector是可变大小的。而且std::vector的capacity如果变得很大，删掉元素而不调用shrink_to_fit()的话，内存还是占用着的。
+ 与传统数组的区别  
支持封装方法，如获取size，判断是否为空。以及应用模板中与容器相关的算法，如std::sort。
+ 
```
// array size must be constexpr
constexpr int len = 4;
std::array<int, len> arr = {1, 2, 3, 4};
```

### 容器--std::forward_list
与std::list相同。使用单向链表实现。没有size()方法

### 容器--无序（unordered）容器
包含：std::unordered_map, std::unordered_multimap, std::unordered_set,  std::unordered_multiset。
只去重，而不排序。新添加的元素位于后面，前面的同一个key的元素将移动。

### 容器--std::tuple
重要的方法：
+ std::make_tuple 创建tuple
+ std::get\<i>()  获取第i个成员
+ std::tie(args ...)  unpacking

#### std::variant<T...>用于支持运行时使用get\<i>获取tuple成员
```
template <size_t n, typename... T>
constexpr std::variant<T...> _tuple_index(const std::tuple<T...>& tpl, size_t i) {
    if constexpr (n >= sizeof...(T))
        throw std::out_of_range("��.");
    if (i == n)
        return std::variant<T...>{ std::in_place_index<n>, std::get<n>(tpl) };
    return _tuple_index<(n < sizeof...(T)-1 ? n+1 : 0)>(tpl, i);
}
template <typename... T>
constexpr std::variant<T...> tuple_index(const std::tuple<T...>& tpl, size_t i) {
    return _tuple_index<0>(tpl, i);
}
template <typename T0, typename ... Ts>
std::ostream & operator<< (std::ostream & s, std::variant<T0, Ts...> const & v) {
    std::visit([&](auto && x){ s << x;}, v);
    return s;
}

int i = 1;
std::cout << tuple_index(t, i) << std::endl;
```

### std::shared_ptr
+ std::make_shared<>() 创建shared_ptr
+ shared_ptr::reset() 当前对象引用-1
+ shared_ptr::use_count() 获取引用数

#### 类型转换static_cast、dynamic_cast、const_cast对应
```
shared_ptr<bad_exception> sp2 = dynamic_pointer_cast<bad_exception>(sp1); 
shared_ptr<std::exception> sp3 = static_pointer_cast<std::exception>(sp2);  
const_pointer_cast<T>()
```

### std::unique_ptr 不支持复制的指针
因为该指针对象不能赋值给其他对象，因此更加安全。
+ 可以使用make_unique()创建。c++14
+ 可以使用std::move(p1)将p1赋值给其他unique_ptr。move之后p1为NULL。
```
std::unique_ptr<int> pointer = std::make_unique<int>(10); // make_unique was introduced in C++14
std::unique_ptr<int> pointer2 = pointer; // illegal

std::unique_ptr<Foo> p1(std::make_unique<Foo>());
std::unique_ptr<Foo> p2(std::move(p1));
```

### std::weak_ptr 弱引用指针
+ 用于解决对象相互使用share_ptr持有对方，而导致无法析构释放的问题。
+ expired() 判断weak_ptr对应的智能指针释放过期。

### noexcept关键字--不抛出异常
两种用法：
1. 放在方法、函数后面表示改方法不会抛出异常。该函数内调用的函数如果抛出异常，此异常不会继续往上抛出。
2. 用于判断一个方法、函数是否抛出异常
```
void no_throw() noexcept {
    return; 
}
std::cout << std::boolalpha
<< "may_throw() noexcept? " << noexcept(may_throw()) << std::endl;

```

### R\""字符串常量
使用R放在\""之前，表明是字符串，不用处理字符的转义
```
const char* path = R"()"
```

### 内存对齐--alignof、alignas
+ 使用alignas声明struct、class的对齐方式。alignas()中声明的对齐字节数必须大于或者等于结构体内所有成员中最大字节数的成员。
+ 使用alignof获取对象的对齐字节数
```
// memory align
struct Storage
{
    char a;
    int b;
    double d;
    long long ll;
};

/*
struct alignas(int) AlignStorage
{
    char a_1;
    char a;
    int b;
    int d;
    short ll;
};
*/
struct alignas(std::max_align_t) AlignStorage
{
    char a_1;
    char a;
    int b;
    long d;
    double ll;
    long long ll2;
};
    std::cout << "alignof Storyge: " << alignof(Storage) << std::endl;
    std::cout << "alignof AlignedStorage: " << alignof(AlignStorage) << std::endl;
 
```

### thread and future
可以使用future获取线程的返回值，这样可以产生类似c#的await的用法。
```c++
// thread and future
void testThreadAndFuture()
{
    std::packaged_task<int()> pkgTask([](){ return -100; });
    std::future<int> ft = pkgTask.get_future();

    std::thread(std::move(pkgTask)).detach();
    std::cout << "waiting...";
    ft.wait();

    std::cout << "Task thread done. Result: " << ft.get() << std::endl;
}
```

### 运行时打印对象，模板参数的类型
使用boost库的TypeIndex库。
```
boost::typeindex::type_id_with_cvr<T>().pretty_name();
boost::typeindex::type_id_with_cvr<decltype<param>>().pretty_name();
```

### enum class
使用enum class声明枚举的好处：
1. 枚举值的作用域更小了，不会全局泛滥
2. 枚举值是强类型的，不再可隐式转换成int
3. 枚举class支持预声明。声明与定义分离。有助于编译
```
enum class Status;
void onStatus(Status st);
```

### 将不想对外公开的方法声明delete
```
class A
{
public:
    A(const A&) = delete;
    A& operator(const A&) = delete;
}
```
**普通的函数也可以声明为delete**
```
bool isLuckNum(int i);
bool isLuckNum(char c) = delete;    //此时 isLuckNum('c'); 将编译不通过
```
**模板的实例可以delete**

### std库中关于类型操作的模版
+ std::decay<T>::type 将T的所有类型修饰去掉，如cost，引用&，volatile。
+ std::is_same<T1,T2>::value 判断两种类型是否一样
+ std::is_base_of<T1,T2>::value 判断T1是否为T2的基类。
+ std::remove_reference_t<T>::value 如果T是引用类型，则得到该类型移除引用后的类型。
+ std::is_integral<T>::value 判断类型T是否为整数型。

#### std::is_enable_if<condition>::value
指定只有condition为true时模版才起效，如果condition为false，此时模版想没有定义一样。

### 关于universal reference
A universal reference isn’t a new kind of reference, it’s actually an rvalue reference in a context where two conditions are satisfied:
+ Type deduction distinguishes lvalues from rvalues. Lvalues of type are deduced to have type , while rvalues of type yield as their deduced type.
+ Reference collapsing occurs.

### 线程、feature、并发相关的API
+ 使用std::async()的性能将优于使用线程。可能是使用了系统的线程池
+ std::async()是基于任务（Task）的。
+ std::async()支持获取函数异步执行完成之后的返回值。

## 标准库 std
### 多线程--互斥量、锁
```
std::shared_mutex
std::mutex
std::shared_lock<>
std::unique_lock<>
```
### std::call_once
```
// 使用std::call_once确保特定函数代码只会执行一次, 且线程安全
static std::once_flag s_flag;
void registerBackend() {
    std::call_once(s_flag, [&]() {
    });
//使用std::call_once确保静态变量或者全局变量只初始化一次, 且线程安全
OpenCLSymbolsOperator* OpenCLSymbolsOperator::createOpenCLSymbolsOperatorSingleInstance() {
    static std::once_flag sFlagInitSymbols;
    static OpenCLSymbolsOperator* gInstance = nullptr;
    std::call_once(sFlagInitSymbols, [&]() {
        gInstance = new OpenCLSymbolsOperator;
    });
    return gInstance;
}
```

### 正则表达式std::regex
[C++标准库之std::regex类的使用--很好的资料](https://blog.csdn.net/qq_28087491/article/details/107608569)
[中文匹配](https://www.bytezonex.com/archives/7bGD1suE.html)
**注意：如果处理中文等，最好是使用std::wstring std::wsmatch**
例子
```
std::vector<std::string> splitIntoSentencesImproved(const std::string& text) {
    std::vector<std::string> sentences;
    std::regex wordRegex(R"(\S+)"); // 匹配单词
    std::smatch matches;

    std::string::const_iterator searchStart(text.cbegin());
    std::string sentence;
    while (std::regex_search(searchStart, text.cend(), matches, wordRegex)) {
        std::string word = matches.str(0);
        sentence += word + " ";

        // 检查句子结束
        if (word.back() == '.' || word.back() == '?' || word.back() == '!') {
            if (!isAbbreviation(word)) { // 排除缩写词
                sentences.push_back(sentence);
                sentence.clear();
            }
        }

        searchStart = matches.suffix().first;
    }

    // 处理最后一个句子
    if (!sentence.empty()) {
        sentences.push_back(sentence);
    }

    return sentences;
}
```

## 零散知识汇总
+ 使用std::cerr可以将调试信息输出到googletest的单元测试程序里web

### cmake中指定c,c++编译器[参考](https://stackoverflow.com/questions/17275348/how-to-specify-new-gcc-path-for-cmake)
+ 方法1 设置 CMAKE_C_COMPILER CMAKE_CXX_COMPILER 
```
set(CMAKE_C_COMPILER /usr/bin/clang CACHE PATH "" FORCE)
set(CMAKE_CXX_COMPILER /usr/bin/clang++ CACHE PATH "" FORCE)
```
+ 方法2 设置环境变量CC、CXX
```
export CC=/usr/local/bin/gcc
export CXX=/usr/local/bin/g++
cmake /path/to/your/project
make
```

### 交叉编译
[linaro交叉编译工具链下载与使用笔记](https://blog.csdn.net/yjkhtddx/article/details/134676016)
例子，cmake文件。比如已下载好了工具链gcc-linaro-5.5.0-2017.10-x86_64_arm-linux-gnueabihf。
**重点：设置CMAKE_SYSTEM_PROCESSOR、CMAKE_C_COMPILER、CMAKE_CXX_COMPILER、CMAKE_LINKER**
```
cmake_minimum_required(VERSION 3.10)

project(main)

set(CMAKE_SYSTEM_PROCESSOR arm)
set(CROSS_CHAIN_PATH /home/zhuqingquan/project/gcc-linaro-5.5.0-2017.10-x86_64_arm-linux-gnueabihf)
set(CMAKE_C_COMPILER "${CROSS_CHAIN_PATH}/bin/arm-linux-gnueabihf-gcc")
set(CMAKE_CXX_COMPILER "${CROSS_CHAIN_PATH}/bin/arm-linux-gnueabihf-g++")
set(CMAKE_LINKER "${CROSS_CHAIN_PATH}/bin/arm-linux-gnueabihf-ld")

add_executable(main main.cc)
```
#### cmake中推荐使用CMAKE_TOOLCHAIN_FILE
```shell
cmake -DCMAKE_TOOLCHAIN_FILE="embedded.cmake" -S . -B build -DCMAKE_BUILD_TYPE:String=Debug
```
相应的TOOLCHAIN_FILE的内容如下
```embedded.cmake
set(CMAKE_SYSTEM_PROCESSOR arm)
set(CROSS_CHAIN_PATH ${CMAKE_SOURCE_DIR}/../tools/gcc-linaro-5.5.0-2017.10-x86_64_arm-linux-gnueabihf)
set(CMAKE_C_COMPILER "${CROSS_CHAIN_PATH}/bin/arm-linux-gnueabihf-gcc")
set(CMAKE_CXX_COMPILER "${CROSS_CHAIN_PATH}/bin/arm-linux-gnueabihf-g++")
set(CMAKE_LINKER "${CROSS_CHAIN_PATH}/bin/arm-linux-gnueabihf-ld")
```

## 编译链接错误
### linux下的静态库链接顺序问题
linux在将多个静态库链接成可执行程序或者so时，有时可能存在liba.a调用了libb.a的函数，而libb.a也调用了liba.a中的函数，此时链接时-la -lb则会报a找不到b的函数。
解决方法：
1. 重复写，比如 -lb -la -lb
2. 使用 -Wl,--start-group  -la -lb -Wl,--end-group  ==注意：-Wl,后面没有空格==

### libstdc++.so.6版本太旧的问题
问题：如果某个库是使用新版的libstdc++.so.6版本构建的so，此so拷贝到低版本的机器上，则会链接时报错：undefined reference to `std::condition_variable::wait(std::unique_lock<std::mutex>&)@GLIBCXX_3.4.30'
解决方式：从有更新版本的libstdc++.so.6的机器上将文件拷贝到编译报错的机器上。
```
# 用于判断当前机器的版本是否满足要求
strings /usr/lib/x86_64-linux-gnu/libstdc++.so.6 | grep GLIBCXX
cp /mnt/e/libstdc++.so.6 /usr/lib/x86_64-linux-gnu/
```

### 在头文件中定义的函数链接时导致的重复定义问题
当函数在头文件（xxx.h）中实现。然后有多个cpp文件通过`#include "xxx.h"`包含此文件，则头文件中实现了的方法可能在链接时产生函数重复定义的错误。
此时可以将头文件中定义的函数声明为static函数。