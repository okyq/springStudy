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

```

## 2.2 字面量

```tsx
//这样把a的值限制为10 类似于常量
let a : 10;
//或者取或，用|连接多个类型，这样a的值是male或者famale
let a: "male"| "female"

//联合类型
let c: boolean| string;

```

## 2.2 any


```tsx
//any表示任意类型,一个变量设置为any后，相当于对any关闭了类型检测
let d : any;
//直接声明的时候不指定类型，也是any
let d;
//any可以赋值给任意变量
let s : string;
s=d;
```



## 2.3 unknow

```tsx
//unknow表示位置类型
let e :unknow 
// unknow 不能赋值给别的对象,实际上是一个类型安全的变量
//=是赋值操作，==先转换类型再比较，===先判断类型，如果不是同一类型直接为false。
s = e //报错
//可以使用类型断言 变量 as 类型  或者  <类型>变量
//s=<string>e or s = e as string

```



## 2.4 void

```tsx

//void 表示空，以函数为例，就表示没有返回值的函数
function fn():void{}
```



## 2.5 never

```tsx

//never 表示永远不会有结果，不会是任何值
function fn2(): never{
    throw new Error('报错了');
}

```



## 2.6 对象

```typescript
//object表示一个对象
let a : object;

//{}用来指定对象中包含哪些属性
// 语法 {属性名：属性值，属性名：属性值}
let b : {name:string , age:number}

//在属性后面加一个？ 表示属性可选
let b : {name :string, age?:number}

//[propName:string]:any 表示任意属性
let c: {name:string , [propName:string]:any}
c={name:'yuqian',a:'a',b:'b',c:'c'}

//限制函数d的结构  =>返回值
let d : (a: number,b: number)=>number;
```



## 2.7 array

```typescript
//两种方式
// 类型[]
// Array<类型>
let f : number[];
let g : Array<number>;
```



## 2.8 元组

元组就是固定长度的数组

```ts
let h : [string , string];
```



## 2.9 枚举

```tsx
enum Gender{
    male = 0,
    female = 1
}
```



## 2.10 类型限制

```tsx
let j : {name : string } & {age : number}
```



## 2.11 类型别名

```tsx
type myType = 1|2|3|4|5;
let k : myType;
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
| tuple    | [4,5]             | 元组，TS新增类型，固定长度数组 |
| enum     | enum{A, B}        | 枚举，TS中新增类型             |

# 三，编译选项

## 3.1 自动编译文件

编译文件时，使用 -w 指令后，TS编译器会自动监视文件的变化，并在文件发生变化时对文件进行重新编译。

示例：

```bush
tsc xxx.ts -w
```

## 3.2 自动编译整个项目

如果直接使用tsc指令，则可以自动将当前项目下的所有ts文件编译为js文件。

**但是能直接使用tsc命令的前提时，要先在项目根目录下创建一个ts的配置文件 tsconfig.json**

tsconfig.json是一个JSON文件，添加配置文件后，只需只需 tsc 命令即可完成对整个项目的编译

配置选项：

#### **include**

- 定义希望被编译文件所在的目录
- 默认值：["**/*"]

示例：

```json
  "include":["src/**/*", "tests/**/*"]
```

上述示例中，所有src目录和tests目录下的文件都会被编译

#### **exclude**

- 定义需要排除在外的目录
- 默认值：["node_modules", "bower_components", "jspm_packages"]

示例：

```json
  "exclude": ["./src/hello/**/*"]
```

上述示例中，src下hello目录下的文件都不会被编译

#### **extends**

- 定义被继承的配置文件

示例：

```json
"extends": "./configs/base"
```

上述示例中，当前配置文件中会自动包含config目录下base.json中的所有配置信息

#### **files**

- 指定被编译文件的列表，**只有需要编译的文件少时才会用到**

示例：

```json
"files": [
    "core.ts",
    "sys.ts",
    "types.ts",
    "scanner.ts",
    "parser.ts",
    "utilities.ts",
    "binder.ts",
    "checker.ts",
    "tsc.ts"
  ]
```

- 列表中的文件都会被TS编译器所编译

#### compilerOptions

- 编译选项是配置文件中非常重要也比较复杂的配置选项
- 在compilerOptions中包含多个子选项，用来完成对编译的配置

项目选项：

- target

  - 设置ts代码编译的目标版本

  - 可选值：

    - ES3（默认）、ES5、ES6/ES2015、ES7/ES2016、ES2017、ES2018、ES2019、ES2020、ESNext

  - 示例：

    - ```
      "compilerOptions": {
          "target": "ES6"
      }
      ```

  - 如上设置，我们所编写的ts代码将会被编译为ES6版本的js代码

- lib

  - 指定代码运行时所包含的库（宿主环境）

  - 可选值：

    - ES5、ES6/ES2015、ES7/ES2016、ES2017、ES2018、ES2019、ES2020、ESNext、DOM、WebWorker、ScriptHost ......

  - 示例：

    - ```
      "compilerOptions": {
          "target": "ES6",
          "lib": ["ES6", "DOM"],
          "outDir": "dist",
          "outFile": "dist/aa.js"
      }
      ```

- module

  - 设置编译后代码使用的模块化系统

  - 可选值：

    - CommonJS、UMD、AMD、System、ES2020、ESNext、None

  - 示例：

    - ```
      "compilerOptions": {
          "module": "CommonJS"
      }
      ```

- outDir

  - 编译后文件的所在目录

  - 默认情况下，编译后的js文件会和ts文件位于相同的目录，设置outDir后可以改变编译后文件的位置

  - 示例：

    - ```
      "compilerOptions": {
          "outDir": "dist"
      }
      ```

    - 设置后编译后的js文件将会生成到dist目录

- outFile

  - 将所有的文件编译为一个js文件

  - 默认会将所有的编写在全局作用域中的代码合并为一个js文件，如果module制定了None、System或AMD则会将模块一起合并到文件之中

  - 示例：

    - ```
      "compilerOptions": {
          "outFile": "dist/app.js"
      }
      ```

- rootDir

  - 指定代码的根目录，默认情况下编译后文件的目录结构会以最长的公共目录为根目录，通过rootDir可以手动指定根目录

  - 示例：

    - ```
      "compilerOptions": {
          "rootDir": "./src"
      }
      ```

- allowJs

  - 是否对js文件编译

- checkJs

  - 是否对js文件进行检查

  - 示例：

    - ```
      "compilerOptions": {
          "allowJs": true,
          "checkJs": true
      }
      ```

- removeComments

  - 是否删除注释
  - 默认值：false

- noEmit

  - 不对代码进行编译
  - 默认值：false

- sourceMap

  - 是否生成sourceMap
  - 默认值：false

- 严格检查

  - strict
    - 启用所有的严格检查，默认值为true，设置后相当于开启了所有的严格检查
  - alwaysStrict
    - 总是以严格模式对代码进行编译
  - noImplicitAny
    - 禁止隐式的any类型
  - noImplicitThis
    - 禁止类型不明确的this
  - strictBindCallApply
    - 严格检查bind、call和apply的参数列表
  - strictFunctionTypes
    - 严格检查函数的类型
  - strictNullChecks
    - 严格的空值检查
  - strictPropertyInitialization
    - 严格检查属性是否初始化

- 额外检查

  - noFallthroughCasesInSwitch
    - 检查switch语句包含正确的break
  - noImplicitReturns
    - 检查函数没有隐式的返回值
  - noUnusedLocals
    - 检查未使用的局部变量
  - noUnusedParameters
    - 检查未使用的参数

- 高级

  - allowUnreachableCode
    - 检查不可达代码
    - 可选值：
      - true，忽略不可达代码
      - false，不可达代码将引起错误
  - noEmitOnError
    - 有错误的情况下不进行编译
    - 默认值：false

# 四，webpack

### 4.1 webpack整合

通常情况下，实际开发中我们都需要使用构建工具对代码进行打包；

TS同样也可以结合构建工具一起使用，下边以webpack为例介绍一下如何结合构建工具使用TS；

步骤如下：

#### 初始化项目

进入项目根目录，执行命令 `npm init -y`，创建package.json文件

#### 下载构建工具

命令如下：

```
npm i -D webpack webpack-cli webpack-dev-server typescript ts-loader clean-webpack-plugin
```

共安装了7个包:

- webpack：构建工具webpack
- webpack-cli：webpack的命令行工具
- webpack-dev-server：webpack的开发服务器
- typescript：ts编译器
- ts-loader：ts加载器，用于在webpack中编译ts文件
- html-webpack-plugin：webpack中html插件，用来自动创建html文件
- clean-webpack-plugin：webpack中的清除插件，每次构建都会先清除目录

#### 配置webpack

根目录下创建webpack的配置文件`webpack.config.js`：

```tsx
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");

module.exports = {
   optimization:{
       minimize: false // 关闭代码压缩，可选
   },
//入口文件
   entry: "./src/index.ts",

   devtool: "inline-source-map",

   devServer: {
       contentBase: './dist'
   },

   output: {
       //指定打包目录
       path: path.resolve(__dirname, "dist"),
       //打包后的文件
       filename: "bundle.js",
       environment: {
           arrowFunction: false // 关闭webpack的箭头函数，可选
       }
   },

   resolve: {
       extensions: [".ts", ".js"]
   },

    //打包时需要的模块
   module: {
       //指定加载规则
       rules: [
           {
               test: /\.ts$/,
               use: {
                   loader: "ts-loader"     
               },
               exclude: /node_modules/
           }
       ]
   },

   plugins: [
       new CleanWebpackPlugin(),
       new HtmlWebpackPlugin({
           title:'TS测试'
       }),
   ]
}
```

#### 配置TS编译选项

根目录下创建tsconfig.json，配置可以根据自己需要

```
{
   "compilerOptions": {
       "target": "ES2015",
       "module": "ES2015",
       "strict": true
   }
}
```

#### 修改package.json配置

修改package.json添加如下配置

```
{
   ...
   "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1",
       "build": "webpack",
       "start": "webpack serve --open chrome.exe"
   },
   ...
}
```

#### 项目使用

在src下创建ts文件，并在并命令行执行`npm run build`对代码进行编译；

或者执行`npm start`来启动开发服务器；



### 4.2 Babel

除了webpack，开发中还经常需要结合babel来对代码进行转换；

以使其可以兼容到更多的浏览器，在上述步骤的基础上，通过以下步骤再将babel引入到项目中；

> 虽然TS在编译时也支持代码转换，但是只支持简单的代码转换；
>
> 对于例如：Promise等ES6特性，TS无法直接转换，这时还要用到babel来做转换；

安装依赖包：

```
npm i -D @babel/core @babel/preset-env babel-loader core-js
```

共安装了4个包，分别是：

- @babel/core：babel的核心工具
- @babel/preset-env：babel的预定义环境
- @babel-loader：babel在webpack中的加载器
- core-js：core-js用来使老版本的浏览器支持新版ES语法

修改webpack.config.js配置文件

```
...
module: {
    rules: [
        {
            test: /\.ts$/,
            use: [
                {
                    loader: "babel-loader",
                    options:{
                        presets: [
                            [
                                "@babel/preset-env",
                                {
                                    "targets":{
                                        "chrome": "58",
                                        "ie": "11"
                                    },
                                    "corejs":"3",
                                    "useBuiltIns": "usage"
                                }
                            ]
                        ]
                    }
                },
                {
                    loader: "ts-loader",

                }
            ],
            exclude: /node_modules/
        }
    ]
}
...
```

如此一来，使用ts编译后的文件将会再次被babel处理；

使得代码可以在大部分浏览器中直接使用；

同时可以在配置选项的targets中指定要兼容的浏览器版本；

# 五，面向对象

要想面向对象，操作对象，首先便要拥有对象；

要创建对象，必须要先定义类，所谓的类可以理解为对象的模型；

程序中可以根据类创建指定类型的对象；

举例来说：

可以通过Person类来创建人的对象，通过Dog类创建狗的对象，不同的类可以用来创建不同的对象；

### 5.1 定义类

```tsx
class 类名 {
    属性名: 类型;
    
    constructor(参数: 类型){
        this.属性名 = 参数;
    }
    
    方法名(){
        ....
    }

}
```

示例：

```tsx
    class Person{
        name: string;
        age: number;
        static gende:string = 'male'
    
        constructor(name: string, age: number){
            this.name = name;
            this.age = age;
        }
    
        sayHello(){
            console.log(`大家好，我是${this.name}`);
        }
    }
```

使用类：

```
const p = new Person('孙悟空', 18);
p.sayHello();
```

### 5.2   构造函数

可以使用`constructor`定义一个构造器方法；

> **注1：在TS中只能有一个构造器方法！**

例如：

```
class C{
    name: string;
    age: number

    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }
}
```

同时也可以直接将属性定义在构造函数中：

```
class C {
    constructor(public name: string, public age: number) {
    }
}
```

上面两种定义方法是完全相同的！

**注2：子类继承父类时，必须调用父类的构造方法（如果子类中也定义了构造方法）！**

例如：

```
class A {
    protected num: number;
    constructor(num: number) {
        this.num = num;
    }
}

class X extends A {
    protected name: string;
    constructor(num: number, name: string) {
        super(num);
        this.name = name;
    }
}
```

如果在X类中不调用`super`将会报错！

### 5.3 封装

对象实质上就是属性和方法的容器，它的主要作用就是存储属性和方法，这就是所谓的封装

默认情况下，对象的属性是可以任意的修改的，为了确保数据的安全性，在TS中可以对属性的权限进行设置

- 静态属性（static）：
  - 声明为static的属性或方法不再属于实例，而是属于类的属性；
- 只读属性（readonly）：
  - 如果在声明属性时添加一个readonly，则属性便成了只读属性无法修改
- TS中属性具有三种修饰符：
  - public（默认值），可以在类、子类和对象中修改
  - protected ，可以在类、子类中修改
  - private ，可以在类中修改

示例：

public：

```
class Person{
    public name: string; // 写或什么都不写都是public
    public age: number;

    constructor(name: string, age: number){
        this.name = name; // 可以在类中修改
        this.age = age;
    }

    sayHello(){
        console.log(`大家好，我是${this.name}`);
    }
}

class Employee extends Person{
    constructor(name: string, age: number){
        super(name, age);
        this.name = name; //子类中可以修改
    }
}

const p = new Person('孙悟空', 18);
p.name = '猪八戒';// 可以通过对象修改
```

protected：

```
class Person{
    protected name: string;
    protected age: number;

    constructor(name: string, age: number){
        this.name = name; // 可以修改
        this.age = age;
    }

    sayHello(){
        console.log(`大家好，我是${this.name}`);
    }
}

class Employee extends Person{

    constructor(name: string, age: number){
        super(name, age);
        this.name = name; //子类中可以修改
    }
}

const p = new Person('孙悟空', 18);
p.name = '猪八戒';// 不能修改
```

private：

```
class Person{
    private name: string;
    private age: number;

    constructor(name: string, age: number){
        this.name = name; // 可以修改
        this.age = age;
    }

    sayHello(){
        console.log(`大家好，我是${this.name}`);
    }
}

class Employee extends Person{

    constructor(name: string, age: number){
        super(name, age);
        this.name = name; //子类中不能修改
    }
}

const p = new Person('孙悟空', 18);
p.name = '猪八戒';// 不能修改
```

### 5.4 属性存取器

对于一些不希望被任意修改的属性，可以将其设置为private

直接将其设置为private将导致无法再通过对象修改其中的属性

我们可以在类中定义一组读取、设置属性的方法，这种对属性读取或设置的属性被称为属性的存取器

读取属性的方法叫做setter方法，设置属性的方法叫做getter方法

示例：

```
class Person{
    private _name: string;

    constructor(name: string){
        this._name = name;
    }

    get name(){
        return this._name;
    }

    set name(name: string){
        this._name = name;
    }

}

const p1 = new Person('孙悟空');
// 实际通过调用getter方法读取name属性
console.log(p1.name);
// 实际通过调用setter方法修改name属性 
p1.name = '猪八戒'; 
```

### 5.5 静态属性

静态属性（方法），也称为类属性。使用静态属性无需创建实例，通过类即可直接使用

静态属性（方法）使用static开头

示例：

```
class Tools{
    static PI = 3.1415926;
    
    static sum(num1: number, num2: number){
        return num1 + num2
    }
}

console.log(Tools.PI);
console.log(Tools.sum(123, 456));
```

### 5.6 this

在类中，使用this表示当前对象

### 5.7 继承

继承时面向对象中的又一个特性

通过继承可以将其他类中的属性和方法引入到当前类中

示例：

```
class Animal{
    name: string;
    age: number;

    constructor(name: string, age: number){
        this.name = name;
        this.age = age;
    }
}

class Dog extends Animal{

    bark(){
        console.log(`${this.name}在汪汪叫！`);
    }
}

const dog = new Dog('旺财', 4);
dog.bark();
```

通过继承可以在不修改类的情况下完成对类的扩展

### 5.8 重写

发生继承时，如果子类中的方法会替换掉父类中的同名方法，这就称为方法的重写

示例：

```
class Animal{
    name: string;
    age: number;

    constructor(name: string, age: number){
        this.name = name;
        this.age = age;
    }

    run(){
        console.log(`父类中的run方法！`);
    }
}

class Dog extends Animal{

    bark(){
        console.log(`${this.name}在汪汪叫！`);
    }

    run(){
        console.log(`子类中的run方法，会重写父类中的run方法！`);
    }
}

const dog = new Dog('旺财', 4);
dog.bark();
```

**在子类中可以使用super来完成对父类的引用**

### 5.9 抽象类（abstract class）

抽象类是专门用来被其他类所继承的类，它只能被其他类所继承不能用来创建实例

```
abstract class Animal{
  abstract run(): void;
  bark(){
      console.log('动物在叫~');
  }
}

class Dog extends Animals{
  run(){
      console.log('狗在跑~');
  }
}
```

使用abstract开头的方法叫做抽象方法，抽象方法没有方法体只能定义在抽象类中，继承抽象类时抽象方法必须要实现;

### 5.10 接口

```tsx
interface myInterface{
    name: string;
    age: number;  
}

接口可以重复定义

interface myInterface{
    name:string
}

这两个可以同时存在
```

