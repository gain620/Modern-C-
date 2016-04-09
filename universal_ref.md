## Universal Reference

<img src="http://static.tumblr.com/f7d551f573e8f94213536d8a0ceb1881/wqrjyce/yNRmq27fy/tumblr_static_tumblr_static_ayanami_rei_neon_genesis_evangelion_desktop_1920x1080_hd-wallpaper-1071766s.jpg" width="100">

<br/>


lvalue는 지속성이 있는 값이고, rvalue는 지속성이 없는 값을 의미한다. 또는 lvalue는 수식에서 identity를 가지는 value를 의미하며, rvalue는 identity를 가지지 않는 값을 rvalue라고도 한다. 다시 말하면 주소를 취할 수 있으면 lvalue, 취할 수 없으면 rvalue이다.

인터넷에 많은 예제와 함께 lvalue와 rvalue에 대해서 나오므로 예제와 함께한 설명은 넘어가겠다. 이렇게 lvalue를 참조하는 것을 lvalue reference, rvalue를 참조하는 것을 rvalue reference라고 한다. 

하지만 modern C++에서는 universal reference 라는 개념을 알아둘 필요가 있다. (C++ 포럼에서는 forward reference라고 사용하는데 Scott Myers(effective C++ 저자)는 forward reference말고 universal reference라고 사용하는데 자신이 만든 명칭이라 한다.) universal은 전 세계의, 보편적인, 우주의 등등 사전적 의미가 있는데, modern C++에서는 보편적인란 단어가 어울리겠다. 즉, 보편적인 참조라는 뜻이다. 즉, lvalue도 될 수 있고, rvalue가 될 수도 있는 참조형식이라는 것이다. 한가지 예를 들어보겠다.


```c++
template<typename T>
void f(T&& arg)
{
 
}
```

위의 코드에서 T&&는 universal reference이다. 즉, arg는 rvalue reference가 될 수도 있고, lvalue reference가 될 수도 있다. 위 함수를 아래와 같이 사용해보자.

```c++
int x = 27;
const int cx = x;
const int& rx = x;
 
f(x);
f(cx);
f(rx);
f(27);
```

f(x)의 경우, x는 lvalue이므로 T&&는 int&가 된다. 즉, T&&는 lvalue reference가 된 것이다.
f(cx)의 경우도 cx는 lvalue이므로 T&&는 const int&가 된다. lvalue reference가 되겠다.
f(rx)의 경우도 rx는 lvalue이므로 lvalue reference이다.
f(27)에서 27은 rvalue이다. 이 경우 T&&는 int&&가 되므로 rvalue reference가 된 것이다.
template code 한줄로 lvalue reference도 rvalue reference도 수용가능하게 되었다.


그렇다면 언제 universal reference가 되는 것일까? 눈치챈 사람도 있겠지만 universal reference는 타입 추론이 발생하게 되는 경우에만 적용이 된다.

```c++
template<typename T>
void f1(T&& arg) {}
 
void f2(int&& arg) {}
```

f1함수는 template함수로써 T에 대한 타입 추론이 발생하게 된다.  즉, 이럴 경우 T&&가 lvalue reference인지 rvalue reference인지 compile 타임에서 파악하게 된다.

f2함수는 이미 타입이 int형으로 정해져있다. 이럴 경우 타입추론을 하지 않게 되니 int&&는 rvalue reference가 되는 것이다. 그래서 위의 universal reference 예제 처럼 f2 함수는 아래와 같이 사용할 수가 없다.

```c++
int a = 10;
f2(a);
```

a는 lvalue이고 f2파라미터는 rvalue reference이기 때문에 컴파일 에러가 발생한다. 자, 그렇다면 T&&(타입추론에 의한&&)라면 모두 universal reference일까? 아래의 경우를 보자.

```c++
template<typename T>
class Foo
{
    ...
    void Bar(T&& arg) { }
}
```
위와 같은 클래스의 Bar 멤버 함수에서 T&&는 universal referenece가 아닌 rvalue reference가 된다. 왜일까, 조금만 생각하면 답을 알 수 있다. 객체 생성 시 이미 타입이 정해졌기 때문이다.

```c++
Foo<int> foo;
```
객체 생성시 T는 int가 되었기 때문에 Bar(T&&)는 Bar(int&&)가 되어버린 것이다. 그러면 어떻게 해야할까,

```c++
template<typename T>
class Foo
{
    ...
    void Bar(T& arg) { }
    void Bar(T&& arg) { }
}
```
멤버 함수 overloading을 통해서 해결할 수 있다. 그러나 overloading을 사용하는 방법은 엄청난 노동을 필요로 하는 경우가 발생한다. Bar멤버함수에 파라미터가 두개 이상이라면? 우리는 파라미터의 ^2에 해당하는 멤버함수가 필요할 것이다. 사실 universal reference로 만드는 방법이 있다.

```c++
template<typename T>
class Foo
{
public:
    template<typename T>
    void Bar(T&& arg) { }
}
```
Bar함수 전에 template을 명시해주면 된다. 그러면 T&&는 universal reference가 된다. 그러나 여기서 알아야 할 것은 class에서 명시한 T와 멤버함수에서 명시한 T는 타입이 다를 수 있다는 것이다. 같은 T라도 타입이 똑같다고 컴파일러는 보장해주지 않는다는 것이다. 전적으로 사용자의 책임이 되는 것이다. 두개 T에 대해서 같은 타입을 보장해주고 싶다면 타입검사가 필요할 것이다. 
```c++
template<typename T>
class Foo
{
public:
    using value_type = T;
    template<typename T>
    void Bar(T&& arg)
    {
        if (typeid(value_type) == typeid(T))
        {
            std::cout << "equal" << std::endl;
        }
    }
}
```
variadic template과 universal reference, std::forward를 사용하면 극강의 코드를 만들 수도 있다. 어떤 객체를 만드는 factory 함수를 만든다고 하자.

```c++
template<typename T, typename... Args>
T* factory(Args&&... args)
{
    return new T(std::forward<Args>(args)...);
}
```
필자는 메모리풀을 만들면서 위의 코드(와 비슷하게 작성했다)로 인해 어떤 고민도 없이 객체 생성의 극강을 맛볼 수 있었다. 만약 variadic template과 universal reference가 없었다면 파라미터 복사와 파라미터에 대한 갯수 제한이 발생했을 것이다.

여기서 std::forward는 넘겨받은 파라미터를 다음 함수로 넘겨줄 때 어떠한 복사도 없이 바로 넘겨주는 역할을 하고 있다. 그리고 forward는 넘겨받은 참조가 rvalue이면 rvalue로 취급해주고, lvalue이면 lvalue로 취급하여 준다. 이것을 **Perfect forwarding**이라고 한다.

마지막으로 꼭 기억해야할 것이 있다면 T&&라고 해서 모두 universal reference가 되는 것은 아니다. rvalue reference가 될 수 있으니 이것을 잘 이해하고 사용해야 헷갈리는 코딩을 피할 수 있다.

2016-04-10
<br/>
END
<br/>
<br/>


To learn more about, universal reference and rvalue reference:

> * http://www.devbb.net/viewtopic.php?f=21&t=57
> * http://itguru.tistory.com/189
> * [What does T&& mean in c++?](http://stackoverflow.com/questions/5481539/what-does-t-double-ampersand-mean-in-c11)
> * [Passing int&& to f(int&&)](http://stackoverflow.com/questions/35314093/passing-int-to-fint)


<br/>
<br/>

* :pencil: Review Quiz
```c++
// question 116 from cppquiz.org
#include <iostream>
#include <utility>

int y(int &) { return 1; }
int y(int &&) { return 2; }

template <class T> int f(T &&x) { return y(x); }
template <class T> int g(T &&x) { return y(std::move(x)); }
template <class T> int h(T &&x) { return y(std::forward<T>(x)); }

int main() {
  int i = 10;
  std::cout << f(i) << f(20);
  std::cout << g(i) << g(20);
  std::cout << h(i) << h(20);
  return 0;
}
```
* What would be the correct output? To check out the correct answer, go [here.](http://cppquiz.org/quiz/question/116?result=OK&answer=112212&did_answer=Answer)




