# C风格字符串
一段连续内存中的字符序列，以空字符 `\0`（值为 0）作为终止标记C风格字符串常用的操作函数      
```
内存布局: "hello"

地址:  0x100  0x101  0x102  0x103  0x104  0x105
内容:  'h'    'e'    'l'    'l'    'o'    '\0'
```    
**没有长度字段**，所有操作都必须从头扫描直到遇见 `\0`。这意味着获取长度的时  
间复杂度是 **O(n)**
## C风格字符串常用函数  
strlen(s) 用于求取字符串长度不包含“/0”,若字符串没正确初始化则会返回错误长度，  
求取长度是逐个扫描直到出现“\0”  
strcpy(dst, src) 拷贝函数，拷贝时不检查缓冲区大小  
strncpy(dst, src, n)拷贝最多n字节，可能拷贝时不追加'\0'  
strcat(dst, src) 拼接函数，依旧存在不检查缓冲区大小的问题  
strncat(dst, src, n)拼接n个字节，依旧需要注意缓冲区&结尾问题  
strcmp(s1, s2) 比较字符串，两个字符串都有结尾的'\0'->安全  
strchr(s, c) 查找字符  
strstr(s1,s2)查找子串  
strtok(s,delim)分词  
## C风格字符串的陷阱  
1.缓冲区溢出  
2.忘记结束'\0'  
3.指针悬挂->返回一个生命周期已经结束变量的指针  
4.char *str=new char[]和 char str1[]={'a','b','c','\0'}的sizeof，一个返回指针本身的大   
小，一个返回字符串的大小  
# Cpp风格的字符串  
`std::string` 是 `std::basic_string<char>` 的类型别名，是一个**封装了动态字符数组的类**。它在内部维护了三个核心信息：  
逻辑模型:
┌──────────────────────────────┐
│  std::string                 │
│  ├── char*  data_   (指针)   │
│  ├── size_t size_   (长度)   │
│  └── size_t capacity_(容量)  │
└──────────────────────────────┘  
获取长度 `size()` 是 **O(1)** 的，因为长度被显式存储。  
## cpp风格字符串的初始化  
```
#include <string>

std::string s1;                        // 空字符串
std::string s2 = "hello";              // 从 C 字符串字面量
std::string s3("hello");               // 同上
std::string s4(5, 'x');               // "xxxxx"
std::string s5(s2);                    // 拷贝构造
std::string s6 = s2;                   // 拷贝构造（不是赋值）
std::string s7(s2, 1, 3);             // 从 s2[1] 开始取 3 个字符 → "ell"
std::string s8{'h','e','l','l','o'};  // initializer_list

// ⭐ 嵌入 '\0' 的正确方式
std::string s9("hello\0world", 11);    // 包含 '\0'，size = 11
std::string s10 = "hello\0world"s;     // C++14 字符串字面量，size = 5（仍截断）
using namespace std::string_literals;
std::string s11 = "hello\0world"s;     // 注意：这个仍然在 \0 处截断！
                                       // 要用 s9 的方式才能保留 \0
```
## Cpp字符串的核心操作  
```
std::string s = "hello";

// --- 容量与大小 ---
s.size();        // 5（O(1)）
s.length();      // 5（与 size() 完全相同）
s.capacity();    // ≥ 5（当前分配的容量）
s.empty();       // false
s.reserve(100);  // 预分配容量，避免多次重分配
s.shrink_to_fit();// 请求释放多余容量

// --- 元素访问 ---
s[0];            // 'h'（不检查越界）
s.at(0);         // 'h'（越界抛 std::out_of_range）
s.front();       // 'h'
s.back();        // 'o'
s.data();        // const char*，C++17 起有非 const 重载

// --- 修改 ---
s += " world";          // 拼接
s.append("!");          // 拼接
s.push_back('?');       // 追加单个字符
s.insert(5, ",");       // 在位置 5 插入
s.erase(5, 1);          // 从位置 5 删除 1 个字符
s.replace(0, 5, "Hi");  // 替换
s.clear();              // 清空（capacity 不变）
s.pop_back();           // 删除末尾字符

// --- 查找 ---
s.find("world");        // 返回位置，找不到返回 std::string::npos
s.rfind("l");           // 从后往前找
s.find_first_of("aeiou");// 找第一个元音
s.find_last_not_of(" "); // 找最后一个非空格

// --- 比较 ---
s == "hello"            // true（重载了运算符）
s < "world"             // 字典序比较
s.compare("hello")      // 0（相等）

// --- 子串 ---
s.substr(0, 5);         // 返回新 string

// --- 数值转换 ---
std::to_string(42);     // "42"
std::stoi("42");        // 42
std::stod("3.14");      // 3.14
```  
# C风格字符串和Cpp风格字符串的互操作  
## Cpp->C  
```
std::string s = "hello";

// c_str()：返回 const char*，保证以 '\0' 结尾
const char* p = s.c_str();
printf("%s\n", p);       // ✅ 安全
strcpy(buf, s.c_str());  // 可以传给 C 函数

// data()：C++11 起也保证 '\0' 结尾
// C++17 起有非 const 重载，可以直接修改
char* p2 = s.data();     // C++17
```    
陷阱  
**生命周期陷阱**：`c_str()` 返回的指针在 `string` 被修改或销毁后失效  
```
const char* p = std::string("hello").c_str();
// 💥 p 指向的内存已被释放！悬垂指针！
```

## C->Cpp  

```
const char* cs = "hello";

std::string s1(cs);          // 构造
std::string s2 = cs;         // 隐式转换
s1.assign(cs);               // 赋值
s1 += cs;                    // 拼接

// 指定长度（处理含 '\0' 的数据）
std::string s3(cs, 5);       // 从 cs 取 5 个字符
```





