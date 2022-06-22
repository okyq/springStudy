# 模板语法

## 1. 插值语法和指令语法

![image-20220622172815299](https://cdn.jsdelivr.net/gh/okyq/img/image-20220622172815299.png)

+ 插值语法一般用于标签体，例如 {{}}
+ 指令语法用于标签属性 例如 `v-bind:`  可以简写为 `:`

![image-20220622173036168](https://cdn.jsdelivr.net/gh/okyq/img/image-20220622173036168.png)

## 2. 数据绑定

+ v-bind 单向绑定，只能从data 到  value
+ v-model 双向绑定，能从data 到  value也能从 value 到 data，v-model只能用于表单类元素（输入类）上，即能够用户输入产生交互的标签上
+ v-model  `v-model:value="name"`可以简写为 `v-model="name"`

## 3. el 和data 的两种写法

![image-20220622180418405](https://cdn.jsdelivr.net/gh/okyq/img/image-20220622180418405.png)

通过$mount绑定

+ data的两种写法

  ![image-20220622180525558](https://cdn.jsdelivr.net/gh/okyq/img/image-20220622180525558.png)

**由vue所管理的函数，一定不能写成箭头函数**

## 4. MVVM模型

![image-20220622181537698](https://cdn.jsdelivr.net/gh/okyq/img/image-20220622181537698.png)

![image-20220622181720608](https://cdn.jsdelivr.net/gh/okyq/img/image-20220622181720608.png)

```vue
const vm = new Vue({
	el:`#root`,
	data:{
		name:`aaa`,
		address:`bbb`,
	}
})
```

+ data中的内容最后都出现在了vm中，vm身上的所有属性以及Vue原型上的所有属性，在vue模板中都可以直接使用
+ ![image-20220622202105662](https://cdn.jsdelivr.net/gh/okyq/img/image-20220622202105662.png)

## 5. Object.defineproperty方法

`Object.defineProperty(对象，属性，{ 配置 })`

![image-20220622202919258](https://cdn.jsdelivr.net/gh/okyq/img/image-20220622202919258.png)

+ enumerable控制是否可以枚举，默认为false

+ defineproperty方法可以为对象添加属性
+ ![image-20220622203950009](https://cdn.jsdelivr.net/gh/okyq/img/image-20220622203950009.png)

## 6. 数据代理

data中的属性出现在了vm中

![image-20220622204842242](https://cdn.jsdelivr.net/gh/okyq/img/image-20220622204842242.png)



new vue对象的时候，传入的是一个配置对象options

![image-20220622210130408](https://cdn.jsdelivr.net/gh/okyq/img/image-20220622210130408.png)

vm把整个data放在了_data中，所以会出现一个三连等

![image-20220622210801490](https://cdn.jsdelivr.net/gh/okyq/img/image-20220622210801490.png)

## 7. 事件的基本使用

![image-20220622221311674](https://cdn.jsdelivr.net/gh/okyq/img/image-20220622221311674.png)

事件修饰符

![image-20220622222217889](https://cdn.jsdelivr.net/gh/okyq/img/image-20220622222217889.png)

+ 捕获阶段和冒泡阶段，先捕获再冒泡

![image-20220622222754600](https://cdn.jsdelivr.net/gh/okyq/img/image-20220622222754600.png)



































