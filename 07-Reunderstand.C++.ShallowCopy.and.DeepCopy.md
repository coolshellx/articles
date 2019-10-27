### 目录

[前言](#pre)

[C++ 的复制机制](#a)

[默认拷贝构造函数](#b)

[拷贝构造函数和赋值操作符](#c)

[浅拷贝](#d)

[深拷贝](#e)

[C++11 右值引用和 move 语义](#f)


### 1、<span id = "pre">前言</span>

最近在找工作面试的过程中，发现面试官经常问到了 C++ 的浅拷贝和深拷贝这个问题，在这里总结一下自己的一些理解和实践，希望对有需要的别人带来一些帮助。

下面我分七个部分进行介绍，分别是：C++ 的复制机制，默认拷贝构造函数，拷贝构造函数和赋值操作符，浅拷贝，深拷贝，C++11 右值引用和 move 语义。

### <span id = "a">2、C++ 的复制机制</span>

C++ 中对象的复制就如同”克隆“，用一个已有的对象快速复制出多个完全相同的对象。一般而言，以下三种情况都会使用对象的复制。



> （1）建立一个新对象，并用另一个同类的已有对象对新对象进行初始化，例如：

```c++
class Base
{
private:
    int w;
    int h;
};
int main(int argc,const char *argv[])
{
    Base b1;
    base b2(b1);  // 使用b1初始化b2，此时会进行对象的复制
    return 0;
}
```



> （2） 当函数的参数是类的对象时，调用此函数使用的是值传递，也会发生对象的复制，例如：

```c++
void fun1(Base base)
{
	//TODO
}
int main(int argc,const char *argv[])
{
    Base b1;
    fun1(b1);  // 此时会发生对象的复制
    return 0;
}
```



> （3） 当函数的返回值是类的对象时，当函数调用结束时，需要将函数的对象复制到一个临时的对象并传给该函数的调用处，也会发生对象的复制，例如：

```c++
Base fun2()
{
	//TODO
	Base base;
	return base;
}
int main(int argc,const char *argv[])
{
    Base b2;
    b2 = fun2(); 
    // 在 fun2 返回对象时，会执行对象复制，复制出一临时对象，然后将此临时对象“赋值”给 b2
    return 0;
}
```

注意到：对象的复制都是通过一种特殊的构造函数来完成的，这种特殊的构造函数就是拷贝构造函数 **（copy constructor，也叫复制构造函数）。**

拷贝构造函数在大多数情况下都很简单，甚至在我们都不知道它存在的情况下也能很好发挥作用，但是在一些特殊情况下，**特别是在对象里有动态成员，或者指针类型变量的时候**，就需要我们特别小心地处理拷贝构造函数了。下面我们就来看看拷贝构造函数的使用。



### <span id = "b"> 3、默认拷贝构造函数</span>
 
很多时候在我们都不知道拷贝构造函数的情况下，传递对象给函数参数或者函数返回对象都能很好的进行，这是因为 **编译器会给我们自动产生一个拷贝构造函数**，这就是 **“默认拷贝构造函数”** ，这个构造函数很简单，仅仅使用“老对象”的数据成员的值对“新对象”的数据成员一一进行赋值，它一般具有以下形式：

```c++
Base::Base(const Base& t)
{
	w = t.w;
	h = t.h;
}
```

当然了，有人说，我怎么一般没实现这个函数呢，照样可以进行类的复制操作啊，这是因为以上代码不用我们编写，编译器会为我们自动生成。但是如果认为这样就可以解决对象的复制问题，那就错了，让我们来考虑以下一段代码：

```c++
//Author:rongweihe
//Date  :2019/04/20
#include <bits/stdc++.h>
using namespace std;
//浅拷贝：对于基本类型的数据和简单的对象，它们之间的拷贝非常简单；就是按位复制内存
class Base
{
public:
    Base()
    {
        c++;
    }
    ~Base()
    {
        cout<<"调用析构函数"<<endl;
        c--;
	cout<<"~c="<<c<<endl;
    }
    static int getC()
    {
        return c;
    }

private:
    int a;
    int b;
    static int c;
};
int Base::c = 0;

int main()
{
    //freopen("in.txt","r",stdin);

    Base obj1;
    cout<<"obj1.getC()="<<obj1.getC()<<endl;
    Base obj2 = obj1;
    cout<<"obj2.getC()="<<obj2.getC()<<endl;
    return 0;
}
```

这段代码对前面的类进行一些修改，加入了一个计数器静态成员，目的是进行计数，统计创建的对象的个数。在每个对象创建时，通过构造函数进行递增，在销毁对象时，通过析构函数进行递减。

运行结果

-    obj1.getC()=1
-    obj2.getC()=1
-    调用析构函数
-    ~c=0
-    调用析构函数
-    ~c=-1

注意到，计数器变为负数了，这是为什么呢？

在主函数中，首先创建对象 obj1，输出此时的对象个数，然后使用 obj1 复制出对象 obj2，再输出此时的对象个数，按照理解，此时应该有两个对象存在，但实际程序运行时，输出的都是 1，反应出只有 1 个对象。

在销毁对象时，会调用销毁两个对象，类的析构函数会调用两次，此时的计数器将变为负数。**出现这些问题最根本就在于在复制对象时，计数器没有递增**，解决的办法就是重新编写拷贝构造函数，在拷贝构造函数中加入对计数器的处理，形成的拷贝构造函数如下：

```c++
class Base
{
public:
    Base()
    {
        c++;
    }
    ~Base()
    {
        cout<<"调用析构函数"<<endl;
        c--;
	cout<<"~c="<<c<<endl;//输出每次析构之后计数器值
    }
    // 拷贝构造函数
    Base(const Base& t)
    {
        a=t.a;
        b=t.b;
        c++;
    }
    static int getC()
    {
        return c;
    }

private:
    int a;
    int b;
    static int c;
};
int Base::c = 0;
```

运行结果

- obj1.getC()=1
- obj2.getC()=2
- 调用析构函数
- ~c=1
- 调用析构函数
- ~c=0



### <span id = "c"> 4、拷贝构造函数和赋值操作符</span>

拷贝构造函数和赋值操作符的行为比较相似，都是将一个对象的值复制给另一个对象，但是其结果却有些不同。

拷贝构造函数使**用传入对象的值生成一个新的对象的实例**，而赋值操作符是将对象的值复制给一个**已经存在的实例**。

这种区别从两者的名字也可以分辨出来：拷贝构造函数也是一种构造函数，那么它的功能就是创建一个新的对象实例；赋值操作符是执行某种运算，将一个对象的值复制给另一个对象（已经存在的）。

**要区分调用的是拷贝构造函数还是赋值操作符，主要是看是否有新的对象实例产生。如果产生了新的对象实例，那调用的就是拷贝构造函数；如果没有，那就是对已有的对象赋值，调用的是赋值操作符**。

调用拷贝构造函数主要的场景也在开头说过了：

- 使用一个对象给另一个对象初始化。

- 对象作为函数的参数，以值传递的方式传给函数。　
- 对象作为函数的返回值，以值的方式从函数返回。

下面来具体实践一下

```c++
#include <bits/stdc++.h>
using namespace std;
class Base
{
public:
    Base() {}
    Base(const Base& b)
    {
        cout<<"调用拷贝构造函数"<<endl;
    }
    Base& operator=(const Base& b)
    {
        cout<<"调用赋值操作符"<<endl;
        return *this;
    }
    ~Base(){}

private:
    int age;
    string name;
};

void f1(Base b)
{
    return;
}

Base f2()
{
    Base b;
    return b;
}

int main()
{
    Base b1;Base b2 = b1;   // 1

    Base b3;b3 = b1;        // 2

    f1(b3);                 // 3

    b3 = f2();              // 4

    Base b4 = f2();         // 5

    return 0;
}
```

上面代码在 Base 类中，显式的定义了拷贝构造函数和赋值运算符。然后定义了两个函数：f1，f2。

f1 以值的方式参传入 Base 对象；f2 以值的方式返回Base对象。

在 main 函数中模拟了 5 种情况，测试调用的是拷贝构造函数还是赋值运算符。我们先不看结果，分析一下正常情况下应该是如何输出的：



> 分析：
>
> - 1、对应于开头说的第一种复制机制，使用对象 b1 来创建一个新的对象 b2。也就是产生了新的对象，所以调用的是拷贝构造函数。
> - 2、首先声明一个对象 b3，然后使用赋值运算符"="，将 b1 的值复制给 b3，显然是调用赋值运算符，为一个已经存在的对象赋值。
> - 3、对应于开头说的第二种复制机制，以值传递的方式将对象 b3 传入函数 f1 内，调用拷贝构造函数构建一个函数 f1 可用的实参。
> - 4、对应于开头说的第三种复制机制，这条语句拷贝构造函数和赋值运算符都调用了。函数 f2 以值的方式返回一个 Base 对象，
>   在返回时会调用拷贝构造函数创建一个临时对象 tmp 作为返回值；返回后调用赋值运算符将临时对象 tmp 赋值给 b3。
> - 5、按照 4 的逻辑，应该是首先调用拷贝构造函数创建临时对象；然后再调用拷贝构造函数使用刚才创建的临时对象创建新的对象 b4，也就是会调用两次拷贝构造函数。



实际执行结果如下：

> - 调用拷贝构造函数
> - 调用赋值操作符
> - 调用拷贝构造函数
> - 调用赋值操作符


结果和输出前的分析不一致，

相似的代码，我在网上看到，有网友跑出的结果如下

![439761-20161207163440429-300030531.png](https://i.loli.net/2019/05/02/5cca9aab3490c.png)

我怀疑是编译器的问题，我在 https://wandbox.org/ 这个在线网站上，分别使用了GCC-4.4.7 - GCC-8.3.0版本编译上面的代码，得到的结果跟我本地运行结果是一样的。

我觉得出现这种现象的原因是：代码在不同的平台，编译器会做不同的优化，导致打印的结果不同，4 部分仅仅打印了 “调用赋值操作符”，然后 5 部分什么都没打印，说明编译器自身做了优化，把 5 当做了默认构造函数。

### <span id = "d"> 5、浅拷贝</span>

**浅拷贝:** 指的是在对象复制时，只是将对象中的数据成员进行简单的赋值，上面的例子都是属于浅拷贝的情况，**默认拷贝构造函数执行的也是浅拷贝**。大多情况下“浅拷贝”已经能很好地工作了，但是一旦对象存在了动态成员，那么浅拷贝就会出问题了，让我们考虑如下一段代码：

```c++
#include <bits/stdc++.h>
using namespace std;
class Base
{
public:
    Base()//构造函数，p 指向堆中分配的空间
    {
        cout<<"new"<<endl;
        p = new int(100);
    }
    ~Base()//析构函数，释放动态分配的空间
    {
        if(p!=NULL)
        {
            delete p;
            cout<<"delete 的地址= "<<p<<endl;
        }
    }
private:
    int w;
    int h;
    int *p;
};

int main()
{
    //freopen("in.txt","r",stdin);
    Base obj1;
    Base obj2 = obj1;//复制对象
    return 0;
}
```

运行结果

> new
> delete 的地址= 0x62b228
> delete 的地址= 0x62b228

**p 指针被分配一次内存，但是程序结束时该内存却被释放了两次，会造成内存泄漏问题！**

原因就在于在进行对象复制时，对于动态分配的内容没有进行正确的操作。我们来分析一下：

在运行定义 obj1 对象后，由于在构造函数中有一个动态分配的语句，因此执行后的内存情况大致如下：

![深拷贝001.jpg](https://i.loli.net/2019/04/20/5cbad59365932.jpg)

在使用 obj1 复制 obj2 时，由于执行的是浅拷贝，只是将成员的值进行赋值，所以此时 obj1.p 和obj2.p 具有相同的值，也即这两个指针指向了堆里的同一个空间，如下图所示：

![深拷贝002.jpg](https://i.loli.net/2019/04/20/5cbad6401bb85.jpg)

当然，这不是我们所期望的结果，在销毁对象时，两个对象的析构函数将对**同一个内存空间释放两次**，这就是错误出现的原因。我们需要的不是两个 *p* 有相同的值，而是两个 *p* **指向**的空间有相同的值，解决办法就是使用“深拷贝”。



### <span id = "e"> 6、深拷贝</span>

在“深拷贝”的情况下，对于对象中动态成员，就不能仅仅简单地赋值了，而应该重新动态分配空间，如上面的例子就应该按照如下的方式进行处理：

```c++
#include <bits/stdc++.h>
using namespace std;
class Base
{
public:
    Base()//构造函数，p 指向堆中分配的空间
    {
        p = new int(100);
        cout<<"new1 and &p = "<<p<<endl;
    }
    Base(const Base& t)
    {
		w = t.w;
		h = t.h;
		p = new int;	// 为新对象重新动态分配空间
		*p = *(t.p);
		 cout<<"new2 and &p = "<<p<<endl;
    }
    ~Base()//析构函数，释放动态分配的空间
    {
        if(p!=NULL)
        {
            delete p;
            cout<<"delete 的地址= "<<p<<endl;
        }
    }
private:
    int w;
    int h;
    int *p;
};

int main()
{
    //freopen("in.txt","r",stdin);
    Base obj1;
    Base obj2 = obj1;//复制对象
    return 0;
}
```

运行结果

> new1 and &p = 0x3cb228
> new2 and &p = 0x3cb238
> delete 的地址= 0x3cb238
> delete 的地址= 0x3cb228

  此时，在完成对象的复制后，内存的一个大致情况如下：

![深拷贝003.jpg](https://i.loli.net/2019/04/20/5cbad8f1bd8d6.jpg)

注意到，此时 obj1 的 p 和 obj2 的 p 各自指向一段内存空间，但它们指向的空间具有相同的内容，这就是所谓的“深拷贝”。




### <span id = "f"> 7、C++11 新特性：右值引用与 move 语义</span>




[C++11 ](https://zh.wikipedia.org/wiki/C%2B%2B11) 的 **右值引用** 是一个颇为重要的新特性，解决了C++ 中一个广为诟病的性能问题。它实现了转移语义 (Move Sementics) 和精确传递 (Perfect Forwarding)。它的主要目的有两个方面：

> 1. 消除两个对象交互时不必要的对象拷贝，节省运算存储资源，提高效率。
> 2. 能够更简洁明确地定义泛型函数。

右值引用特性允许我们对右值进行修改。借此可以实现 **move语义**： **右值不需要被复制直接传递给构造函数，操作结束后空的右值析构也不会销毁内存。**

在 C++ 之前的标准中，右值是不允许被改变的，实践中也通常使用 `const T&` 的方式传递右值。然而这是效率低下的做法，例如：

```c++
Base get(){
    Base p;
    return p;
}
Base p = get();
```

上述获取右值并初始化 `p` 的过程包含了 `Base` 的 3 个构造过程和 2 个析构过程。 使用**右值引用**的特性我们可以避免其中不必要的内存拷贝，从右值中直接拿数据过来初始化或修改左值。 一个 `move` 构造函数是这样声明的：

```c++
class Person{
public:
    Person(Person&& rhs){...}
    ...
};
```



#### 左值和右值

C++ 表达式的值有三种类别：左值、右值和临终值。 其中**左值**是指在表达式的外部保留的对象，可以将左值视为有名称的对象，所有的变量都是左值。 **右值**是一个不在使用它的表达式的外部保留的临时值，比如函数的返回值、字面常量。 **临终值**是生命周期已经结束，但内存仍未回收的值，比如函数的返回值可以声明为 `int&&`。 若要更好地了解左值和右值之间的区别，看下面的示例：

```c++
int a = 3;              // a 是变量，所以它是一个左值
                        // 3 是字面常量，所以它是一个右值
int b = a;              // b 是变量，也是一个左值。a 是有名称的，也是一个左值
b = (a + 1);            // (a + 1) 是一个右值，它是一个背后的没有名称的值
b = getValue();         // getValue() 的返回值是一个右值，他没有名称
```

> 可以通过表达式的值是否可以取地址来判断左值还是右值。左值都是可以取地址的。

右值的特点在于它 **不被后续计算所需要**，因为它连名字都没有，程序中无法再次访问一个右值。

**左值的声明符号为”&”， 为了和左值区分，右值的声明符号为”&&”。**

例子代码

```c++
void process_value(int& i) { 
 std::cout << "LValue processed: " << i << std::endl; 
} 
 
void process_value(int&& i) { 
 std::cout << "RValue processed: " << i << std::endl; 
} 
 
int main() { 
 int a = 0; 
 process_value(a); 
 process_value(1); 
}
运行结果 :
LValue processed: 0 
RValue processed: 1
```



#### 右值的重复拷贝

右值虽然是不被后续计算所需要的，但它仍然需要构造和析构。 这在 C++ 中造成了不少的代价，下面来看具体的例子：

```c++
class Base{
    char* name;
public:
    Base(const char* p){
        size_t n = strlen(p) + 1;
        name = new char[n];
        memcpy(name, p, n);
    }
    Base(const Base& p){
        size_t n = strlen(p.name) + 1;
        name = new char[n];
        memcpy(name, p.name, n);
    }
    ~Base(){ delete[] name; }
};
```

当我们拷贝 `Base` 对象时，会有额外的不需要的内存分配过程，例如：

```c++
Base getAlice(){
    Base p("alice");      // 对象创建。调用构造函数，一次 new 操作
    return p;             // 返回值创建。调用拷贝构造函数，一次 new 操作
                          // p 析构。一次 delete 操作
}
int main(){
    Base a = getAlice();  // 对象创建。调用拷贝构造函数，一次 new 操作
                          // 右值析构。一次 delete 操作
    return 0;
}                         // a 析构。一次 delete 操作
```

正常情况下，会执行三次构造函数三次析构函数。**返回值优化** 和 **move语义** 便是用来避免这些不必要的构造过程和动态内存操作的。



#### 返回值优化

事实上编译器会对上述代码进行 [返回值优化](https://zh.wikipedia.org/wiki/返回值优化)，其实这里是 **返回值优化（RVO）**，可以减少两次拷贝构造。 上述代码其实只需要一次构造和一次析构。为了让代码更加清晰，我们来看去掉动态内存相关代码的`Base`类：

```c++
struct Base{
    Base(const char* p){
        cout<<"constructor"<<endl;
    }
    Base(const Base& p){
        cout<<"copy constructor"<<endl;
    }
    const Base& operator=(const Base& p){
        cout<<"operator="<<endl;
        return *this;
    }
    ~Base(){
        cout<<"destructor"<<endl;
    }
};

Base getAlice(){
    Base p("alice"); return p;
}

int main(){
    cout<<"______________________"<<endl;
    Base a = getAlice();
    cout<<"______________________"<<endl;
    a = getAlice();
    cout<<"______________________"<<endl;
}
```

程序输出是：

![001.PNG](https://i.loli.net/2019/05/02/5ccaa914d437b.png)

```
______________________
constructor             // 1) getAlice 里的 p 被构造
______________________
constructor             // 2) getAlice 里的 p 被构造
operator=               // 3) 右值赋值给左值
destructor              // 4) 右值析构
______________________
destructor              // 5) a 析构
```



可见上述代码经过 RVO 之后，返回值没有被拷贝：

- 对于赋初值运算，甚至连 `a` 的拷贝构造函数都没有执行，直接使用了 `getAlice` 里的对象。
- 对于赋值运算符，虽然没有拷贝返回值，但 `operator=` 还是执行了的。



## move 语义

右值虽然是临时的，但程序仍然调用了拷贝构造和拷贝赋值，造成了没有意义的资源申请和释放的操作。如果能够直接使用临时对象已经申请的资源，既能节省资源，有能节省资源申请和释放的时间。这正是定义转移语义的目的。右值引用的语法是`&&`：

```c++
const Base& operator=(Base&& rhs){
    cout<<"move operator="<<endl;
    delete[] name;
    name = rhs.name;
    rhs.name = nullptr;
    return *this;
}
```

注意这里的 `rhs.name`要设为空指针，这样编译器就不会去`delete`它了。同样地，我们把拷贝构造函数也声明为 move 拷贝构造函数：

```c++
Base(Base&& p){
    cout<<"move copy constructor"<<endl;
    name = p.name;
    p.name = nullptr;
}
```

然后在来重新执行`main`函数，可以得到输出：

![002.PNG](https://i.loli.net/2019/05/02/5ccaac23d3c11.png)

```
______________________
constructor             // 1) getAlice 里的 p 被构造
______________________
constructor             // 2) getAlice 里的 p 被构造
move operator=          // 3) 右值赋值给左值，调用move赋值运算符！
destructor              // 4) 右值析构
______________________
destructor              // 5) a 析构
```

注意：因为拷贝构造函数已经被 **返回值优化** 掉了，所以 move 拷贝构造函数也不会调用。



<br/>

<br/>

参考：

https://blog.csdn.net/bluescorpio/article/details/4322682；

https://harttle.land/2015/10/11/cpp11-rvalue.html；

https://www.ibm.com/developerworks/cn/aix/library/1307_lisl_c11/index.html

《*C++*编程思想 第*1*卷》   *Bruce Eckel*。



<br/>

<br/>
