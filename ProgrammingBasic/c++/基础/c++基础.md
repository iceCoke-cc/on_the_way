## c++基础

[TOC]

### 命名空间：

```c+
namespace spaceA{
    int a = 10;
}
使用spaceA::a
```

C++对全局变量定义的增强：c中成功，C++中不允许

```c
int g_a; //bss段
int g_a = 20;　//data段
```

对struct增强：

```c
struct teacher {
    int age;
};
struct teacher t1; //c中
teacher t2; //c++中
```

c++更加强调类型，必须显示指明类型。

对bool类型增强

左值(能够被修改)、右值。

c++中的三目云算法可以当左值：( ( a< b) ? a : b ) = 10

const增强：不可通过指针来改变const变量的值。

在编译阶段，在data区有个符号表，key-value形式存储。没有地址。

\#define A 10　也是常量，不过在预处理阶段就处理了(无脑展开)。而const是在编译阶段处理。

```c++
const int a = 10;　//将a出现的地方，用10替换。在符号表中，没有地址，只读。
int *p = (int*)&a;//如果对一个常量取地址，编译器会临时开辟一个空间temp,让这个指针指向这个临时空间。
*p = 20;
//结果是 *p=20, a=10
```

枚举

```c++
enum season{
    SPR, //0
    SUM
};
enum season s = SUM;//不能像c那样用数字
```

引用：变量的别名 int&

- 声明它和原变量的关系，类型保持一致，且不分配内存，与被引用的变量有相同的地址。

- **声明的时候必须初始化**，一经声明，不可变更。常量也需要初始化。引用实现就是常量指针：int * const p=&a。编译器隐藏了取地址&和解地址*的操作。

- 可对引用再次引用，多次引用的结果就是多个别名

- &前有数据类型，则是引用，否则为取地址

- 引用所占大小跟指针相同

  ​

```c++
int a = 10, c = 20;
int &b = a;//b为int&类型，是a的引用
b = 50;
b = c;//赋值操作，而不是改变b的引用
```

在常量区的有：常量变量a，常量指针p，数组名array，引用。

```c++
int const a = 10;
int * const p = &b;
int array[10];　//array在常量区，指向栈区的10的int
int &r = b;
```

在函数返回值为变量、指针时，其实都是通过值拷贝，返回一个值，只能作为右值。

当函数返回一个引用时，即返回一个变量的别名，则可以作为左值、右值。

通过二级指针实现：

```c++
//因为传进来的是指针的地址，所以要使用二级指针
int get_mem(struct teacher** tpp){
    struct teacher *tp = NULL;
    tp = (struct teacher*)malloc(sizeof(struct teacher));
    if(  NULL == tp)
        return -1;
    tp->id = 100;
    strcpy(tp->name, "Duke");
    *tpp = tp;
    return 0;
}
void free_teacher(struct teacher **tpp){
    if( NULL == tpp )
        return;
    struct teacher *tp = *tpp;
    if( NULL != tp ){
        free(tp);
        *tpp = NULL;
    }
}
int main(){
    struct teacher *tp = NULL;
    get_mem(&tp);
    free_teacher(&tp);
}
```

通过引用实现：

```c++
//通过给类型为struct teacher*的引用，则函数内部的tp与外部的tp使用一样
int get_mem2(struct teacher * & tp){
    tp = (struct teacher*)malloc(sizeof(struct teacher));
    if( NULL == tp )
        return -1;
    tp->id = 100;
    strcpy(tp->name, "Duke");
    return 0;
}
void free_teacher(struct teacher * & tp){
    if( NULL != tp ){
        free(tp);
        tp = NULL:
    }
}
int main(){
    struct teacher *tp = NULL;
    get_mem2(tp);
    free_teacher2(tp);
}
```

如果要对向量引用，则必须是一个const引用。

```c++
const int a = 10;
const int & b = a;
```

普通变量也可以用const引用，但不能通过引用改变。但其自身可以改变。

对常量做引用，必须加const

```c++
int& a = 10;//错误
cosnt int & a = 10;//对常量10的内存空间取别名
```

### 函数指针

*优先级高于()

函数指针：在给函数指针赋值的时候就发生重载，调用时就直接调用指定的函数即可。

```c++
int func(int a, int b){return 0}
//方式一：定义一种函数类型
typedef int(MY_FUNC)(int, int)
MY_FUNC * fp = NULL;
fp = func;
fp(10,20);
(*fp)(10,20);
//方式二：定义一个指向函数类型的指针类型
typedef int(*MY_FUNC_P)(int, int)
MY_FUNC_P fp1 = NULL;
fp1 = func;
fp1(10,20);
//方式三：
int(*fp3)(int, int) = NULL;
fp3 = func; 
fp3(10,20);
```

```c++
void(*p)() = add;//函数指针
p();//调用
p=add;
p=&add
void(*p[3])() = {add, multi, divide};//函数指针　数组
p[1]();//调用
void fun(int x, int(*p)(int a, int b)){ //将函数作为参数传进去
    p(x,1);//回调函数
}
fun(1,add);
```



### inline函数

inline函数：

- 在编译阶段，直接将函数插入在函数调用的地方，导致出现副本，增加代码空间避免了普通函数调用的压栈、跳转、返回的开销。
- 是一种特殊函数，具有普通函数的特征(参数检查，返回类型等)
- 内联函数由编译器处理，会进行分析；宏代码由预处理器处理，简单的文本替换。
- 如果函数内部代码量过大，过于复杂，展开的开销太大甚至超过压栈开销时，则直接当做普通函数处理。所以内联函数不能有循环语句、过多的条件判断、过大的函数体、不能对函数进行取地址操作。
- 使用场景：函数体很小、频繁使用的函数

inline void func(){}

占位参数：亚元

```c++
void func(int){} //
void func(int=10){} //默认为１０
```

函数重载：

函数的类型：函数的返回值、函数形参列表（参数个数、参数类型、参数顺序）

c语言中：只要函数名字相同，就是函数重定义

函数重载：**函数名相同，但参数列表不同**。与返回值无关。**最好不要写默认参数**，防止歧义。

编译器调用重载函数的准侧：调用时发生重载。

1. 将所有同名函数作为候选者
2. 精确匹配
3. 默认参数匹配
4. 隐式类型转换匹配
5. 若出现二义性，则匹配失败，编译失败。

重载底层实现：name mangling倾轧技术来改名函数名，区分参数不同的同名函数，用每个类型首字母表示。

```c++
void func(char a); // func_c(char a)
void func(char a, int b, double c); // func_cid(char a, int b, double c)
```



### 数组类型

```c++
//一、先定义一个数组类型，然后再定义一个该类型的指针指向一个数组
typedef int(ARRAY_INT_10)[10];
int array[10]; //array是一个指向int类型的指针
ARRAY_INT_10 *array_10_p = &array;
(*array_10_p)[i];//只能通过这种方式来访问，因为array_10_p+1步长为真个数组大小，跟数组名等价。
//二、定义一个数组类型的指针
typedef int(*ARRAY_INT_10_P)[10];
ARRAY_INT_10_P array_10_p = &array;
(*array_10_p)[i];
//三、指向数组的指针,是一个指针
int(*p)[10] = &array;
(*p)[i];
//补充
int*p[10];//指针数组，是一个数组
```



### 数据类型转换

c++是严格的数据类型。所以数据转换较为安全。

1. static_cast：内置的基本类型转换，具有继承关系的指针/引用（父转子、子转父）。

```c++
int a = 10;
char c = static_cast<char>(a);
```

2. dynamic_cast：转换具有**继承关系**的指针/引用，在转换前会进行对象类型安全检查（父类转为子类不安全），即**只能由子类转为父类**。

3. const_cast：对指针／引用／对象指针取消掉/添加const。

   ```c++
   int a = 10;
   const int& b = a;
   int& c = const_cast<int&>(b);
   c = 20;    
   ```

4. reinterpret_cast：强制类型转换，无关的指针类型都可以转换，包括函数指针。



### 异常

- 函数的返回值可以忽略，但异常不能忽略，没有捕获异常，就会终止。异常作为一个类，可以传递足够的信息。
- c++异常跨函数，底层函数抛出异常，上级函数没有捕获的话，会自动往上抛，若main函数中还没有捕获，则终止。

```c++
int divide(int x, int y){
    Person p;
    if ( 0 == y )
        throw y; //抛一个int类型异常。抛出时，前面的对象将回收/析构（占解旋）
    return x / y;
}
void test01(){
    try{
        divide(10,0);
    } catch(int e){　//捕获int类型异常
        cout << "捕获到整形异常" << e << endl;
    } catch(...){ //捕获所有异常
        
    }
}
```

```c++
//该函数只能抛出指定的异常，否则报错。
void func() throw(int, float, char){
    
}
void func1 throw(){}//不能抛出任何异常
void func2{}//可以抛出任何类型的异常

```

异常对象的声明周期：

最**好使用引用，只会调用一次构造**。使用指针时需要谨慎。

1. 普通类型：会调用构造函数和拷贝构造，共两个对象，都在catch之后析构

```c++
void func(){
    throw MyException(); // 1.创建匿名对象
}
void test(){
    try{
        func();
    }
    catch (MyException e){ // 2.通过拷贝构造创建对象e
        cout << "异常捕获"　<< endl;
    } // 3.调用匿名对象和e的析构
}
```

2. 引用类型：会调用构造函数，共1个对象，都在catch之后析构

```c++
void func(){
    throw MyException(); // 1.创建匿名对象
}
void test(){
    try{
        func();
    }
    catch (MyException& e){ // 2.将匿名对象提升为e
        cout << "异常捕获"　<< endl;
    } // 3.调e的析构
}
```
3. 1指针类型：会调用构造函数，共1个对象，但是在catch之前析构

```c++
void func(){
    throw &(MyException()); // 1.创建匿名对象
}
void test(){
    try{
        func();
    }
    catch (MyException* e){ // 2.析构e，无法在后面使用该对象
        cout << "异常捕获"　<< endl;
    } 
}
```
3. 2指针类型：会调用构造函数，共1个对象，但是在catch之前析构

```c++
void func(){
    throw new MyException(); // 1.手动创建对象，需要手动释放。
}
void test(){
    try{
        func();
    }
    catch (MyException* e){ // 2.使用e
        cout << "异常捕获"　<< endl;
        delete e; //3. 手动释放
    } 
}
```

c++标准库的异常：exception是所有异常类的父类。重载父类的what函数和虚析构函数。

![creenshot from 2018-12-19 14-39-0](/home/quandk/Pictures/Screenshot from 2018-12-19 14-39-07.png)

```c++
class MyOutOfRange : public exception{
public:
    char* pError;
public:
    MyOutOfRange(char* error){
        pError = new char[strlen(error)+1];
        strcpy(pError, error);
    }
    ~MyOutOfRange(){
        if( NULL != pError ){
            delete[] pError;
        }
    }
    vitrual const char * what() const{
        return pError;
    }
}
```



### 输入输出

p759

cerr没有缓冲区，clog有缓冲区。键盘上的EOF：ctrl+z

- cin.get() :一次读一个字符。键盘回车时，输入的字符才放入缓冲区。
- cin.get(ch)：读取一个字符
- cin.get(buf, 256)：从缓冲区读一个字符串
- cin.getline(buf,256); 读一行数据，不读取换行符

```c++
 
```

#### 文件操作

```c++
//文本文件，windows下将\r\n转为\n，linux则不用转。
char* fileName = "a.txt";
ifstream ism(fileName, iso:im); //只读方式打开文件
if( !ism ){
    return;
}
char ch;
while(ism.get(ch)){
    cout<<ch;
}
ism.close();
ofstream osm(fileName, iso:out);//写 iso:out | iso:app 追加
osm.put(ch);
osm.close();
```
对象序列化

```c++
//二进制文件
Person p(10);//内存中为二进制
ofstream osm(fileName, iios::out|iso::binary); //二进制写
osm.wirte((char*)&p, sizeof(Person));
osm.close;
//读
ifstream ism(fileName, ios::in | ios::binary);
Person p;
ism.read((char*)&p, sizeof(Person));
p.show();
ism.close;
```

















