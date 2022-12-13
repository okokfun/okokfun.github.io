正则表达式是C++11时加入标准库👉[正则表达式](https://zh.cppreference.com/w/cpp/regex)

# 匹配规则：

| 字符匹配规则 | 释义               |
| ------------ | ------------------ |
| `\w`         | 匹配任何字母字符   |
| `\W`         | 匹配任何非字母字符 |
| `\d`         | 匹配任何数字字符   |
| `\D`         | 匹配任何非数字，   |
| `\s`         | 匹配任何空格       |
| `\S`         | 匹配任何非空白，   |

| 正则表达式量词 | 指定数量           |
| -------------- | ------------------ |
| *              | 0 或更多           |
| +              | 1 或更多           |
| ？             | 0 或 1             |
| {n}            | 正好是n            |
| {n, m}         | 介于n和m之间（含） |
| {n,}           | 至少n              |

| 字符 | 指定                                |
| ---- | ----------------------------------- |
| X\|Y | 字符X或Y                            |
| \Y   | 特殊字符Y作为文字（换句话说，转义） |
| \n   | 新一行                              |
| \r   | 回车                                |
| \t   | Tab                                 |
| \0   | Null                                |
| \xYY | YY对应的十六进制字符                |
# 例子
反转自定义字符：例如，字符类`[^aeiou]`包括所有非元音字符。
`c\w{n, m}t`:匹配以`c`开头，以`t`结尾，中间匹配`n ~ m`数量的字母字符；
也可以改为`c\w*t`为匹配以`c`开头，以`t`结尾，中间匹配任意数量的字母字符；
例子：
`\w{n}`:匹配n个字母字符

# 模板函数
## regex_match
`regex_match`尝试匹配一个正则表达式到整个字符序列
    匹配的函数模版有7个，浓缩下来有两个
```cpp
bool regex_match(first, last, [results], regex, [flags]);

bool regex_match(str, [results], regex, [flags]);
```
参数：
`first`, `last`: 应用 regex 到的目标字符范围，以迭代器给定
`results`: 匹配出来的结果
`str`: 目标字符串，以空终止 C 风格字符串给出
`s`: 目标字符串，以 std::basic_string 给出
`results`: 正则表达式
例子：
```cpp
#include <iostream>
#include <string>
#include <regex>
 
int main() {
    // 简单正则表达式匹配
    std::string fnames[] = {"foo.txt", "bar.txt", "baz.dat", "zoidberg"};
    std::regex txt_regex("[a-z]+\\.txt");
 
    for (const auto &fname : fnames) {
        std::cout << fname << ": " << std::regex_match(fname, txt_regex) << '\n';
    }   
 
    // 提取子匹配
    std::regex base_regex("([a-z]+)\\.txt");
    std::smatch match_results;
 
    for (const auto &fname : fnames) {
        if (std::regex_match(fname, match_results, base_regex)) {
            // 首个 sub_match 是整个字符串；下个
            // sub_match 是首个有括号表达式。
            if (match_results.size() == 2) {
                std::ssub_match base_sub_match = match_results[1];
                std::string base = base_sub_match.str();
                std::cout << fname << " has a base of " << base << '\n';
            }
        }
    }
 
    // 提取几个子匹配
    std::regex pieces_regex("([a-z]+)\\.([a-z]+)");
    std::smatch pieces_match_results;
 
    for (const auto &fname : fnames) {
        if (std::regex_match(fname, pieces_match_results, pieces_regex)) {
            std::cout << fname << '\n';
            for (size_t i = 0; i < pieces_match_results.size(); ++i) {
                std::ssub_match sub_match = pieces_match_results[i];
                std::string piece = sub_match.str();
                std::cout << "    submatch " << i << ": " << piece << '\n';
            }   
        }   
    }   
}
```
结果：
```shell
foo.txt: 1
bar.txt: 1
baz.dat: 0
zoidberg: 0
foo.txt has a base of foo
bar.txt has a base of bar
foo.txt
    submatch 0: foo.txt
    submatch 1: foo
    submatch 2: txt
bar.txt
    submatch 0: bar.txt
    submatch 1: bar
    submatch 2: txt
baz.dat
    submatch 0: baz.dat
    submatch 1: baz
    submatch 2: dat
```
## regex_search
尝试匹配一个正则表达式到字符序列的任何部分
匹配的函数模版有7个，浓缩下来有两个
```cpp
bool regex_search(first, last, [results], regex, [flags]);

bool regex_search(str, [results], regex, [flags]);
```

`first`, `last`: 标识目标字符序列的范围
`str`: 指向空终止字符序列的指针
`regex`: 应当应用到目标字符序列的 std::regex
`results`: 匹配结果
`flags`: 掌管搜索行为的 std::regex_constants::match_flag_type

例子：
```cpp
#include <iostream>
#include <string>
#include <regex>
 
int main() {
    std::string lines[] = {"Roses are #ff0000",
                           "violets are #0000ff",
                           "all of my base are belong to you"};
 
    std::regex color_regex("#([a-f0-9]{2})"
                            "([a-f0-9]{2})"
                            "([a-f0-9]{2})");
 
    // 简单匹配
    for (const auto &line : lines) {
        std::cout << line << ": " << std::boolalpha
                  << std::regex_search(line, color_regex) << '\n';
    }   
    std::cout << '\n';
 
    // 展示每个匹配中有标记子表达式的内容
    std::smatch color_match;
    for (const auto& line : lines) {
        if(std::regex_search(line, color_match, color_regex)) {
            std::cout << "matches for '" << line << "'\n";
            std::cout << "Prefix: '" << color_match.prefix() << "'\n";
            for (size_t i = 0; i < color_match.size(); ++i) 
                std::cout << i << ": " << color_match[i] << '\n';
            std::cout << "Suffix: '" << color_match.suffix() << "\'\n\n";
        }
    }
 
    // 重复搜索（参阅 std::regex_iterator ）
    std::string log(R"(
        Speed:	366
        Mass:	35
        Speed:	378
        Mass:	32
        Speed:	400
	Mass:	30)");
    std::regex r(R"(Speed:\t\d*)");
    std::smatch sm;
    while(regex_search(log, sm, r)) {
        std::cout << sm.str() << '\n';
        log = sm.suffix();
    }
 
    // C 风格字符串演示
    std::cmatch cm;
    if(std::regex_search("this is a test", cm, std::regex("test"))) 
        std::cout << "\nFound " << cm[0] << " at position " << cm.prefix().length();
}
```

结果：
```shell
Roses are #ff0000: true
violets are #0000ff: true
all of my base are belong to you: false
 
matches for 'Roses are #ff0000'
Prefix: 'Roses are '
0: #ff0000
1: ff
2: 00
3: 00
Suffix: ''
 
matches for 'violets are #0000ff'
Prefix: 'violets are '
0: #0000ff
1: 00
2: 00
3: ff
Suffix: ''
 
Speed:	366
Speed:	378
Speed:	400
 
Found test at position 10
```
## regex_replace
以格式化的替换文本来替换正则表达式匹配的出现位置
匹配的函数模版有6个，浓缩下来有两个
```cpp
// 返回输出迭代器 out 在所有插入后的副本
OutputIt regex_replace(out, first, last, basic_regex_re, basic_string_fmt, [flags]); 

// 返回含有输出的字符串 result 
std::basic_string<CharT,STraits,SAlloc>
    regex_replace(basic_string_s, basic_regex_re, basic_string_fmt, [flags]);
```

参数：
`first`, `last`: 以一对迭代器表示的输入字符序列
`basic_string_s`: 以 std::basic_string 或字符数组表示的输入字符序列
`basic_regex_re`: 将与输入序列匹配的 std::basic_regex
`flags`: std::regex_constants::match_flag_type 类型的匹配标志
`basic_string_fmt`: 正则表达式替换格式字符串，准确语法依赖于 flags 的值
`out`: 存储替换结果的输出迭代器

例子：
```cpp
#include <iostream>
#include <iterator>
#include <regex>
#include <string>
 
int main() {
   std::string text = "Quick brown fox";
   std::regex vowel_re("a|e|i|o|u");
 
   // 写结果到输出迭代器
   std::regex_replace(std::ostreambuf_iterator<char>(std::cout),
                      text.begin(), text.end(), vowel_re, "*");
 
   // 构造保有结果的字符串
   std::cout << '\n' << std::regex_replace(text, vowel_re, "[$&]") << '\n';
}
```

结果：
```shell
Q**ck br*wn f*x
Q[u][i]ck br[o]wn f[o]x
```
