# 가상함수 (Virtual Function)

가상함수는 부모 클래스 포인터/참조로 함수를 호출해도  
실제 객체(자식)의 함수가 호출되도록 하는 함수입니다.

즉, **다형성을 구현하기 위해 사용하는 함수**이며,  
멤버 함수 앞에 `virtual` 키워드를 붙여 선언합니다.

<br>

## 가상함수는 왜 필요한가?

일반 멤버 함수는 **정적 바인딩(Static Binding)** 으로 동작합니다.  
즉, 컴파일 시점에 호출할 함수가 결정됩니다.

하지만 다형성을 사용할 경우에는  
부모 타입의 포인터가 실제로는 자식 객체를 가리킬 수 있습니다.

이때 실제 객체 타입에 맞는 함수를 호출하려면  
런타임에 호출 대상을 결정하는 **동적 바인딩(Dynamic Binding)** 이 필요하며,  
이를 가능하게 해주는 것이 가상함수입니다.

<br>

## 사용 예제

```cpp
#include <iostream>
using namespace std;

class Base {
public:
    virtual void print() {
        cout << "Base\n";
    }
};

class Derived : public Base {
public:
    void print() override {
        cout << "Derived\n";
    }
};

int main() {
    Base* ptr = new Derived;
    ptr->print();  // Derived
    delete ptr;
}
```

가상함수가 없다면 부모 포인터를 통해 함수를 호출할 때
부모 클래스의 함수가 호출됩니다.

하지만 가상함수를 사용하면
실제 객체 타입인 Derived의 함수가 호출됩니다.

<br>

#### 순수 가상함수 (Pure Virtual Function)

순수 가상함수는 반드시 자식 클래스에서 재정의해야 하는 함수를 의미합니다.

```
class Base {
public:
    virtual void test() = 0;
};
```

> = 0 으로 선언하며,  
> 이런 함수가 하나라도 존재하면 해당 클래스는 추상 클래스(Abstract Class) 가 됩니다.

> 순수 가상함수는 자식 클래스가 반드시 구현해야 할 인터페이스를 강제할 때 사용합니다.

<br>

## 가상함수 테이블 (vtable)

가상함수는 내부적으로 vtable(가상함수 테이블)을 통해 동작합니다.

> vtable: 가상함수 주소를 저장하는 테이블  
> vptr: 객체가 자신의 vtable을 가리키기 위해 가지는 포인터

- 각 클래스마다 하나의 vtable이 존재하고, 객체는 자신의 클래스에 해당하는 vtable을 가리키는 vptr을 가짐
- 가상함수를 가진 객체는 런타임에 vtable을 참조하여 실제 호출할 함수를 결정
- 가상함수를 사용하면 객체 내부에 vptr이 추가되어 객체 크기가 증가

<br>

**예제**

```
#include <iostream>
using namespace std;

class VirtualBase {
public:
    int a;
    virtual void print() {}
};

class Base {
public:
    int a;
    void print() {}
};

int main() {
    Base a;
    VirtualBase b;

    cout << sizeof(a) << '\n';
    cout << sizeof(b) << '\n';
}
```

> 가상함수가 있는 클래스는
> vptr 때문에 일반 클래스보다 크기가 더 커질 수 있습니다.

<br>

## 가상함수 사용 시 주의사항

#### 1. 부모 클래스 소멸자는 반드시 가상 소멸자로 선언해야 한다

부모 클래스 포인터로 자식 객체를 삭제할 수 있는 구조라면  
부모 클래스의 소멸자는 반드시 virtual 이어야 합니다.

그렇지 않으면 부모 소멸자만 호출되고  
자식 소멸자는 호출되지 않아 미정의 동작이 발생할 수 있습니다.

→ 자식에서 할당한 자원이 해제되지 않아 메모리 누수나 리소스 누수가 발생할 수 있습니다.

<br>

**잘못된 예**

```
#include <iostream>
using namespace std;

class Base {
public:
    ~Base() {
        cout << "Base" << '\n';
    }
};

class Derived : public Base {
public:
    ~Derived() {
        cout << "Derived" << '\n';
    }
};

int main() {
    Base* ptr = new Derived;
    delete ptr;
}
```

<br>

**올바른 예**

```
#include <iostream>
using namespace std;

class Base {
public:
    virtual ~Base() {
        cout << "Base" << '\n';
    }
};

class Derived : public Base {
public:
    ~Derived() override {
        cout << "Derived" << '\n';
    }
};

int main() {
    Base* ptr = new Derived;
    delete ptr;
}
```

> 이 경우 자식 소멸자 → 부모 소멸자 순서로 정상 호출

<br>

<br>

#### 2. 생성자와 소멸자 안에서 가상함수를 호출하면 안 된다

객체 생성 과정에서는  
부모 클래스 생성자가 먼저 실행되고, 그 다음 자식 클래스 생성자가 실행됩니다.

부모 생성자 실행 시점에는 아직 자식 부분이 완전히 초기화되지 않았기 때문에,  
가상함수를 호출해도 자식 버전으로 동적 바인딩되지 않습니다.

소멸자도 마찬가지로  
자식이 먼저 소멸된 뒤 부모 소멸자가 실행되므로,  
이 과정에서 가상함수를 호출하면 의도한 대로 동작하지 않을 수 있습니다.

→ 생성/소멸 과정에서는 객체의 타입이 완전하지 않기 때문에  
가상함수는 현재 클래스 기준으로만 호출됩니다.

**즉, 생성자 / 소멸자 내부에서는 가상함수 호출을 피해야 합니다.**

<br>

**예제**

```
#include <iostream>
using namespace std;

class Base {
public:
    Base() {
        print();
    }

    virtual ~Base() {}

    virtual void print() {
        cout << "Base" << '\n';
    }
};

class Derived : public Base {
public:
    int value = 10;

    Derived() {}

    void print() override {
        cout << "Derived: " << value << '\n';
    }
};

int main() {
    Derived d;
}
```

> 이 경우 Base 생성자에서 호출되는 print()는  
> Derived::print()가 아니라 Base::print()처럼 동작합니다.

<br>

<br>

#### 3. 가상함수의 기본 매개변수는 재정의하지 않는 것이 좋다

가상함수는 함수 본문은 동적 바인딩되지만,  
기본 매개변수(Default Argument)는 정적 바인딩됩니다.

→ 함수 선택은 동적 바인딩, 기본값은 정적 바인딩으로 결정됩니다.

따라서 부모와 자식에서 기본값을 다르게 정의하면  
예상과 다른 결과가 나올 수 있습니다.

<br>

**예제**

```
#include <iostream>
using namespace std;

class Base {
public:
    virtual void test(int a = 5) {
        cout << "Base " << a << '\n';
    }
};

class Derived : public Base {
public:
    void test(int a = 10) override {
        cout << "Derived " << a << '\n';
    }
};

int main() {
    Base* a = new Derived;
    Derived* b = new Derived;

    a->test();  // Derived 5
    b->test();  // Derived 10

    delete a;
    delete b;
}
```

<br>

## 📌 핵심 정리

- 가상함수는 다형성을 구현하기 위해 사용하는 함수
- 부모 포인터/참조로 자식 객체를 다룰 때 실제 객체 기준으로 함수가 호출됨
- 내부적으로 vtable / vptr을 통해 런타임에 호출 함수가 결정됨
- 순수 가상함수는 자식 클래스에 구현을 강제하며, 추상 클래스를 만듬

<br>

**주의할 점**

- 부모 클래스 소멸자는 가상 소멸자로 선언
- 생성자 / 소멸자 내부에서 가상함수 호출 금지
- 가상함수의 기본 매개변수 재정의 주의
- virtual 함수는 약간의 성능 오버헤드가 존재하므로, 필요한 경우에만 사용하는 것이 좋음
