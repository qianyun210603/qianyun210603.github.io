---
title: 【C++知识库】定义指向成员函数的函数指针
date: 2024-07-04T17:55:23+08:00
toc: true
tags: 
categories: 
- Programming
- C++
---

要在 C++ 中定义一个指向类成员函数的原始函数指针，你需要知道成员函数有一个隐式的 this 指针作为它们的第一个参数，这使它们与自由函数或静态成员函数区别开来。因此，声明指向成员函数的指针的语法包括类名和 ::* 操作符。
这是定义指向非静态成员函数的原始函数指针的一般语法：
```
ReturnType (ClassName::*pointerName)(ParameterTypes...) = &ClassName::FunctionName;
```

### 示例：
考虑一个类 MyClass，它有一个成员函数 doSomething：
```C++
class MyClass {
public:
    void doSomething(int value) {
        // 实现
    }
};
```
要定义一个指向 doSomething 成员函数的指针，你可以这样写：
```C++
void (MyClass::*funcPtr)(int) = &MyClass::doSomething;
```


### 使用成员函数指针：

要使用这个指针，你需要一个 `MyClass` 的实例，并使用 `.*` 操作符（如果你使用的是指向实例的指针，则使用 `->*`）来调用函数：
```C++
MyClass myObject;
(myObject.*funcPtr)(10); // 使用对象实例

MyClass* myObjectPtr = new MyClass();
(myObjectPtr->*funcPtr)(10); // 使用指向对象的指针
```

### 关于 Const 成员函数的说明：
```C++
void (MyClass::*funcPtrConst)(int) const = &MyClass::FunctionName;
```


### 静态成员函数：

静态成员函数没有 `this` 指针，可以被常规函数指针指向，不需要类特定的语法：
```
ReturnType (*pointerName)(ParameterTypes...) = &ClassName::StaticFunctionName;
```


## 参考资料
> - Github Copilot
