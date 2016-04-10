## [C++11/14] decltype과 auto 반환

1. decltype

참고
https://en.wikipedia.org/wiki/Decltype
https://msdn.microsoft.com/ko-kr/library/dd537655.aspx
http://en.cppreference.com/w/cpp/language/decltype
[A note about decltype](http://www.drdobbs.com/cpp/a-note-about-decltype/231002789)

decltype은 declared type(선언된 형식)의 줄임말로써, 주어진 표현식의 타입을 알려주는 키워드이다.
TR1부터 도입된 decltype은 주로 주어진 템플릿 인자에 기반한 generic programming의 어려움을 해소하기 위해 도입되었다.

```c++
decltype(expression)
```

auto가 값에 상응하는 타입을 추론시켜주는 키워드라면,
decaltype은 값으로부터 타입을 추출해 낼 수 있는 키워드라고 생각하면 된다.
(두 키워드 모두 전적으로 컴파일 타임 기능인 것은 말해 입 아프다)

다음의 예제들을 살펴보자.

```c++
int x = 3;
 
// x의 타입은 int 이므로 아래 표현식은 int y = x와 같으며...
// 이런 경우엔 그냥 auto y = x;가 훨씬 쓰기도 보기도 좋다
decltype(x) y = x;
 
struct A
{
   double x;
};
const A* a = new A{0};
 
// 이 경우 a->x는 a의 멤버 엑세스로 처리되어, double 형식
decltype( a->x ) x3;
 
// 내부괄호로 (a->x)를 묶으면, 멤버 엑세스가 아닌 표현식으로 평가한다.
// 이 경우 a가 const pointer이므로, (a->x)는 const double& 형식이 된다
decltype((a->x)) x4 = x3;
```

지금까지의 예제만 보면, decltype의 효용성이 크게 와닿지 않는다.
뭐 변수를 선언할 때야 auto를 쓰는 것이 훨씬 편하고, 에지간히 복잡한 generic programming을 하지 않는 이상, 일반적으로 이 녀석을 사용해야 할 경우가 그리 많지 않기 때문이다.

2. decltype과 auto 반환 in C++11

C++11(VS2013)부터 auto 타입 반환이 가능해지고 trailing return type(후행 반환 형식)으로 decltype을 사용함으로써,
템플릿 함수의 auto 반환이 상당히 유연해지고 자유로워졌다.

```c++
// auto 반환함수와 후행 반환 형식으로 int 사용
auto add_function(int a, int b) -> int
{
    return a + b;
}
 
// auto 반환과 후행 반환 형식으로 decltype()을 사용
template <typename T, typename U>
auto add_template(T&& x, U&& y) -> decltype(std::forward<T>(x) + std::forward<U>(y))
{
    return std::forward<T>(x) + std::forward<U>(y);    
}
 
// BUILDER의 makeObject() 반환 형식으로부터 자유로워짐
template <typename TBuilder>
auto MakeAndProcessObject(const TBuilder& builder) -> decltype(builder.makeObject())
{
    auto val = builder.makeObject();
    // process...
    return val;
}
```

C++11에서 auto 반환 함수는 반드시 후행 반환 형식을 지정해 주어야 하며,
특히 템플릿 함수들의 경우 타입을 템플릿 인자들로부터 추론해야 하므로 decltype을 활용하지 않으면,
컴파일 단계에서 auto 반환 형식을 재대로 추론하지 못해 컴파일 에러가 발생하게 된다.

```c++
error: 'add_template' function uses 'auto' type specifier without trailing return type
 auto add_template(T x, U y)// -> decltype(x + y)
```

3. auto 반환 in C++14

헌데, C++14(VS2015)부터는 auto 반환시 후행 반환 형식을 지정해 주지 않아도 문제 없이 반환 타입을 추론해 준다.
즉, C++11에서 그랬듯 일일히 후행 반환 형식을 써주지 않아도 되는 것이다.

```c++
#include <iostream>
#include <string>
 
template <typename T, typename U>  
auto add_template(T&& x, U&& y) // -> decltype(std::forward<T>(x) + std::forward<U>(y))
{
    return std::forward<T>(x) + std::forward<U>(y);    
}
 
auto add_function(int a, int b) // -> decltype(a+b)
{
    return a + b;
}
 
template <typename TBuilder>
auto MakeAndProcessObject(const TBuilder& builder)      // -> decltype(builder.makeObject())
{
    auto val = builder.makeObject();
    // process...
    return val;
}
 
struct Builder
{
    static std::string makeObject() { return std::string("hello"); }
};
 
int main()
{
    add_template(1, 2);
    add_function(1, 2);
   
    // a is std::string type
    Builder builder;
    auto a = MakeAndProcessObject(builder);
}
```

C++14(VS2015)부터 바로 후행 반환 형식을 붙이지 않아도 되도록 발전되었기에,
decltype을 일반적으로 사용하는 빈도가 다시 적어지긴 했지만, decltype을 써야만 하는 곳들이 있을 것이므로 어떤 녀석인지는 알아두는 것이 좋을 듯 하다.