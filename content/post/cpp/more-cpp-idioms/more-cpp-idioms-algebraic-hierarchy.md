---
title: "More C++ Idioms - Algebraic Hierarchy"
date: 2023-12-05T11:32:07+08:00
description: "More C++ Idioms - Algebraic Hierarchy"
featured: false
toc: false
usePageBundles: false
featureImage: ""
featureImageAlt: ''
featureImageCap: ''
thumbnail: "/images/iso_cpp_logo.png"
shareImage: "/images/iso_cpp_logo.png"
codeLineNumbers: true
figurePositionShow: true
categories:
  - C++
tags:
  - C++ Idioms
comment: true
---

## Intent

To hide multiple closely related algebraic abstractions (numbers) behind a single generic abstraction and provide a generic interface to it.

>将多个密切相关的代数抽象（数字）隐藏在单个泛型抽象之后，并为其提供泛型接口。

## Also Known As

State (Gamma et al.)

## Motivation

In pure object-oriented languages like Smalltalk, variables are run-time bindings to objects that act like labels. Binding a variable to an object is like sticking a label on it. Assignment in these languages is analogous to peeling a label off one object and putting it on another. On the other hand, in C and C++, variables are synonyms for addresses or offsets instead of being labels for objects. Assignment does not mean re-labelling, it means overwriting old contents with new one. Algebraic Hierarchy idiom uses delegated polymorphism to simulate weak variable to object binding in C++. Algebraic Hierarchy uses the Envelope Letter idiom in its implementation.

>在Smalltalk这样的纯面向对象语言中，变量是运行时绑定到类似标签的对象。将变量绑定到对象就像给它贴上标签一样。在这些语言中，赋值类似于将一个对象的标签剥离，并将其放在另一个对象上。
>
>另一方面，在C和C++中，变量是地址或偏移量的同义词，而不是对象的标签。分配并不意味着重新标记，它意味着用新的内容覆盖旧的内容。
>
>代数层次惯用法使用委托多态性来模拟C++中弱变量到对象的绑定。
>
>代数层次在它的实现中使用信封字母（Envelope Letter）的惯用法。

The motivation behind this idiom is to be able to write code like this:

```
Number n1 = Complex (1, 2);  // Label n1 for a complex number.
Number n2 = Real (10);  // Label n2 for a real number.
Number n3 = n1 + n2;  // Result of addition is labelled n3.
Number n2 = n3;  // Re-labelling.
```

## Solution and Sample Code

Complete code showing an implementation of the Algebraic Hierarchy idiom:

```c++
#include <iostream>

struct BaseConstructor
{
    BaseConstructor(int = 0) {}
};

class RealNumber;
class Complex;
class Number;

class Number
{
    friend class RealNumber;
    friend class Complex;

public:
    Number& operator=(const Number& n);
    Number(const Number& n);
    virtual ~Number();

    virtual Number operator+(const Number& n) const;
    void           swap(Number& n) throw();

    static Number makeReal(double r);
    static Number makeComplex(double rpart, double ipart);

protected:
    Number();
    Number(BaseConstructor);

private:
    void           redefine(Number* n);
    virtual Number complexAdd(const Complex& n) const;
    virtual Number realAdd(const RealNumber& n) const;

    Number* rep;
    short   referenceCount;
};

class Complex : public Number
{
    friend class RealNumber;
    friend class Number;

    Complex(double d, double e);
    Complex(const Complex& c);
    virtual ~Complex();

    virtual Number operator+(const Number& n) const;
    virtual Number realAdd(const RealNumber& n) const;
    virtual Number complexAdd(const Complex& n) const;

    double rpart, ipart;
};

class RealNumber : public Number
{
    friend class Complex;
    friend class Number;

    RealNumber(double r);
    RealNumber(const RealNumber& r);
    virtual ~RealNumber();

    virtual Number operator+(const Number& n) const;
    virtual Number realAdd(const RealNumber& n) const;
    virtual Number complexAdd(const Complex& n) const;

    double val;
};

/// Used only by the letters.
Number::Number(BaseConstructor)
    : rep(0)
    , referenceCount(1)
{
}

/// Used by static factory functions.
Number::Number()
    : rep(0)
    , referenceCount(0)
{
}

/// Used by user and static factory functions.
Number::Number(const Number& n)
    : rep(n.rep)
    , referenceCount(0)
{
    std::cout << "Constructing a Number using Number::Number" << std::endl;
    if (n.rep)
    {
        n.rep->referenceCount++;
    }
}

Number Number::makeReal(double r)
{
    Number n;
    n.redefine(new RealNumber(r));
    return n;
}

Number Number::makeComplex(double rpart, double ipart)
{
    Number n;
    n.redefine(new Complex(rpart, ipart));
    return n;
}

Number::~Number()
{
    if (rep && --rep->referenceCount == 0)
    {
        delete rep;
    }
}

Number& Number::operator=(const Number& n)
{
    std::cout << "Assigning a Number using Number::operator=" << std::endl;
    Number temp(n);
    this->swap(temp);
    return *this;
}

void Number::swap(Number& n) throw()
{
    std::swap(this->rep, n.rep);
}

Number Number::operator+(const Number& n) const
{
    return rep->operator+(n);
}

Number Number::complexAdd(const Complex& n) const
{
    return rep->complexAdd(n);
}

Number Number::realAdd(const RealNumber& n) const
{
    return rep->realAdd(n);
}

void Number::redefine(Number* n)
{
    if (rep && --rep->referenceCount == 0)
    {
        delete rep;
    }
    rep = n;
}

Complex::Complex(double d, double e)
    : Number(BaseConstructor())
    , rpart(d)
    , ipart(e)
{
    std::cout << "Constructing a Complex" << std::endl;
}

Complex::Complex(const Complex& c)
    : Number(BaseConstructor())
    , rpart(c.rpart)
    , ipart(c.ipart)
{
    std::cout << "Constructing a Complex using Complex::Complex" << std::endl;
}

Complex::~Complex()
{
    std::cout << "Inside Complex::~Complex()" << std::endl;
}

Number Complex::operator+(const Number& n) const
{
    return n.complexAdd(*this);
}

Number Complex::realAdd(const RealNumber& n) const
{
    std::cout << "Complex::realAdd" << std::endl;
    return Number::makeComplex(this->rpart + n.val, this->ipart);
}

Number Complex::complexAdd(const Complex& n) const
{
    std::cout << "Complex::complexAdd" << std::endl;
    return Number::makeComplex(this->rpart + n.rpart, this->ipart + n.ipart);
}

RealNumber::RealNumber(double r)
    : Number(BaseConstructor())
    , val(r)
{
    std::cout << "Constructing a RealNumber" << std::endl;
}

RealNumber::RealNumber(const RealNumber& r)
    : Number(BaseConstructor())
    , val(r.val)
{
    std::cout << "Constructing a RealNumber using RealNumber::RealNumber"
              << std::endl;
}

RealNumber::~RealNumber()
{
    std::cout << "Inside RealNumber::~RealNumber()" << std::endl;
}

Number RealNumber::operator+(const Number& n) const
{
    return n.realAdd(*this);
}

Number RealNumber::realAdd(const RealNumber& n) const
{
    std::cout << "RealNumber::realAdd" << std::endl;
    return Number::makeReal(this->val + n.val);
}

Number RealNumber::complexAdd(const Complex& n) const
{
    std::cout << "RealNumber::complexAdd" << std::endl;
    return Number::makeComplex(this->val + n.rpart, n.ipart);
}

namespace std {
template<>
void swap(Number& n1, Number& n2)
{
    n1.swap(n2);
}
}  // namespace std

int main(void)
{
    Number n1 = Number::makeComplex(1, 2);
    Number n2 = Number::makeReal(10);
    Number n3 = n1 + n2;

    std::cout << "Finished" << std::endl;

    return 0;
}
```

## Known Uses

## Related Idioms

Handle Body

Envelope Letter

## References

Coplien, James (1992). Advanced C++ Programming Styles and Idioms. Addison-Wesley.
