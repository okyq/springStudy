# typeScript简介

## 1.1 ts是什么

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220426111552.png)

## 1.2 ts增加了什么

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220426112304.png)

# 一，搭建TypeScript开发环境

1. 安装nodejs
2. 使用npm全局安装ts
   + npm i -g typescript

# 二，基本类型

## 2.1 类型申明

- 类型声明是TS非常重要的一个特点；
- 通过类型声明可以指定TS中变量（参数、形参）的类型；
- 指定类型后，当为变量赋值时，TS编译器会自动检查值是否符合类型声明，符合则赋值，否则报错；
- 简而言之，类型声明给变量设置了类型，使得变量只能存储某种类型的值；

```tsx
//申明变量
let a : string;

//申明完变量直接复制
let c : boolean = false;

//如果变量的申明和赋值是同时进行的，ts可以自动对变量进行类型检测
let c = false;  //那么c的类型就是boolean

//在参数和返回值中声明类型
//在括号后面的就是返回值类型
function sum(a:number,b:number) : number{
    return a+b;
}

//这样把a的值限制为10 类似于常量
let a : 10;
//或者取或，用|连接多个类型，这样a的值是male或者famale
let a: "male"| "female"

//联合类型
let c: boolean| string;

//any表示任意类型,一个变量设置为any后，相当于对any关闭了类型检测
let d : any;
//直接声明的时候不指定类型，也是any
let d;
//any可以赋值给任意变量
let s : string;
s=d;

//unknow表示位置类型
let e :unknow 
// unknow 不能赋值给别的对象,实际上是一个类型安全的变量
//=是赋值操作，==先转换类型再比较，===先判断类型，如果不是同一类型直接为false。

```

+ 类型：

| **类型** | **例子**          | **描述**                       |
| -------- | ----------------- | ------------------------------ |
| number   | 1, -33, 2.5       | 任意数字                       |
| string   | 'hi', "hi", `hi`  | 任意字符串                     |
| boolean  | true、false       | 布尔值true或false              |
| 字面量   | 其本身            | 限制变量的值就是该字面量的值   |
| any      | *                 | 任意类型                       |
| unknown  | *                 | 类型安全的any                  |
| void     | 空值（undefined） | 没有值（或undefined）          |
| never    | 没有值            | 不能是任何值                   |
| object   | {name:'孙悟空'}   | 任意的JS对象                   |
| array    | [1,2,3]           | 任意JS数组                     |
| tuple    | [4,5]             | 元素，TS新增类型，固定长度数组 |
| enum     | enum{A, B}        | 枚举，TS中新增类型             |