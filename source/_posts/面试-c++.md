title: c++
author: ming
date: 2020-08-17 10:56:00
tags:
---
## 0.重要链接https://stibel.icu/

## 1.环境搭建
1. 安装g++
```bash
sudo apt-get install build-essential
```

## 2.helloworld
```c++
// 0. #include 预处理指令
// 1. iostream头文件, 标准c的头文件stdio.h, 从c转化到c++的头文件cmath
// 2. std是命名空间
// 3. cout是对象
// 4. <<是运算符
// 5. "hello!"是运算符的右操作数
// 6. endl也是对象, 与“\n”的差别是，先换行然后flush

#include <iostream>

int main() {
    std::cout << "hello!" << std::endl;
    std::cout << "nani" "yyyy" << std::endl;
    return 0;
}
```

## 3.随机数
```c++
#include <iostream>
#include <cstdlib>
#include <ctime>

int main() {
    srand((unsigned)time(NULL));
    std::cout << rand() << std::endl;
    std::cout << rand() << std::endl;
    return 0;
}
```

## 4.整形
```c++
#include <iostream>
#include <climits>

int main() {
    std::cout << CHAR_BIT << std::endl;
    std::cout << CHAR_MIN << std::endl;
    std::cout << CHAR_MAX << std::endl;
    std::cout << SCHAR_MIN << std::endl;
    std::cout << SCHAR_MAX << std::endl;
    std::cout << UCHAR_MAX << std::endl;

    std::cout << INT_MIN << std::endl;
    std::cout << INT_MAX << std::endl;
    std::cout << UINT_MAX << std::endl;

    std::cout << LONG_MIN << std::endl;
    std::cout << LONG_MAX << std::endl;
    std::cout << ULONG_MAX << std::endl;

    std::cout << LLONG_MIN << std::endl;
    std::cout << LLONG_MAX << std::endl;
    std::cout << ULLONG_MAX << std::endl;
    return 0;
}
```

## 5.初始化器{}
```c++
#include <iostream>

int main() {
    int val = {3};
    int x{5};
    std::cout << val << std::endl;
    std::cout << x << std::endl;
    return 0;
}
```

## 6.进制 
```c++
#include <iostream>

int main() {
    int val1 = 10;
    int val2 = 0x10;
    int val3 = 010;
    std::cout << val1 << std::endl;
    std::cout << std::hex; // 切换到16进制显示
    std::cout << val2 << std::endl;
    std::cout << std::oct; // 切换到8进制显示
    std::cout << val3 << std::endl;
    return 0;
}
```

## 7. 字符
```c++
#include <iostream>

int main() {
    char ch;
    std::cout << "please input a char:";
    std::cin >> ch;
    std::cout.put(ch);
    std::cout.put('!');
    std::cout.put('\');
    std::cout << std::endl;
    return 0;
}
```

## 8. 宽字符
```c++
#include <iostream>

int main() {
    wchar_t ch;
    std::wcin >> ch;
    std::wcout << "what=" << ch << std::endl;
    return 0;
}
```

## 9. bool
```c++
#include <iostream>

int main() {
    bool flag = -12;
    if (flag) {
        std::cout << "flag is false" << std::endl;
    } else {
        std::cout << "flag is true" << std::endl;
    }
    std::cout << flag << std::endl;
    std::cout << true << std::endl;
    return 0;
}
```

## 10. const
```c++
#include <iostream>

int main() {
    const int NANI = 100;
    std::cout << NANI << std::endl;
    //NANI = 3;
    //std::cout << NANI << std::endl;
    return 0;
}
```

## 11. 浮点数
```c++
#include <iostream>

int main() {
    float val = 10.0 / 3.0;
    double val1 = 10.0 / 3.0;
    const float sub = 1.0e6;
    std::cout.setf(std::ios_base::fixed, std::ios_base::floatfield);
    std::cout << val << std::endl;
    std::cout << val1 << std::endl;
    std::cout << val * sub << std::endl;
    std::cout << 10 * val * sub << std::endl;
    std::cout << val1 * sub << std::endl;
    std::cout << 10 * val1 * sub << std::endl;
    return 0;
}
```

## 12. 类型转化
```c++
#include <iostream>

int main() {
    int a = 1.3f + 1.8f;
    std::cout << a << std::endl;

    int b = int(1.3f) + int(1.8f);
    std::cout << b << std::endl;

    int c = static_cast<int> (2.3);
    std::cout << c << std::endl;
    return 0;
}
```

## 13. 自动类型 auto
```c++
#include <iostream>

int main() {
    auto a = 1.2;
    std::cout << a << std::endl;
    auto b = 1.3f;
    std::cout << b << std::endl;
    auto c = 0;
    std::cout << c << std::endl << sizeof(c) << std::endl;
    return 0;
}
```

## 14. 数组 
```c++
#include <iostream>

int main() {
    int arr[5];
    int len = sizeof(arr) / sizeof(int);
    for (int i=0; i<len; i++) {
        std::cout << arr[i] << std::endl;
    }
    std::cout << "------" << std::endl;
    std::cout << arr << std::endl;
    std::cout << sizeof(arr[1]) << std::endl;
    std::cout << sizeof(arr) << std::endl;
    return 0;
}
```

## 15. 字符串
```c++
// cin 输入的字符串会自动在末尾追加'\0'
#include <iostream>
#include <cstring>

void print_arr(char arr[]) {
    std::cout << "-----" << std::endl;
    for (int i=0; i<5; i++) {
        std::cout << int(arr[i]) << std::endl;
    }
    std::cout << "-----" << std::endl;
}

int main() {
    char arr[5] = {0, 0, 0, 0, 5};
    print_arr(arr);
    std::cin >> arr;
    print_arr(arr);
    std::cout << strlen(arr) << std::endl;
    std::cout << arr << std::endl;
    return 0;
}
```

```c++
// getline 获取指定长度的字符串
#include <iostream>

int main() {
    const int MAX_LEN = 5;
    char name[MAX_LEN];
    std::cin.getline(name, MAX_LEN);
    std::cout << name << std::endl;
    return 0;
}
```

```c++
#include <iostream>

int main() {
    const int MAX_LEN = 5;

    char name1[MAX_LEN];
    std::cin.get(name1, MAX_LEN);
    std::cout << name1 << std::endl;

    // 用于读取'\n', getline会默认读取'\n'
    std::cin.get();

    char name2[MAX_LEN];
    std::cin.get(name2, MAX_LEN);
    std::cout << name2 << std::endl;
    return 0;
}
```

## 16. string类
```c++
// 字符串拼接，c和c++两种方式
#include <iostream>
#include <string>
#include <cstring>

int main() {
    std::string s1;
    std::cin >> s1;
    std::string s2;
    std::cin >> s2;
    std::cout << "-->" + s1 + s2 << std::endl;

    char s3[20];
    strcpy(s3, "-->");
    std::cin >> (s3 + 3);
    char s4[10];
    std::cin >> s4;
    char* res = strcat(s3, s4);
    std::cout << res << std::endl;
    return 0;
}
```

```c++
#include <iostream>
#include <string>
#include <cstring>

int main() {
    std::string s1;
    std::cin >> s1;
    std::cout << "-->" << s1.size() << std::endl;

    char arr[10];
    std::cin >> arr;
    std::cout << "-->" << strlen(arr) << std::endl;
    return 0;
}
```

## 17. 字符串字面值
```c++
#include <iostream>
#include <string>
#include <cstring>

int main() {
    wchar_t arr0[] = L"1234";
    char16_t arr1[] = u"1234";
    char32_t arr2[] = U"1234";
    std::cout << sizeof(wchar_t) << std::endl;
    std::cout << arr0 << "-->" << sizeof(arr0) << std::endl;
    std::cout << arr1 << "-->" << sizeof(arr1) << std::endl;
    std::cout << arr2 << "-->" << sizeof(arr2) << std::endl;

    std::string s = "严";
    std::cout << s << "-->" << s.size() << std::endl;
    return 0;
}
```

```c++
// +*( 会被当作分隔符
#include <iostream>
#include <string>
#include <cstring>

int main() {
    std::string s = R"+*(ls\ll"hkks"("0\)0")+*";
    std::cout << s << std::endl;
    return 0;
}
```

## 18.结构
```c++
#include <iostream>
#include <string>
#include <cstring>

struct Dog {
    std::string name;
    bool sex;
    int age;
};

// 占用的bit数
struct Cat {
    bool sex : 1;
    int age : 32;
};

int main() {
    Dog dog1 = { "gouzi", 1, 3 };
    std::cout << dog1.name << std::endl;
    dog1.name = "nani";
    std::cout << dog1.name << std::endl;

    struct Dog dog2 = { "what", 0, 3 };
    std::cout << dog2.name << std::endl;

    Dog dog3 = dog2;
    std::cout << dog3.name << std::endl;

    Dog dogs[3] = { dog1, dog2, dog3 };
    for (int i=0; i<3; i++) {
        std::cout << &(dogs[i].name) << "-->" <<  dogs[i].name << std::endl;
    }

    std::cout << sizeof(Cat) << "    " << sizeof(Dog) << std::endl;
    return 0;
}
}
```

## 19.共用体
```c++
#include <iostream>
#include <string>

struct Dog {
    std::string name;
    bool sex;
    int age;
    // 匿名共用体
    union {
        int ival;
        long lval;
    } id;
};

int main() {
    Dog dog = {
        "what",
        1,
        12,
        4L
    };
    std::cout << dog.id.lval << std::endl;
    std::cout << dog.id.ival << std::endl;
    return 0;
}
```

## 20.枚举
```c++
#include <iostream>

enum Color {
    red,
    green,
    blue
};

int main() {
    Color color = green;
    std::cout << color << std::endl;

    Color color1 = Color(2);
    std::cout << color1 << std::endl;
    return 0;
}
```

## 21.指针
```c++
#include <iostream>
#include <string>

int main() {
    int* p = new int;
    int* pArr = new int[10] {
        5, 6, 2, 4, 6
    };
    int arr[10] = {4, 3, 2, 1};

    pArr++;
    std::cout << pArr[2] << std::endl;
    pArr--;

    std::cout << arr[1] << std::endl;

    delete[] pArr;
    delete p;
    return 0;
}
```

```c++
// 指向结构体的指针
#include <iostream>
#include <string>

struct Dog {
    std::string name;
    int sex;
};

int main() {
    Dog* pDog = new Dog;
    pDog->name = "gouzi";
    pDog->sex = 1;
    std::cout << pDog->name << std::endl;
    std::cout << sizeof(*pDog) << std::endl;
    delete pDog;
    return 0;
}
```

```c++
// 复合类型指针
#include <iostream>
#include <string>

struct Dog {
    std::string name;
    int sex;
};

int main() {
    const Dog** arr = new const Dog*[5];
    Dog** parr = (Dog**) arr;
    for (int i=0; i<5; i++) {
        Dog* p = new Dog;
        p->name = "dog-" + std::to_string(i);
        *parr = p;
        parr++;
    }

    std::cout << arr[1]->name << std::endl;

    for (int i=0; i<5; i++) {
        delete arr[i];
    }
    delete[] arr;

    return 0;
}
```

## 22. vector & array
```c++
#include <iostream>
#include <string>
#include <vector>
#include <array>

struct Dog {
    std::string name;
    int sex;
};

int main() {
    std::vector<int> list;
    int x = 0;
    while (x >= 0) {
        std::cin >> x;
        list.push_back(x);
        std::cout << x << "-->"  << list.size() << std::endl;
    }

    // array分配在栈上
    std::array<int, 5> arr = { 3, 4, 5 };
    arr[4] = 9;
    for (int i=0; i<5; i++) {
        std::cout << arr[i] << std::endl;
    }
    return 0;
}
```

## 23. cin对象
```c++
#include <iostream>
#include <string>

struct Dog {
    std::string name;
    int sex;
};

int main() {
    std::string str;
    char ch;
    // 这几种判断方式都可以, 判断cin对象，还可以检查磁盘错误
    //while (!std::cin.eof()) {
    //while (std::cin) {
    while (std::cin.get(ch)) {
        //std::cin.get();
        str.append(1, ch);
    }
    std::cout << "-->" << str << std::endl;
    return 0;
}
```

## 24. cctype
```c++
// 统计总字符数和空格数
#include <iostream>
#include <string>
#include <cctype>

int main() {
    int other = 0;
    int space_count = 0;
    int punct_count = 0;
    int alpha_count = 0;
    int enter_count = 0;
    int num_count = 0;

    char ch;
    while (std::cin.get(ch)) {
        if (isspace(ch)) space_count++;
        if (ispunct(ch)) punct_count++;
        if (isdigit(ch)) num_count++;
        if (isalpha(ch)) alpha_count++;
        if (ch == '\n') enter_count++;
        other++;
    }
    std::cout << "space count:" << space_count << std::endl;
    std::cout << "punct count:" << punct_count << std::endl;
    std::cout << "num count:" << num_count << std::endl;
    std::cout << "alpha count:" << alpha_count << std::endl;
    std::cout << "enter count:" << enter_count << std::endl;
    std::cout << "other count:" << other << std::endl;
    return 0;
}
```

## 25. fstream
```c++
#include <iostream>
#include <string>
#include <fstream>

int main() {
    std::ofstream out_file;
    out_file.open("aa");

    const int MAX_LINE_SIZE = 100;
    char* line = new char[MAX_LINE_SIZE];
    while (std::cin) {
        std::cin.getline(line, MAX_LINE_SIZE);
        out_file << line << std::endl;
    }
    out_file.close();
    return 0;
}
```

```c++
#include <iostream>
#include <string>
#include <fstream>

int main() {
    std::ifstream in_file;
    in_file.open("bb");
    if (!in_file.is_open()) {
        std::cout << "Error: open bb error" << std::endl;
    }
    // seek到倒数第0个字符
    in_file.seekg(0, std::ios_base::end);
    int length = in_file.tellg();
    std::cout << length << std::endl;
    in_file.seekg(0, std::ios_base::beg);

    std::ofstream out_file;
    out_file.open("aa");
    if (!out_file.is_open()) {
        std::cout << "Error: open aa error" << std::endl;
    }

    int count = 0;
    const int MAX_LINE_SIZE = 10;
    char* buffer = new char[MAX_LINE_SIZE];
    while (in_file.good() && out_file.good()) {
        int size = length - count;
        size = size > MAX_LINE_SIZE ? MAX_LINE_SIZE : size;
        in_file.read(buffer, size);
        out_file.write(buffer, size);
        count += size;
        std::cout << "write size=" << size << std::endl;
        if (count >= length) break;
    }

    delete []buffer;

    in_file.close();
    out_file.close();


    return 0;
}
```

## 26. 数组作为函数参数
```c++
#include <iostream>
#include <string>
#include <fstream>

void modify(int arr[], int len);
void print_arr(int arr[], int len);

int main() {
    const int len = 10;
    // 如果被当作指针，则sizeof返回8
    //int* arr = new int[len];
    //std::cout << sizeof(arr) << std::endl;

    // 如果被当作数组，则sizeof返回40
    int arr[len] = {};
    std::cout << sizeof(arr) << std::endl;

    print_arr(arr, len);
    modify(arr, len);
    print_arr(arr, len);
    return 0;
}

void modify(int arr[], int len) {
    for (int i=0; i<len; i++) {
        arr[i] += 1;
    }
}

void print_arr(int arr[], int len) {
    // 编译错误，数组作为函数参数，会被当作指针
    //std::cout << sizeof(arr) << std::endl;
    for (int i=0; i<len; i++) {
        std::cout << arr[i] << ", ";
    }
    std::cout << std::endl;
}
```

## 27. const指针
```c++
// int* const src = new int[len];
// const int* src = new int[len];

#include <iostream>
#include <string>
#include <fstream>

void modify(int arr[], int len);
void print_arr(const int* arr, int len);

int main() {
    const int len = 10;
    // src的值不能改变
    int* const src = new int[len];
    // arr[i]的值不能改变 
    const int* arr = src;

    print_arr(arr, len);
    modify(src, len);
    print_arr(arr, len);
    return 0;
}

void modify(int arr[], int len) {
    for (int i=0; i<len; i++) {
        arr[i] += 1;
    }
}

void print_arr(const int* arr, int len) {
    //std::cout << sizeof(arr) << std::endl;
    for (int i=0; i<len; i++) {
        std::cout << arr[i] << ", ";
    }
    std::cout << std::endl;
}
```

## 28. 传指针 & 传引用
```c++
#include <iostream>
#include <string>
#include <fstream>

struct Dog {
    const char* name;
    int age;
};

void print_dog(Dog* dog);
void print_dog(Dog& dog);

int main() {
    Dog dog = {"gouzi", 10};
    print_dog(&dog);
    print_dog(dog);
    return 0;
}

void print_dog(Dog* dog) {
    std::cout << &(dog->name) << "-->" << dog->name << std::endl;
    std::cout << &(dog->age) << "-->" << dog->age << std::endl;
}

void print_dog(Dog& dog) {
    std::cout << &(dog.name) << "-->" << dog.name << std::endl;
    std::cout << &(dog.age) << "-->" << dog.age << std::endl;
}
```

## 29. 函数指针
```c++
#include <iostream>
#include <string>
#include <fstream>

struct Dog {
    const char* name;
    int age;
};

void print_dog(Dog* dog) {
    std::cout << &(dog->name) << "-->" << dog->name << std::endl;
    std::cout << &(dog->age) << "-->" << dog->age << std::endl;
}

void print_dog(Dog&, void (*f)(Dog*));

int main() {
    Dog dog = {"gouzi", 10};
    print_dog(dog, print_dog);
    return 0;
}

void print_dog(Dog& dog, void (*p_dog)(Dog*)) {
    p_dog(&dog);
}
```

```c++
// 使用typedef简化声明和定义的写法
#include <iostream>
#include <string>
#include <fstream>

struct Dog {
    const char* name;
    int age;
};

void print_dog(Dog* dog) {
    std::cout << &(dog->name) << "-->" << dog->name << std::endl;
    std::cout << &(dog->age) << "-->" << dog->age << std::endl;
}

// f是类型名
typedef void (*f)(Dog*);
typedef Dog& RDog;
void print_dog(RDog, f);


int main() {
    Dog dog = {"gouzi", 10};
    print_dog(dog, print_dog);
    return 0;
}


void print_dog(RDog dog, f p_dog) {
    p_dog(&dog);
}
```

## 30. 内联函数inline
```c++
// 当函数调用的开销，占函数执行的开销比重比较大的时候，可以使用内联，否则意义不大
inline void print_dog(Dog* dog) {
    std::cout << &(dog->name) << "-->" << dog->name << std::endl;
    std::cout << &(dog->age) << "-->" << dog->age << std::endl;
}
```

## 31. 变量引用 
```c++
#include <iostream>
#include <string>
#include <fstream>

struct Dog {
    const char* name;
    int age;
};

int main() {
    Dog dog = {"gouzi", 10};
    Dog& dog1 = dog;
    Dog* dog2 = &dog;
    std::cout << &dog << std::endl;
    std::cout << &dog1 << std::endl;
    std::cout << dog2 << std::endl;
    return 0;
}
```

```c++
#include <iostream>
#include <string>

void swap(int& a, int& b);
void swap_ptr(int* a, int* b);

int main() {
    int a = 3;
    int b = 5;
    std::cout << a << ", " << b << std::endl;

    swap(a, b);
    //swap_ptr(&a, &b);
    std::cout << "-->" << a << ", " << b << std::endl;

    return 0;
}

void swap_ptr(int* a, int* b) {
    int tmp = *a;
    *a = *b;
    *b = tmp;
}

void swap(int& a, int& b) {
    int tmp = a;
    a = b;
    b = tmp;
}
```

```c++
// Dog和Dog&是两个类型
#include <iostream>
#include <string>
#include <cstring>

struct Dog {
    char* name;
    int age;
};

int main() {
    Dog dog = {"gouzi", 20};
    std::cout << &dog << std::endl;

    // 这个等号表示拷贝
    Dog dog1 = dog;
    std::cout << &dog1 << std::endl;

    // 这个是起别名
    Dog& dog2 = dog;
    std::cout << &dog2 << std::endl;

    return 0;
}
```

## 32. 函数返回引用类型
```c++
#include <iostream>
#include <string>
#include <cstring>

struct Dog {
    char* name;
    int age;
};

void print_dog(Dog& dog);
Dog& clone(Dog& dog);
void destory_dog(Dog* dog);

int main() {
    Dog dog = {"hello", 20};
    print_dog(dog);

    Dog& dog1 = clone(dog);
    std::cout << "clone1:" << &dog1 << "---" << dog1.name << std::endl;
    print_dog(dog1);

    destory_dog(&dog1);
    return 0;
}

void print_dog(Dog& dog) {
    std::cout << &(dog.name) << "-->" << dog.name << std::endl;
}

Dog& clone(Dog& dog) {
    Dog* dog1 = new Dog;
    int len = std::strlen(dog.name);
    dog1->name = new char[len + 1];
    std::strncpy(dog1->name, dog.name, len);
    dog1->name[0] = 'M';
    dog1->age = dog.age;
    std::cout << "clone:" << dog1 << "---" << dog1->name << std::endl;
    return *dog1;
}

void destory_dog(Dog* dog) {
    delete [](dog->name);
    delete dog;
}
```

```c++
#include <iostream>
#include <string>
#include <cstring>

std::string& func1(const std::string&, const std::string&);

int main() {
    std::string input;
    getline(std::cin, input);
    std::string& res = func1(input, "nani");
    std::cout << res << std::endl;

    delete &res;
    return 0;
}

std::string& func1(const std::string& s1, const std::string& s2) {
    std::string* s = new std::string(s1);
    *s += s2;
    return *s;
}
```

## 33. 函数模版 
```c++
#include <iostream>
#include <string>
#include <cstring>

template <typename T>
void swap(T&, T&);

int main() {
    std::string a = "aaa";
    std::string b = "bbb";
    std::cout << a << ", " << b <<std::endl;
    swap(a, b);
    std::cout << a << ", " << b <<std::endl;
    return 0;
}

template <typename T>
void swap(T& a, T& b) {
    T tmp = a;
    a = b;
    b = tmp;
}
```

```c++
// 具体化模版
#include <iostream>
#include <string>
#include <cstring>

struct Dog {
    std::string name;
    int age;
};

template <typename T>
void swap(T&, T&);

void swap(Dog&, Dog&);

int main() {
    std::string a = "aaa";
    std::string b = "bbb";
    std::cout << a << ", " << b <<std::endl;
    swap(a, b);
    std::cout << a << ", " << b <<std::endl;

    std::cout << "----------" << std::endl;

    Dog dog1 = {"dog aa", 5};
    Dog dog2 = {"dog bb", 7};
    std::cout << dog1.name << ", " << dog1.age <<std::endl;
    std::cout << dog2.name << ", " << dog2.age <<std::endl;
    swap(dog1, dog2);
    std::cout << dog1.name << ", " << dog1.age <<std::endl;
    std::cout << dog2.name << ", " << dog2.age <<std::endl;
    return 0;
}

void swap(Dog& a, Dog& b) {
    swap(a.name, b.name);
}

template <typename T>
void swap(T& a, T& b) {
    T tmp = a;
    a = b;
    b = tmp;
}
```

## 34. 命名空间 
```c++
#include <iostream>
#include <string>
#include <cstring>

namespace ming {
    void hello() {
        std::cout << "ming hello" << std::endl;
    }

    namespace mad {
        void hello() {
            std::cout << "ming mad hello" << std::endl;
        }
    }
}

void hello();

int main() {
    ming::mad::hello();
    return 0;
}


void hello() {
    std::cout << "global hello" << std::endl;
}
```

## 35. 类
```c++
#ifndef DOG_H_
#define DOG_H_

#include <string>

class Dog {
    public:
        Dog(std::string name, int age);
        ~Dog();
        std::string say();
        std::string getName();
        int getAge();

    private:
        std::string m_name;
        int m_age;
};

#endif
```

```c++
#include <iostream>
#include <string>
#include "Dog.h"

Dog::Dog(std::string name, int age) {
    this->m_name = name;
    this->m_age = age;
    std::cout << "Dog() called..." << std::endl;
}

Dog::~Dog() {
    std::cout << "~Dog() called..." << std::endl;
}

std::string Dog::say()  {
    return "My name is " + this->m_name;
}

std::string Dog::getName() {
    return this->m_name;
}

int Dog::getAge() {
    return this->m_age;
}
```

```c++
#include <iostream>
#include "Dog.h"

int main() {
    Dog* dog = new Dog("gouzi", 10);
    std::cout << dog->say()<< std::endl;
    delete dog;
    return 0;
}
```

## 36. 对象数组
```c++
#include <iostream>
#include <string>
#include "Dog.h"

int main() {
    Dog** arr = new Dog*[5];
    for (int i=0; i<5; i++) {
        arr[i] = new Dog("dog_"+ std::to_string(i), i);
    }
    std::cout << arr[0]->say()<< std::endl;
    for (int i=0; i<5; i++) {
        delete *(arr+i);
    }
    return 0;
}
```

## 37. 声明对象常量和类常量
```c++
#ifndef DOG_H_
#define DOG_H_

#include <string>

class Dog {
    public:
        static const int NANI = 99;
        Dog(const std::string& name, int age);
        ~Dog();
        std::string say();
        std::string getName();
        int getAge();

    private:
        const int SIZE = 100;
        std::string m_name;
        int m_age;
};

#endif
```

## 38. 运算符重载 & 友元函数 & 重载<<
```c++
#ifndef TIME_H_
#define TIME_H_

#include <ostream>

class Time {
    public:
        Time();
        Time(int hour, int minute);
        ~Time();
        void show() const;
        Time to_time(const int minutes) const;
        int to_minute(const Time& t) const;
        Time operator+(const Time& t) const;
        Time operator-(const Time& t) const;
        // 添加末尾的const，表示此函数不会修改当前对象成员变量
        Time operator*(const double val) const;
        // 友元函数不是成员函数，所以不能加末尾的const
        friend Time operator*(const double val, const Time& t);
        friend std::ostream& operator<<(std::ostream& os, const Time& t);

    private:
        int m_hour;
        int m_minute;
};

#endif  // TIME_H_
```

```c++
#include <iostream>
#include "Time.h"

Time::Time() {
    this->m_hour = 0;
    this->m_minute = 0;
}

Time::Time(int hour, int minute) {
    this->m_hour = hour;
    this->m_minute = minute;
}

Time::~Time() {

}

void Time::show() const {
    std::cout << this->m_hour << ":" << this->m_minute << std::endl;
}

Time Time::to_time(const int minutes) const {
    Time tmp(minutes / 60, minutes % 60);
    return tmp;
}

int Time::to_minute(const Time& t) const {
    return t.m_hour * 60 + t.m_minute;
}

Time Time::operator+(const Time& t) const {
    Time tmp;
    tmp.m_minute = this->m_minute + t.m_minute;
    tmp.m_hour = this->m_hour + t.m_hour;
    if (tmp.m_minute >= 60) {
        tmp.m_hour += 1;
        tmp.m_minute %= 60;
    }
    return tmp;
}

Time Time::operator-(const Time& t) const {
    return to_time(to_minute(*this) - to_minute(t));
}

Time Time::operator*(const double val) const {
    Time tmp;
    tmp.m_hour = this->m_hour * val;
    tmp.m_minute = this->m_minute * val;
    if (tmp.m_minute >= 60) {
        tmp.m_hour += tmp.m_minute / 60;
        tmp.m_minute %= 60;
    }
    return tmp;
}

// 友元函数的定义不用加friend
Time operator*(const double val, const Time& t) {
    return t * val; // 又调用成员函数
}

// 友元
std::ostream& operator<<(std::ostream& os, const Time& t) {
    os << t.m_hour << ":" << t.m_minute;
    return os;
}
```

```c++
#include <iostream>
#include "Time.h"

int main() {
    Time t1(2, 30);
    Time t2(1, 20);

    (t1 + t2).show();
    (t1 - t2).show();
    t1.operator-(t2).show();

    (t1 * 2).show();
    (2 * t1).show(); // 调用友元函数

    std::cout << t2 << std::endl;

    return 0;
}
```

## 39. 类型转化(对象-->int || int --> 对象)
```c++
#ifndef DOG_H_
#define DOG_H_

#include <ostream>

class Dog {
    public:
        Dog(int age);
        ~Dog();
        // Dog转化成int
        // explicit表示强制显示类型转化
        explicit operator int() const;
        friend std::ostream& operator<<(std::ostream& os, Dog& dog);
    private:
        int m_age;
};
#endif //DOG_H_
```

```c++
#include "Dog.h"

Dog::Dog(int age) {
    this->m_age = age;
}

Dog::~Dog() {

}

std::ostream& operator<<(std::ostream& os, Dog& dog)  {
    return os << dog.m_age;
}

Dog::operator int() const {
    return this->m_age;
}
```

```c++
#include <iostream>
#include "Dog.h"

int main() {
    Dog dog = 100;
    std::cout << dog << std::endl;

    dog = (Dog) 101;
    std::cout << dog << std::endl;

    dog = Dog(102);
    std::cout << dog << std::endl;

    int age = dog;
    std::cout << age << std::endl;

    int age1 = (int) dog;
    std::cout << age1 << std::endl;

    int age2 = int(dog);
    std::cout << age2 << std::endl;

    return 0;
}
```

## 40. 拷贝构造 & =运算符重载
```c++
#ifndef MYSTRING_H_
#define MYSTRING_H_

#include <ostream>
#include <istream>

class Mystring {
    public:
        Mystring();
        Mystring(const Mystring& s);
        Mystring(const char* s);
        ~Mystring();

        void create(const char* s);
        int getLen() const;

        Mystring& operator=(const Mystring& s);
        Mystring& operator=(const char* s);
        char& operator[](const int i);
        const char& operator[](const int i) const;
        friend std::ostream& operator<<(std::ostream& os, const Mystring& s);
        friend std::istream& operator>>(std::istream& is, const Mystring& s);
        friend int operator>(const Mystring& s1, const Mystring& s2);
        friend int operator<(const Mystring& s1, const Mystring& s2);
        friend int operator==(const Mystring& s1, const Mystring& s2);

        static int howMany();


    private:
        char* m_str;
        int m_len;
        static int s_count;
};

#endif //MYSTRING_H_
```

```c++
#include "Mystring.h"
#include <cstring>
#include <iostream>

int Mystring::s_count = 0;

Mystring::Mystring() {
    create("c++");
    s_count++;
    std::cout << "Mystring() c++" << std::endl;
}

void Mystring::create(const char* s) {
    m_len = strlen(s) + 1;
    m_str = new char[m_len];
    strncpy(m_str, s, m_len);
}

Mystring::Mystring(const Mystring& s) {
    create(s.m_str);
    s_count++;
    std::cout << "copy Mystring()" << s.m_str << std::endl;
}

Mystring::Mystring(const char* s) {
    create(s);
    s_count++;
    std::cout << "Mystring()" << s << std::endl;
}

Mystring::~Mystring() {
    std::cout << "~Mystring()" << m_str << std::endl;
    delete []m_str;
    m_str = NULL;
    m_len = 0;
    s_count--;
}

Mystring& Mystring::operator=(const Mystring& s) {
    std::cout << "operator=(const Mystring&)" << std::endl;
    if (this == &s) return *this;
    delete this->m_str;
    create(s.m_str);
    std::cout << "operator=" << m_str << std::endl;
    return *this;
}

Mystring& Mystring::operator=(const char* s) {
    std::cout << "operator=(const char*)" << std::endl;
    create(s);
    return *this;
}

int Mystring::getLen() const {
    return this->m_len;
}

std::ostream& operator<<(std::ostream& os, const Mystring& s) {
    return os << s.m_str;
}

std::istream& operator>>(std::istream& is, const Mystring& s) {
    return is >> s;
}

int operator>(const Mystring& s1, const Mystring& s2) {
   return std::strcmp(s1.m_str, s2.m_str) > 0;
}

int operator<(const Mystring& s1, const Mystring& s2) {
   return std::strcmp(s1.m_str, s2.m_str) < 0;
}

int operator==(const Mystring& s1, const Mystring& s2) {
   return strcmp(s1.m_str, s2.m_str) == 0;
}

char& Mystring::operator[](const int i) {
    std::cout << R"(char& Mystring::operator[](const int i))";
    return m_str[i];
}

const char& Mystring::operator[](const int i) const {
    std::cout << R"(const char& Mystring::operator[](const int i) const)" << std::endl;
    return m_str[i];
}

int Mystring::howMany() {
    return s_count;
}
```

```c++
#include <iostream>
#include "Mystring.h"

int main() {
    Mystring s1 = "12345";
    std::cout << s1 << std::endl;
    std::cout << "--------" << std::endl;

    Mystring t2 = "aaa000";
    Mystring* s2 = new Mystring(t2);
    std::cout << *s2 << std::endl;
    delete s2;
    std::cout << "--------" << std::endl;

    Mystring t3 = "bbb000";
    Mystring s3(t3);
    std::cout << "--------" << std::endl;

    Mystring t4 = "ccc000";
    Mystring s4;
    s4 = t4;
    std::cout << "--------" << std::endl;

    Mystring t5 = "abcd";
    std::cout << t5[2] << std::endl;
    std::cout << "--------" << std::endl;

    Mystring t6 = "abc";
    Mystring t7 = "abc";
    std::cout << (t6 > t7) << std::endl;
    std::cout << (t6 < t7) << std::endl;
    std::cout << (t6 == t7) << std::endl;
    std::cout << "--------" << std::endl;

    Mystring t8 = "xyz";
    t8[1] = 'z';
    std::cout << t8 << std::endl;
    std::cout << "--------" << std::endl;

    const Mystring t9 = "opq";
    std::cout << t9[2] << std::endl;
    std::cout << "--------" << std::endl;

    Mystring t10;
    t10 = "www";
    std::cout << "--------" << std::endl;

    std::cout << Mystring::howMany() << std::endl;
    std::cout << "--------" << std::endl;

    std::cout << "main finish" << std::endl;
    return 0;
}
```

## 41. 定位new运算符
```c++
#include <iostream>
#include "mystring/Mystring.h"

const int BUF_SIZE = 512;
int main() {
    char* buffer = new char[BUF_SIZE];

    Mystring* s1 = new (buffer) Mystring("abc");
    std::cout << *s1 << "-->" << s1 << std::endl;
    //delete s1;

    Mystring* s2 = new Mystring("bcd");
    std::cout << *s2 << "-->" << s2 << std::endl;
    delete s2;

    delete []buffer;
    return 0;
}
```

## 42. 类继承 & 多态 & 虚析构
```c++
#ifndef ANIMAL_H_
#define ANIMAL_H_

#include <string>

class Animal {
    public:
        Animal();
        Animal(const std::string& name, int age);
        // 这个要写成虚析构, 当释放父类指针的时候，才会去调用子类的析构
        virtual ~Animal();
        // 多态的标准动作, 虚函数
        virtual void eat(const std::string& something);
        std::string& whoami();

    private:
        std::string m_name;
        int m_age;
        std::string m_info;
};

class Dog : public Animal {
    public:
        enum Color {
            BLACK,
            WHITE,
            OTHER
        };
        Dog();
        Dog(const std::string& name, int age, Color color=BLACK);
        Dog(Animal& animal, Color color=BLACK);
        ~Dog();
        virtual void eat(const std::string& something);
        Color getColor();
    private:
        Color m_color;
};

#endif //ANIMAL_H_
```

```c++
#include <iostream>
#include "Animal.h"

Animal::Animal() {
    std::cout << "Animal()" << std::endl;
}

Animal::Animal(const std::string& name, int age) {
    std::cout << "Animal(name, age)" << std::endl;
    m_name = name;
    m_age = age;
}

Animal::~Animal() {
    std::cout << "~Animal()" << std::endl;
}

void Animal::eat(const std::string& something) {
    std::cout << "animal eat " << something << std::endl;
}

std::string& Animal::whoami() {
    m_info = m_name + "," + std::to_string(m_age);
    return m_info;
}
```

```c++
#include <iostream>
#include "Animal.h"

Dog::Dog() : Animal() {
    std::cout << "Dog()" << std::endl;
}

Dog::Dog(const std::string& name, int age, Color color) : Animal(name, age), m_color(color) {
    std::cout << "Dog(name, age, color)" << std::endl;
}

Dog::Dog(Animal& animal, Color color) : Animal(animal), m_color(color){
    std::cout << "Dog(animal, color)" << std::endl;
}

Dog::~Dog() {
    std::cout << "~Dog()" << std::endl;
}

Dog::Color Dog::getColor() {
    return m_color;
}

void Dog::eat(const std::string& something) {
    std::cout << "Dog eat " << something << std::endl;
}
```

```c++
#include <iostream>
#include "Animal.h"

int main() {
    Animal* animal = new Animal("nani", 10);
    std::cout << animal->whoami() << std::endl;
    animal->eat("meat..");
    delete animal;

    std::cout << "--------" << std::endl;

    Dog* dog = new Dog("what", 6);
    std::cout << dog->whoami() << std::endl;
    dog->eat("meat..");
    delete dog;
    std::cout << "--------" << std::endl;

    Animal* p = new Dog("hello", 3);
    std::cout << p->whoami() << std::endl;
    delete p;

    return 0;
}
```

## 43. 多态的实现 
```c++
// 编译的时候，为每个类添加了一个成员，就是虚表指针。
// 创建对象的时候，为每个对象创建了虚函数表，保存了指向函数的地址，当调用函数的时候，就会使用真实对象的虚标指针去查表，然后调用
```

## 44. 纯虚函数
```c++
// 父类
virtual void fuck(Animal& animal) = 0;
// 子类
void fuck(Animal& dog);
```

## 45. 对象做成员 & valarray模版
```c++
#ifndef STUDENT_H_
#define STUDENT_H_

#include <string>
#include <valarray>
#include <ostream>
#include <istream>

class Student {
    private:
        std::string m_name;
        std::valarray<int> m_scores;

    public:
        Student(std::string name, std::valarray<int> scores);
        ~Student();
        int getAgv() const;
        const std::string& getName() const;
        int& operator[](int i);
        int operator[](int i) const;
        friend std::ostream& operator<<(std::ostream& os, Student& stu);
        friend std::istream& operator>>(std::istream& is, Student& stu);
};

#endif //STUDENT_H_
```

```c++
#include "Student.h"
Student::Student(std::string name, std::valarray<int> scores): m_name(name), m_scores(scores) {

}

Student::~Student() {

}

int Student::getAgv() const {
    return m_scores.sum() / m_scores.size();
}

const std::string& Student::getName() const {
    return m_name;
}

int& Student::operator[](int i) {
    return m_scores[i];
}

int Student::operator[](int i) const {
    return m_scores[i];
}

std::ostream& operator<<(std::ostream& os, Student& stu) {
    return os << stu.m_name;
}

std::istream& operator>>(std::istream& is, Student& stu) {
    return is >> stu.m_name;
}
```

```c++
#include <iostream>
#include "Student.h"

int main() {
    Student s1("ming", {10, 20, 30, 40});
    std::cout << s1 << "-->" << s1.getAgv() << std::endl;
    return 0;
}
```
## 46. 私有继承 & 保护继承 & 公有继承
```c++
// 子类对外的接口，是基类和子类的叠加，选择较小的范围
// 对于私有继承，如果想访问基类的public成员，则可以使用using 把方法声明成public的
```

## 47. 类模版
```c++
#ifndef STACK_H_
#define STACK_H_

template <class T>
class Stack {

    public:
        Stack();
        bool isEmpty() const;
        bool isFull() const;
        bool push(T&);
        bool pop(T&);

    private:
        static const int MAX_SIZE = 10;
        T m_arr[MAX_SIZE];
        int m_top;
};

#endif //STACK_H_
```

```c++
#include "Stack.h"

template<class T>
Stack<T>::Stack() {
    m_top = 0;
}

template<class T>
bool Stack<T>::isEmpty() const {
    return m_top <= 0;
}

template<class T>
bool Stack<T>::isFull() const {
    return m_top >= Stack::MAX_SIZE;
}

template<class T>
bool Stack<T>::push(T& item) {
    if (isFull()) return false;
    m_arr[m_top++] = item;
    return true;
}

template<class T>
bool Stack<T>::pop(T& item) {
    if (isEmpty()) return false;
    item = m_arr[--m_top];
    return true;
}
```

```c++
#include <iostream>
#include <string>
#include "Stack.cpp"

int main() {
    Stack<std::string> s;
    std::string x = "aaa";
    s.push(x);
    std::string y = "bbb";
    s.push(y);

    std::string c;
    s.pop(c);
    std::cout << c << std::endl;

    std::string b;
    s.pop(b);
    std::cout << b << std::endl;

    return 0;
}
```

## 48. 模版 实例化&具体化
```c++
// 1.隐式实例化
Stack<std::string> s;

// 2.显示实例化 
template <> class Stack<std::string>;

// 3.显示具体化
template <> class Stack<const char*> {
    //...
}
// 因为有些操作，指针类型和实际类型没法做一个统一模版，只能查分处理
Stack<int> s;
Stack<const char*> s;

// 4.部分具体化
// 只具体化第二个类型
template <class T1, class T2> class Pair;
template <class T1> class Pair<T1, int>;
```

## 49. 成员模版
```c++
// 即模版类中的成员的类型，使用另一个模版类创建
#ifndef BETA_H_
#define BETA_H_

#include <iostream>

template <typename T> class beta {
    private:
        template <typename V> class Hold {
            private:
                V val;
            public:
                Hold(V v=0) : val(v) {}
                V getVal() const {
                    return val;
                }
                void show() {
                    std::cout << val << std::endl;
                }
        };
        Hold<T> a;
        Hold<int> b;

    public:
        beta(T t, int x) : a(t), b(x) {}
        void show() {
            a.show();
            b.show();
        }
        template <typename U>
            U blab(U u, T t) {
                return (a.getVal() + b.getVal()) * u / t;
            }
};

#endif // BETA_H_
```

```c++
#include <iostream>
#include "beta.h"

int main() {
    beta<double> obj(2.1, 10);
    obj.show();

    double x = obj.blab(3.0, 4.0);
    std::cout << x << std::endl;
    return 0;
}
```

## 50. 模版当参数
```c++
#ifndef CRAB_H_
#define CRAB_H_

template <typename T>
class S {
    public:
        bool isEmpty() const;
        bool isFull() const;
        bool push(T&);
        bool pop(T&);
};

// int 和 double也可以改成参数
// template <template <typename T> class S, typename I, typename D>
template <template <typename T> class S>
class crab {
    private:
        S<int> s1;
        S<double> s2;

    public:
        bool push(int a, double b) {
            return s1.push(a) && s2.push(b);
        }

        bool pop(int& a, double& b) {
            return s1.pop(a) && s2.pop(b);
        }
};

#endif // CRAB_H_
```

```c++
#include <iostream>
#include "stack/Stack.cpp"
#include "crab.h"

int main() {
    // 相当于用Stack替换模版S
    crab<Stack> what;
    what.push(10, 5.5);

    int a = 20;
    double b = 3.5;
    what.pop(a, b);
    std::cout << a << ", " << b << std::endl;

    return 0;
}
```

## 51. 模版类的友元
```c++
// 
```

## 52. 友元类
```c++
// 类中的所以方法都是友元函数
```

## 53. 嵌套类
```c++
// 没啥可说的
```

## 54. 异常
```c++
#include <iostream>

int test(int, int);

int main() {
    int res = 0;
    try {
        res = test(9, 0);
    } catch(const char* e) {
        std::cout << "ming:" << e << std::endl;
    }
    std::cout << res << std::endl;
    return 0;
}

int test(int a, int b) {
    if (b == 0) {
        throw "Error: b is zero!";
    }
    return a / b;
}
```

## 55. RTTI
```c++
#ifndef WD_H
#define WD_H

#include <string>
#include <iostream>

class Wolf {
    protected:
        std::string m_name;
    public:
        Wolf(const std::string name) : m_name(name) {

        }

        virtual void showName() {
            std::cout << "wolf-->" << m_name << std::endl;
        }
};

class Dog : public Wolf {
    public:
        Dog(const std::string name) : Wolf(name) {

        }

        void showName() {
            std::cout << "dog-->" << m_name << std::endl;
        }
};

#endif //WD_H


```

```c++
#include <iostream>
#include "wd.h"

int main() {
    Wolf* f = new Wolf("haka");
    f->showName();

    Wolf* d = new Dog("bulu");
    d->showName();

    Dog* dp = dynamic_cast<Dog*>(d);
    std::cout << dp << std::endl;

    std::string* ep = dynamic_cast<std::string*>(d);
    std::cout << ep << std::endl;

    std::cout << "-------------" << std::endl;

    int res = typeid(d) == typeid(f);
    std::cout << res << std::endl;

    int res1 = typeid(d) == typeid(ep);
    std::cout << res1 << std::endl;

    return 0;
}
```

## 56.类型转换
```c++
// 1. dynamic_cast: 父子类的转换 

// 2. const_cast: 同种类型的带const和不带const的转换(包含volatile)
    const Wolf* a = new Wolf("haka");
    Wolf* b = const_cast<Wolf*>(a);

// 3. static_cast: 隐式转换合法时，才能用

// 4. reinterpret_cast: 危险的转换
struct {short a; short b} dat;
long val = 0xA0123456;
dat* val = reinterpret_cast<dat*>(&val);
```

## 57. string类
```c++
#include <iostream>
#include <string>

using namespace std;
int main() {
    string s1("wha's hhhh aabb ends well");
    cout << s1 << endl;

    string s2(10, 'a');
    cout << s2 << endl;

    string s3(s1);
    cout << s3 << endl;

    string s4 = "b";
    s4 += s3;
    cout << s4 << endl;

    cout << s1[1] << endl;

    string s5(s1, 10);
    cout << s5 << endl;

    string s6(&s1[2], &s1[11]);
    cout << s6 << endl;

    string s7(s1, 2, 12);
    cout << s7 << endl;
    return 0;
}
```

```
wha's hhhh aabb ends well
aaaaaaaaaa
wha's hhhh aabb ends well
bwha's hhhh aabb ends well
h
 aabb ends well
a's hhhh
a's hhhh aab
```

## 58. getline
```c++
#include <iostream>
#include <fstream>
#include <string>

int main() {
    std::ifstream fin;
    fin.open("test.cpp");
    if (fin.is_open() == false) {
        std::cout << "open file error!" << std::endl;
        return 1;
    }

    std::string tmp;
    getline(fin, tmp, ';');
    while (fin) {
        std::cout << tmp << std::endl;
        getline(fin, tmp, ';');
    }
    fin.close();
    std::cout << "Done" << std::endl;
    return 0;
}
```

## 59. 智能指针模版类
```c++
#include <iostream>
#include "Dog.h"
#include <memory>

int main() {
    Dog* dog = new Dog("gouzi");
    dog->show();
    {
        // unique_ptr
        // share_ptr
        std::auto_ptr<Dog> pd(new Dog("wana"));
        pd->show();
    }
    std::cout << "ending" << std::endl;
    return 0;
}
```

## 60. vector
```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include "Dog.h"

using namespace std;

void show(Dog& dog) {
    dog.show();
}

int main() {
    vector<Dog> list;
    Dog arr[] = {Dog("wawa1"), Dog("wawa2"), Dog("wawa3")};
    for (int i=0; i<3; i++) {
        list.push_back(arr[i]);
    }

    //vector<Dog>::iterator ip;
    //for (ip=list.begin(); ip!=list.end(); ip++) {
    //    ip->show();
    //}

    cout << "---------" << endl;
    for_each(list.begin(), list.end(), show);

    return 0;
}
```