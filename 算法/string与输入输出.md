## 标准输入输出



## 文件输入输出

```cpp
#include <fstream.h>
ifstream filein("data.in");   // 定义一个文件输入流
ofstream fileout("data.out"); //cout<< 写成 fileout<<
filein.eof() //文件到末尾,返回非零值
```

## string

```cpp
#include <string>
using namespace std;
```

功能	C++ string	C字符数组
定义字符串	string str;	char str[100];
单个字符输出	str[i] / str.at(i)	str[i]
字符串长度	str.length() / str.size()	strlen(str)
读取一行	getline(cin,str)	gets(str)
赋值	str = “Hello”;	strcpy(str,“Hello”);
连接字符串	str = str + “Hello”	strcat(str,“Hello”);
比较字符串	str == “Hello”;	strcmp(str,“Hello”);

要使用strlen()、strcpy()函数需要包含C语言的字符串操作函数头文件：

```cpp
#include <string.h>
using namespace std;
//上述两行代码等价于下面一行代码
#include <cstring>
```

### 其他方法

```cpp
//改变容量
str.resize(100);

//查找
string str = "aaaaddddssdfsasdf";  
size_t pos = str.find("ssdf", 3);//注意pos的数据类型string::size_type
//如果没找到，返回一个特殊的标志npos
// 可以用if(pos != string::npos)则表示找到。

//子串
string str2 = str.substr(pos, num);//[pos,pos+num)
```



## string与字符数组

| 函数    | 功能                                             |
| ------- | ------------------------------------------------ |
| c_str() | 返回一个以‘/0’结尾的字符数组                     |
| data()  | 以字符数组的形式返回字符串内容，但并不添加’/0’   |
| copy()  | 字符串的内容复制或写入既有的c_string或字符数组内 |

## string流操作

```cpp
#include <sstream>
using namespace std;
stringstream ss;//io
istringstream is;//i
ostringstream os;//o

// 字符串流清零，且清空缓存，或重新赋值
ss.str("");
ss.str("sdafsdf")；

//转成string输出
cout<<ss.str(); 

// 清空流的错误状态
ss.setstate(std::ios::eofbit);//设置流的状态标志位
std::cout << ss.rdstate() << std::endl;//获取当前流的状态标志位 结果为1
ss.clear();
std::cout << ss.rdstate() << std::endl;// 结果为0

//用来读取文件
string str;
ifstream in;
in.open("Hello.txt");
//读取文件的缓冲内容到数据流中
stringstream ss;
ss << in.rdbuf();
in.close();//关闭文件
str = ss.str();//将stringstream流中的数据赋值给string类型字符串
const char* p = str.c_str();//将字符串内容转化为C_string类型
```



