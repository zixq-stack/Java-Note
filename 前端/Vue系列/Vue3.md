# Vue3

官网：https://cn.vuejs.org/guide/introduction.html



## 创建vue3项目

官网文档：https://cli.vuejs.org/zh/guide/creating-a-project.html#vue-create

### 使用vue-cli创建

> 注意：需要保证自己的vue-cli版本在4.5.0以上

```bash
# 查看自己@vue/cli版本
Vue -V

# 安装或升级@vue/cli
npm install -g @vue/cli

# 创建Vue3项目
vue create 项目名

# 使用ui可视化界面创建【PS：会在浏览器中打开一个管理窗口，进行手动创建和配置选项，目前不建议用，学完所有就可以使用了 】
vue ui

# 启动Vue3项目
cd 进入到创建的项目中
npm run serve
```





### 使用vite创建

官网地址；https://v3.cn.vuejs.org/guide/installation.html#vite

[vite](https://vitejs.cn/) 是另一门技术，是下一代的前端构建工具，是vue官网打造的，久一代的就是webpack。

vite的优势：

- 开发环境中，无需打包操作，可快速的冷启动。
- 真正的按需编译，无需等待整个应用编译完成再启动【PS：图中我解读的哪里，是根据路由再找模块 】。
- 轻量快速的热重载（HMR）【PS：webpack也可以热部署，只是vite更轻量和快 】。快：体现在热部署的方式不同【PS：官网中有对比图 】。

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026317-1105129922.png)



> 整理vite创建vue3的相关指令：

```bash
# 创建项目
npm init vite-app 项目名

# 进入项目目录
cd 项目名

# 安装依赖
npm install


# 运行项目
npm run dev
```







## 分析vue-cli创建的vue3项目

使用vscode打开vue-cli创建的项目。



### 查看整个目录结构

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026044-385109988.png)


![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026465-60369577.png)






### 分析入口文件main.js

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140025940-1189345421.png)

其他的东西和vue2中没什么两样



> 注意点：template中的写法和vue2相比有点变化

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140024819-1657881489.png)





## 安装vue3的开发者工具

> 注意：这里安装的测试版（data）。
>
> 另外：vue2和vue3的开发者工具最好别一起开启，很容器出问题。

直接使用google进行安装（需会“魔法”上网）即可，无法进入google应用商店的话，那么：去百"度chrome插件网"就可以了。

![image-20240215184040236](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026169-963717389.png)



## 常用Composition API / 组合式API

### 认识 setup() 函数

> setup是vue3的地基，<span style="color:red">setup是一个函数，且必须有返回值</span>，玩vue3，那么就需要提供一个平台，而这个平台就是setup。
>
> 而所谓的组合式也就是动不动就需要import引入一下才可以使用，后续会慢慢感受的。



#### 对 setup 函数快速上手

```html
<template>
  <!-- 这里面能够拿到以下的东西，全靠的是setup的return返回回来的结果 -->
  <h1>姓名: {{name}}</h1>
  <h1>性别: {{sex}}</h1>
  <h1>工种: {{job}}</h1>

  <br>
  <br>

  <button @click="helloword">调用一下vue3的setup中定义的方法</button>
</template>

<script>

export default {
  name: 'App',

  // 一、配置setup平台
  setup(){
    // 这里面的配置和vue2中的差不多，什么数据、方法、计算属性、生命周期.......只是写法有区别

    // 配置数据【PS：直接定义即可 】
    let name = '紫邪情';
    let sex = '女';
    let job = 'Java';

    // 配置方法
    function helloword(){
      // 注意点：alert()里面用的是模板字符串 - 飘字符嘛；${name}就是取上面定义的数据
      alert(`我叫: ${name},性别: ${sex}.工种: ${job}`)
    }

    // setup必须有返回值 return	就是为了把setup配置的东西交出去嘛，不然别人怎么拿到
    // 但是：return有两种写法

    // 1、对象写法
    return {
      // 返回数据
      name,sex,job,

      // 返回方法
      helloword
    }
  }
}
</script>
```

![image-20240215184523631](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026009-259230944.png)



> 补充：setup函数返回值的另一种写法：返回渲染函数写法 - 了解即可【PS：这是为了自定义渲染内容的 】。

- 第一步：引入渲染函数h，指令：`import {h} from 'vue'`。


- 第二步：使用渲染函数 并 返回

```javascript
// 2、第二种写法：返回渲染函数 - 了解即可
// 这种写法：会将下面自定义写的渲染内容 放到 前面template中去渲染
// 即：template的渲染依赖于下面自定义内容

// 第一步：需要在本组件中引入渲染函数h
// 第二步：使用渲染函数 并 返回
// return ((h) => h( 'h1', '这是setup中的返回渲染函数用户'))
// 简写
return () => h('h1','这是setup中的返回渲染函数用户')
```








#### 聊聊 setup 函数的细节问题

> 用setup和vue2的写法一起用。<span style = "color:red">最好：坚决别用</span>。


![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140025879-1909168715.png)



![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140025972-244852109.png)





原因：是因为vue3向下兼容嘛，所以：可以加入vue2的写法。

但是：说过不建议vue3和vue2混合使用。来演示一下bug。

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026073-2057070791.png)


![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140024837-1261917934.png)

注意：反过来在vue2的配置中可以获取setup函数中配置的东西，演示就跳过了。



> 另外还有一个注意点：如果在vue2和vue3的setup中配置了相同的东西，那么：优先使用setup中的。





> setup()的另外一个注意点 和 其可以接收的两个参数

setup执行时机：

- 在beforeCreate之前执行一次，同时在 `setup(){}` 中的this是undefined，即：在setup中不可以用this拿到东西 【PS：可以试着定义一个beforeCreate和setup，里面都输出一句话，看控制台谁的话在前面即可】。



setup可以接受的两个参数：

- props：值为对象，**包含：组件外部传递过来 且 组件内部声明接收了的属性** 【PS：也就是在组件内部和vue2一样配置了props配置项。有三种配置方式，vue2的基础知识，滤过了】。

注意点：若在外部传递了数据，而内部没有配置props配置项进行声明接收，那么：vue2中不会有什么错误，但是：在vue3中就会在控制台抛错。

- context：上下文对象。它里面有三个属性。
  - attrs：俗称捡漏王。值为对象，**包含：组件外部传递过来，但没有在props配置中声明的属性**，相当于vue2中的`this.$attrs`。

换言之：就是如果父组件传递了数据，但：子组件中的props配置没有声明要接收，那么：传递的数据就在子组件的attrs属性上。

  - slots：看名字就知道，就是收到的插槽内容，相当于vue2中的`this.$slots`。
  - emit：也是看名字就知道的，触发自定义事件的函数嘛，相当于vue2中的`this.$emit`。

但是：注意在vue3中，这个东西根据vue2的正常写法，好使，可是：控制台会抛警告，不想看到警告，那就在使用了这个emit的组件中，配置一个`emits`配置项即可。和props声明接收属性的配置一样，如：`emits: ['getField']`。








#### vue3的 setup 函数总结

> 所谓的setup，就是vue3中的一个全新配置项而已，值是一个函数，它是vue3中Composition API / 组合式API的地基，且这个setup函数必须有一个返回值。

组件中所用的：数据、方法、计算属性、生命周期等，均可配置在setup函数中【PS：注意写法不太一样 】。

setup函数的两种返回值写法：

1. **返回一个对象，若是这种：那么对象中的属性、方法等，在template模板中均可以直接使用【重点】**。

2. 返回一个渲染函数，这种方式是为了自定义渲染内容，玩法如下：

- 1）、引入渲染函数h，指令：`import {h} from 'vue'`。
- 2）、使用，如指令：`return () => h('h1','这是setup中的返回渲染函数用户')`。



注意点：

1. 尽量不要与vue2的配置混合使用。

- 在vue2配置（data、methods、computed... ）中可以访问到setup函数中的属性、方法等。但是：在setup函数中不能访问到vue2中配置的内容。
- 如果setup和vue2中的配置有重名的，则：优先使用setup中的配置。

2. setup不能是一个"async 函数"，因为返回值不再是retunr的对象，而是promise，这样的话：模板看不到return对象中的属性。



`setup()`的另外一个注意点 和 其可以接收的两个参数：

- setup执行时机：在beforeCreate之前执行一次，同时在 `setup(){}` 中的this是undefined。



- setup可以接受的两个参数：
  - props：值为对象，**包含：组件外部传递过来 且 组件内部声明接收了的属性**。
    - 注意点：若在外部传递了数据，而内部没有配置props配置项进行声明接收，那么：vue2中不会有什么错误，但是：在vue3中就会在控制台抛错。
  - context：上下文对象 【PS：它里面有三个属性 】。
    - attrs：俗称捡漏王。值为对象，**包含：组件外部传递过来，但没有在props配置中声明的属性**，相当于vue2中的`this.$attrs`。
      - 换言之：就是如果父组件传递了数据，但：子组件中的props配置没有声明要接收，那么：传递的数据就在子组件的attrs属性上。
    - slots：看名字就知道，就是收到的插槽内容，相当于vue2中的`this.$slots`。
    - emit：也是看名字就知道的，触发自定义事件的函数嘛，相当于vue2中的`this.$emit`。
      - 但是：注意在vue3中，这个东西根据vue2的正常写法，好使，可是：控制台会抛警告，不想看到警告，那就在使用了这个emit的组件中，配置一个`emits`配置项即可。和props声明接收属性的配置一样，如：`emits: ['getField']`。



### ref 函数：基本数据类型数据托管

> 先做一个实例：修改setup中的数据

```html
<template>
  <h1>vue3的setup函数得到的操作</h1>
  <h2>姓名: {{name}}</h2>
  <h2>性别: {{sex}}</h2>
  <button @click="changeData">修改setup中的数据</button>
</template>

<script>
export default {
  name: 'App',

  // 一、配置setup平台
  setup(){

    let name = '紫邪情';
    let sex = '女';

    function changeData(){
      name = '紫邪晴'
      sex = '男'
      console.log("修改之后的数据: ",name,sex);
    }

    // 1、对象写法
    return {
      // 返回数据
      name,sex,

      // 返回方法
      changeData
    }
  }
}
</script>
```

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140024858-537605556.png)

没实现出来，原因就是：vue不认你的修改，因此：需要借助ref函数来套娃。



#### 看看 ref 函数的真身

```html
<template>
  <h1>vue3的setup函数得到的操作</h1>
  <h2>姓名: {{name}}</h2>
  <h2>性别: {{sex}}</h2>
  <button @click="changeData">修改setup中的数据</button>
</template>

<script>

import {ref} from 'vue'

export default {
  name: 'App',

  // 一、配置setup平台
  setup(){

    // 使用ref函数来进行实现,进行套娃，把数据丢给ref函数进行管理
    let name = ref('紫邪情');
    let sex = ref('女');

    function changeData(){
      // console.log("修改之后的数据: ",name,sex);

      // 看一下ref函数的真身
      console.log(name);
    }

    // 1、对象写法
    return {
      // 返回数据
      name,sex,

      // 返回方法
      changeData
    }
  }
}
</script>
```

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140027616-1487158469.png)



既然知道了ref函数的真身，那么：想要实现数据的改变就变得轻松了。

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026360-424219032.png)

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140024820-193071497.png)



> 有个注意点

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026421-1314326332.png)



#### 使用 ref 处理对象类型

##### 看一下 ref 函数中套对象的样子是怎样的

```html
<template>
  <h2>工种: {{job.type}}</h2>
  <h2>薪资: {{job.salary}}</h2>
  <button @click="changeData">查看一下ref函数中套对象的样子</button>
</template>

<script>

import {ref} from 'vue'

export default {
  name: 'App',

  // 一、配置setup平台
  setup(){
    // 套对象在ref中
    let job = ref({
      type: 'Java',
      salary: '20k'
    });

    function changeData(){
      // 先看一下ref中套对象的样子是怎样的
      console.log(job.value);
    }

    // 1、对象写法
    return {
      // 返回数据
      job,
      // 返回方法
      changeData
    }
  }
}
</script>
```

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026281-992871199.png)





##### 修改 ref 函数中对象的属性值

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026413-772385841.png)





#### ref 函数总结一波

作用：定义一个响应式的数据【PS：即，修改数据之后可以把改后的数据渲染到页面中 】。

语法：`const xxx = ref(initValue)`。

- 创建一个包含响应式数据的引用对应。
- js中操作数据：`xxx.value`。
- 模板中读取数据：不需要.value，直接：`<div>{{xxx}}</div>`。



注意点：

- `ref()`函数中接收的数据可以是：基本类型、也可以是对象类型。
  - ref函数修饰的是基本类型时【PS：即直接用 `let name = ref('紫邪情')` 】，则：数据代理就是 `Object.dinfineProperty` 的setter和getter。
  - ref函数修饰的是对象类型时【PS：即 `let job = ref({}}` 】，则：数据代理的原理是Proxy对象【PS：这个对象其实是由Object转了一遍，即：Object ——> Proxy，而这个对象是window的ES6的全新配置 。这个对象后续会进行说明 】，对象内部 / 属性实质是借助了vue3的一个新函数 —— `reactive()函数`。





### 认识 reactive() 函数：对象或数组类型数据托管 与 深度监视

> 这个函数就是专门用来处理数据是对象 / 数组类型的。
>
> `reactive()`函数不能处理基本类型，想要处理基本类型，那么：就使用ref()函数。
>
> `ref({ })`这里面套对象的类型时，它的原理就是调用了`reactive()`函数。



#### 上手 reactive() 函数

1. 引入 `reactive()` 函数，指令：`    import {reactive} from 'vue'`。

2. 使用reactive()函数。

```html
<template>
  <h1>ref托管的数据</h1>
  <h2>{{name}}</h2>


  <h1>reactive托管的数据</h1>
  <h2>{{job.type}}</h2>
  <h2>{{job.salary}}</h2>

  <button @click="changeData">修改ref和reactive托管的数据</button>
</template>

<script>

import {ref,reactive} from 'vue'

export default {
  name: 'App',

  // 一、配置setup平台
  setup(){
    // 配置基本类型数据 - 通过ref实现
    let name = ref('紫邪情');
    // 使用reactive来管理数据
    let job = reactive({
      type: 'Java',
      salary: '20k'
    })
    // 修改基本类型数据
    function changeData(){
      // 修改ref管理的数据类型
      name.value = '紫邪晴';

      // 修改reactive托管的数据 - 相比ref，不再跟value了
      job.type = 'C';
      job.salary = '3毛';
    }

    return {
      // 返回基本类型数据 - ref托管
      name,

      // 返回reactive托管的数据
      job,

      // 返回函数
      changeData,
    }
  }
}
</script>
```





#### reactive( )怎么处理基本类型

> 了解reactive的细节问题：`reactive()`怎么处理基本类型。

前面说：`reacitve()`函数不能处理基本类型，那测试一下。

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140025036-1020697799.png)



![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026517-1175805944.png)



`reacitve()`函数托管数组类型：

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026469-1775473576.png)


![image](https://img2022.cnblogs.com/blog/2421736/202203/2421736-20220306203433121-1051507001.png)



> 想让 `reactive()` 函数也能够托管基本类型的数据，怎么办？
>
> - 把基本类型 使用 对象写法嘛，包装一下呗。

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026390-1254090404.png)


![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026545-1753487429.png)



`reactive()` 函数深度监视效果

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026733-861355665.png)



![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140025961-1735916623.png)





#### reactive() 函数总结一波

作用：定义一个对象类型的响应式数据【PS：基本类型别用它，用ref函数 】。

语法：

- 引入 `reactive()` 函数，指令：`import {reactive} from 'vue'`。
- 使用：`const 代理对象 = reactive(源对象)` 接收一个对象 / 数组，返回一个代理对象 / proxy对象。



reactive定义的响应式数据是“深层次的”。

reactive的内部是基于ES6的Proxy实现的，通过代理对象操作源对象内部数据。







### Vue3 中数据监视的原理

#### Proxy 数据监视原理

> vue2中数据监视：
>
> 1. 如果是对象类型的，那么是通过 `Object.defineProperty()` 的getter和setter来做到数据监视的；
>
> 2. 如果是数组类型那么就是通过那7个API做到数据监视，

但是这种方式有弊端，如下：

- 新增属性、删除属性，界面不会更新。
- 直接通过下标修改数组，界面不会更新。



但是：vue3中就不会出现上面的几种情况



##### Proxy 长什么样子

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140025167-1640093064.png)


![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140024985-1877479762.png)







##### 使用 Proxy 进行修改数据

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>研究Proxy监视数据原理</title>
</head>
<body>
    <script>
        // 定义一个对象
        let person = {
            name: '紫邪情',
            sex: '女'
        }

        // 利用Window.Proxy()来进行修改person

        // 先看一下Proxy的样子
        // console.log( new Proxy(person,{} ) );

        // 使用Proxy进行数据修改
        /* 
            people			就是代理对象 它代理的就是person
            new Proxy()		就是创建一个代理对象嘛 - 后端的人太熟悉不过了
        */
        const people = new Proxy( person, {
            // 获取对象的属性时调用
            /* 
                target 就是源对象 即：person
                propName  就是对象中的属性名  如：name、sex.....
            */
            get(target,propName){
                console.log( "target,propName这两个参数为: ", target,propName);
                console.log(`有人获取person中的${propName}属性`);
                return target[propName];
            },

            // 修改对象中的属性时调用【PS：修改含增、改、删除是另一个配置 】
            // value就是修改之后的值
            set( target,propName,value ){
                console.log( "target,propName,value这三个参数为: ", target,propName,value);
                console.log( `有人修改person中的${propName}属性` );
                return target[propName] = value;
            },

            // 删除对象中的属性时调用
            deleteProperty(target,propName){
                console.log( "target,propName这两个参数为: ", target,propName);
                return delete target[propName];
            }
        })
    </script>
</body>
</html>
```

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140024857-772263597.png)





#### Reflect 数据监视原理

> 在vue3中数据监视不止用了window的Proxy对象，还用了window的Reflect对象。



##### 认识 Reflect 对象

看看Reflect长什么样。

```javascript
// 先看看Reflect长什么样
console.log(window.Reflect);
```

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026404-2080206543.png)



经过上图的查看之后，其实也就知道Reflect该怎么玩了，调对应的API就可以了【PS：ECMA组织正打算把常用的一些API放到Reflect对象身上，如：目前把 `Object.defineProperty()` 就放在Reflect中了】。



##### 使用 Reflect 实现数据监视

```javascript
let person = {
    name: '紫邪情',
    sex: '女'
}

// 先看看Reflect长什么样
// console.log(window.Reflect);

// 使用Reflect实现数据监视

// 1、获取对象的属性 - key-value的形式
/* 
    key 就是对象名
    value 就是对象的属性名
*/
Reflect.get( person,'name' );


// 2、修改对象的属性 
Reflect.set( person,'sex','男');
Reflect.set( person,'age', '18');

// 3、删除对象的属性
Reflect.deleteProperty( person,'sex');
```

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140024964-90690407.png)



> 注意：使用Reflect做对应的操作之后是有返回值的，如：`Reflect.set(person,'age',18)`，返回值是true，所以：就可以利用这个返回值做很多事情，如：进行封装，而 `Object.defineProperty()` 并没有返回值。
>
> 同时：Reflect支持属性名重复，即：若用 `set()` 这个API对同一个对象的同一个属性做多次相同的操作，则：不会返回异常，而是返回true / false，因此：才说可以用这个返回值做很多事情；若用 `Object.defineProperty()` 来进行相同的操作，则：会直接抛异常，甚至想要后续的代码还能运行，就只能使用try......catch....来对该部分的代码进行包裹了





#### Vue3真正做到数据监视的原理：Proxy和Reflect对象套娃

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>vue3实现数据监视的真正原理</title>
</head>
<body>
    <script>
        let person = {
            name: '紫邪情',
            sex: '女'
        }

        const people = new Proxy( person, {
            // 获取对象的属性时调用
            get(target,propName){
                console.log( "target,propName这两个参数为: ", target,propName);
                console.log(`有人获取person中的${propName}属性`);
                // 此处进行了Reflect套娃
                // return target[propName];
                return Reflect.get(target,propName);
            },

            // 修改对象中的属性时调用【PS：修改含增、改、删除是另一个配置 】
            set( target,propName,value ){
                console.log( "target,propName,value这三个参数为: ", target,propName,value);
                console.log( `有人修改person中的${propName}属性` );
                // 此处进行了Reflect套娃
                // return target[propName] = value;
                return Reflect.set(target,propName,value);
            },

            // 删除对象中的属性时调用
            deleteProperty(target,propName){
                console.log( "target,propName这两个参数为: ", target,propName);
                // 此处进行了Reflect套娃
                // return delete target[propName];
                return Reflect.defineProperty(target,propName);
            }
        })
    </script>
</body>
</html>
```





#### Vue3 中数据监视原理总结

通过Proxy代理对象：拦截对象中任意属性的变化，包括：属性值的读写、属性的添加、属性的删除等

通过Reflect反射对象：对被代理对象的属性进行操作，例子如下：

```javascript
const people = new Proxy( person, {
    // 拦截读取属性值
    get(target,propName){
        console.log( "target,propName这两个参数为: ", target,propName);
        console.log(`有人获取person中的${propName}属性`);
        // 此处进行了Reflect套娃
        // return target[propName];
        return Reflect.get(target,propName);
    },

    // 拦截修改属性值【PS：是修改和新增 】
    set( target,propName,value ){
        console.log( "target,propName,value这三个参数为: ", target,propName,value);
        console.log( `有人修改person中的${propName}属性` );
        // 此处进行了Reflect套娃
        // return target[propName] = value;
        return Reflect.set(target,propName,value);
    },

    // 拦截删除属性值
    deleteProperty(target,propName){
        console.log( "target,propName这两个参数为: ", target,propName);
        // 此处进行了Reflect套娃
        // return delete target[propName];
        return Reflect.defineProperty(target,propName);
    }
})

```



另外：附上Proxy和Reflec对象说明的官网链接

- Proxy：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy

- Reflect：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect











### reactive() 和 ref() 的对比

从定义数据角度对比：

- ref 用来定义：基本类型数据。
- reactive 用来定义：对象 / 数组类型数据。
- 注：ref 也可以用来定义对象 / 数组类型数据，它内部会自动通过reactive转成Proxy代理对象。



从原理角度对比：

- ref 通过 `Object.defineProperty()` 的get和set来实现的数据劫持 / 数据监视 / 响应式。
- reactive 通过使用Proxy代理对象来实现数据劫持，并通过Reflect操作源对象内部的数据。



从使用角度对比：

- ref 定义的数据：操作数据需要`.value`，读取数据时模板中直接读取，不需要`.value`。
- reactive 定义的数据：操作数据与读取数据，均不需要`.value`。





### Computed() 计算属性函数

其实和Vue2中的计算属性没什么两样，只是多了一步引入的问题 以及 排放的问题而已。

```html
<template>
  姓；<input type="text" v-model="person.firstName">

  <br>

  名: <input type="text" v-model="person.lastName">

  <br>

  <span>全名: {{person.fullName}}</span>
</template>

<script>
/*	顺便说一下
	@vue/runtime-core		这里的@表示特定路径的名称	可在 build/webpack.base.conf.js 中配置，如：
	
		resolve: {
			extensions: [',js', '.vue', '.json'],
			alias: {
				'vue$': 'vue/dist/vue.esm.js',
				'@': resolve('src'),
			}
		}
	
	在vue中导包最好都用这种写法
*/
import { reactive } from '@vue/reactivity'

// 1、引入computed计算属性函数
import { computed } from '@vue/runtime-core'

export default {
  name: 'App',

  setup(){

    // 数据
    let person = reactive({
      firstName: '紫',
      lastName: '邪情'
    })

    // 2、使用计算属性函数 setup中this无效，所以computed()中使用兰姆达和正常写法都无所谓
    // 简写形式 - 只考虑读的问题
    person.fullName = computed( ()=>{
      return person.firstName + "-" + person.lastName;
    })

    // 完整写法 - 考虑读和改的问题
   /*  person.fullName = computed({
      get(){
        return person.firstName + "-" + person.lastName;
      },

      set(value){
        const nameDataArr = value.split('-')
        person.firstName = nameDataArr[0]
        person.lastName = nameDataArr[1]
      }
    }) */

    // 返回数据
    return {
      person,
    }
  }
}
</script>
```





### watch() 监视属性函数

和vue2中的watch也差不多。



#### 监视 ref() 定义的数据



##### 简单写法：监视ref托管的单个响应式数据

```html
<template>
  <h1>当前值为: {{num}}</h1>
  <button @click="num ++ ">num++</button>
</template>

<script>

// 1、引入watch函数
import { ref, watch } from '@vue/runtime-core'


export default {
  name: 'App',

  setup(){

    // 准备数据		用ref托管
    let num = ref(0)


    // 一、简单写法
    // 2、使用watch函数
    /* 
      可以接受三个参数
          第一个：监视的是谁？
          第二个：回调函数 - 新值 和 旧值
          第三个：配置项 - deep深度监视也可以配置
    */
    watch(num , (newValue,oldValue)=>{
      console.log("num的值发生改变了",newValue,oldValue);
    },{immediate:true})


    return {
      num,
    }
  }
}
</script>
```





##### 数组写法（监视多个属性）：监视ref托管的多个响应式数据

```html
<template>
  <span>当前名字为: {{name}}</span>
  <br>
  <button @click="name += '!'">改变name</button>
</template>

<script>

// 1、引入watch函数
import { ref, watch } from '@vue/runtime-core'


export default {
  name: 'App',

  setup(){

    // 准备数据 - 用ref托管
    let num = ref(0)

    let name = ref('紫邪情')

    // 监视ref托管的多个响应式数据 - 变化就在这里 监事的是谁？采用数组写法即可
    watch([num,name],(newValue,oldValue)=>{
      console.log("num 和 name的值发生改变了",newValue,oldValue);
    },{immediate:true})
    return {
      num,name
    }
  }
}
</script>
```





#### 监视 reactive() 定义的数据

##### 监视 reactive 托管的一个响应式数据中的全部属性

```html
<template>
  姓名: <input type="text" v-model="person.name"> <br>
  性别: <input type="text" v-model="person.sex"> <br>
  地址: <input type="text" v-model="person.address.detailed.value">

  <span>姓名: {{person.name}}</span> <br>
  <span>性别: {{person.sex}}</span> <br>
  <span>地址: {{person.address.detailed.value}}</span>
</template>

<script>
import { reactive } from '@vue/reactivity'
import { watch } from '@vue/runtime-core'

export default {
  name: 'App',

  setup(){

    // 准备数据		用reactive托管
    let person = reactive({
      name: '紫邪情',
      sex: '女',
      address: {
        detailed: {
          value: '浙江省杭州市'
        }
      }
    })

    // 监视reactive托管的一个响应式数据中的全部属性
    watch(person,(newValue,oldValue)=>{
      console.log("person被修改了", newValue,oldValue);
    })

    return {
      person,
    }
  }
}
</script>
```

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140024979-444371739.png)



上面这种坑就是在监视此种reactive托管的一个响应式数据的全部属性时，并不能获得旧值oldValue，因为：旧值oldValue和新值newValue一样。



> 还有一种坑，就是：此种类型是强制开启了深度监视，即：配置`deep:false`不顶用

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140025386-1538670457.png)



![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140025728-32251890.png)



##### 监视reactive托管的一个响应式数据中的某一个属性

```javascript
// 类型二、监视reactive托管的一个响应式数据中的某一个属性
/* 
  奇葩的地方：
      1、要监视的这个属性需要写成函数式 ()=> person.name
      2、可以争取获取newValue、oldValue
*/
watch(()=> person.name , (newValue,oldValue)=>{
  console.log("person中的name属性被修改了",newValue,oldValue);
})
```

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140024840-1679395242.png)





\

##### 函数式数组写法：监视reactive托管的一个响应式数据中的某些属性

```javascript
// 类型三、监视reactive托管的一个响应式数据中的某些属性
/* 
  奇葩的地方：
      1、监视的多个属性需要使用数组套起来
      2、数组中的每一个属性需要写成函数式
*/
watch([()=> person.name , ()=> person.sex] , (newValue,oldValue)=>{
  console.log("person中的name和sex属性被修改了",newValue,oldValue);
})
```





##### 特殊情况：监视reactive托管的一个响应式数据中的某一个属性

> 此属性套娃了，又是一个对象 。

```javascript
// 类型四、监视reactive托管的一个响应式数据中的某个属性，但：此属性又套娃了
/* 
  奇葩的地方：
      1、需要开启深度监视 即：deep:true 又生效了
      2、不加 deep:true 配置，代码会无效
*/
watch(()=> person.address , (newValue,oldValue)=>{
  console.log("person中的address属性被修改了",newValue,oldValue);
},{deep:true})
```

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026182-492050419.png)





但是：如果不加`deep:true`配置呢？

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140025052-2008486894.png)









##### 监视 reactive 托管的一个响应式数据的各种类型总结

> 注：在vue3中可以同时配置多个watch，而在vue2中配置重复的，那只有前者有效。

```javascript
// 准备数据 - 用reactive托管
let person = reactive({
  name: '紫邪情',
  sex: '女',
  address: {
    detailed: {
      value: '浙江省杭州市'
    }
  }
})


// 类型一、监视reactive托管的一个响应式数据中的全部属性
/*
  此种类型的坑：
      1、无法正确获得oldValue的值【PS：因newValue和oldValue的值一样】
      2、简直强制开启了深度监视 【PS：即deep:false配置无效】
*/
watch(person,(newValue,oldValue)=>{
  console.log("person被修改了", newValue,oldValue);
},{deep:false})
/*
  如：这里关闭深度监视 理论上：应该监视不到address.detailed.value
      但是：天真
*/



// 类型二、监视reactive托管的一个响应式数据中的某一个属性
/*
  奇葩的地方：
      1、要监视的这个属性需要写成函数式 ()=> person.name
      2、可以争取获取newValue、oldValue
*/
watch(()=> person.name , (newValue,oldValue)=>{
  console.log("person中的name属性被修改了",newValue,oldValue);
})



// 类型三、监视reactive托管的一个响应式数据中的某些属性
/*
  奇葩的地方：
      1、监视的多个属性需要使用数组套起来
      2、数组中的每一个属性需要写成函数式
*/
watch([()=> person.name , ()=> person.sex] , (newValue,oldValue)=>{
  console.log("person中的name和sex属性被修改了",newValue,oldValue);
})



// 类型四、监视reactive托管的一个响应式数据中的某个属性，但：此属性又套娃了
/*
  奇葩的地方：
      1、需要开启深度监视 即：deep:true 又生效了
      2、不加 deep:true配置，代码会无效
*/
watch(()=> person.address , (newValue,oldValue)=>{
  console.log("person中的address属性被修改了",newValue,oldValue);
},{deep:true})

return {
  person,
}
```







### watchEffect() 智能监视函数

> 注：此种监视对应ref托管 和 reactive托管都可以监视到。

```html
<template>
  姓名: <input type="text" v-model="person.name"> <br>
  性别: <input type="text" v-model="person.sex"> <br>
  地址: <input type="text" v-model="person.address.detailed.value">

  <span>姓名: {{person.name}}</span> <br>
  <span>性别: {{person.sex}}</span> <br>
  <span>地址: {{person.address.detailed.value}}</span>
</template>

<script>
import { reactive } from '@vue/reactivity'

// 1、引入watchEffect函数
import { watchEffect } from '@vue/runtime-core'

export default {
  name: 'App',

  setup(){

    let person = reactive({
      name: '紫邪情',
      sex: '女',
      address: {
        detailed: {
          value: '浙江省杭州市'
        }
      }
    })

    // 2、使用watchEffect函数对响应式数据进行智能监视
    /* 
      1、不需要指名要监视谁
      2、不需要newValue 和 oldValue【PS：因为都不知道要监视谁 】
    */
    watchEffect(()=>{
      // 所谓智能：就体现在这里面的函数体中
      //          要监视谁，取决于这个函数体里面用到了谁，那就监视谁

      // 如：要监视person中的name，那就直接写改写的代码即可，此函数会自动判定，从而监视
      const personName = person.name

      // 如：要监视person中的sex，那就用它就可以了
      const personSex = person.sex

      console.log("watchEffect智能监视函数被调用了");

      // 而此函数体中没有用到的，那么：就不会去监视它
    })

    return {
      person,
    }
  }
}
</script>
```





### Vue3中的 watch 函数 和 watchEffect 函数的对比

watch 函数的套路是：既要指明要监视哪个属性，也有指明监视的回调。

watchEffect 函数的套路是：不用指明要监视哪个属性，回调中用到了哪个属性，那就监视哪个属性。

watchEffect 函数 和 Computed 函数有点像：

- Computed 函数注重：计算出来的值 【PS：回调函数的返回值 】 ，所以必须写返回值。
- watchEffect 函数注重：过程 【PS：回调函数的函数体 】，所以不用写返回值。





### Vue3中的生命周期图

和vue2的生命周期差不多，只是需要注意一些点而已

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026566-1700868154.png)





#### Vue3中生命周期的注意项

> 对比vue2中的生命周期，vue3中改动的地方，如下所示：

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140025434-1865858250.png)











![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140024922-645088242.png)





> vue3中生命周期的写法问题

1. 配置项写法：和name、`setup()`保持平级，写法就是：按照官网中说的哪些名字直接写即可。

```javascript
<script>

    export default {
      name: 'App',

      setup() {},

      // vue3中的生命周期 - 配置项写法 【PS：和name、setup保持平级即可 】
      beforeCreate(){ console.log("------beforeCreate-----"); },
      created(){ console.log("------created-----"); },
          
      beforeMount(){ console.log("------beforeMount-----"); },
      mounted(){ console.log("------mounted-----"); },
          
      beforeUpdate(){ console.log("------beforeUpdate-----"); },
      updated(){ console.log("------updated-----"); },
          
      beforeUnmount(){ console.log("------beforeUnmount-----"); },
      unmounted(){ console.log("------unmounted-----"); },
    }
</script>
```



2. 组合式API写法 ：万事引入对应函数嘛。不过此种方式名字有点区别。

- `beforeCreate` ====> `setup()`
- `created` ====> `setup()`
- `beforeMount` ====> `onBeforeMount`
- `mounted` ====> `onMounted`
- `beforeUpdate` ====> `onBeforeUpdate`
- `updated` ====> `onUpdated`
- `beforeUnMount` ====> `onBeforeUnMount`
- `UnMounted` ====> `onUnMount`



```javascript
<script>
// 1、引入对应的钩子函数
import { onBeforeMount, onMounted } from '@vue/runtime-core'

export default {
  name: 'App',

  setup() {
    
    // 另一种写法 - 组合式API写法 - 万事引入对应的函数嘛
    /* 
      只是注意：setup()就相当于beforeCreate() 和 created()
    */

    // 2、使用对应的钩子函数
    onBeforeMount(()=>{
      console.log("------beforeMount-----");
    })

    onMounted(()=>{
      console.log("------onMounted-----");
    })

    // 其他的都是一样的，就不写了，注意名字即可
  },
</script>
```



> 需要注意一个点：
>
> - 配置项写法 和 组合式API写法同时存在同一个钩子函数时。则：setup()中所用的组合式API写法比配置项写法优先执行。



### Vue3中的函数封装思想

这个玩意儿就相当于是vue2中的mixin混入，也是为了抽离代码而已。

1, 新建一个封装函数的js文件。

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026520-982557757.png)



2. 在需要的地方引入并使用。

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140025852-715592738.png)









### Vue3中的 toRef 和 toRefs 数据拆分函数

> 这两个东西就是为了解决在模板中渲染时稍微方便点而已，因为在模板中使用插值表达式 xxx.xxxx.xxx 这种取值并不合理，插值表达式的宗旨就是简单取值嘛，所以通过 xxx.xxxx.xxx 的方式并不好。

当然：toRef和toRefs也不一定能够完全解决插值表达式的问题【PS：主要看自己设计 】。



#### toRef() 函数

> 使用 toRef() 函数交出单个数据

```html
<template>
  <h2>姓名: {{person.name}}</h2>
  <h2>性别: {{person.sex}}</h2>
  <h2>地址: {{person.address.value}}</h2>
  <!-- 上面这种方式并不好，简化 -->
  <br>
  <br>
  <h1>使用toRef和toRefs函数进行简化</h1> <br>
  <!-- 下面就可以直接简写了 -->
  <h2>姓名: {{name}}</h2>
  <h2>性别: {{sex}}</h2>
  <h2>地址: {{address}}</h2>
</template>

<script>
// 1、组合式还是逃不开引入的问题
import { reactive, toRef } from '@vue/reactivity'


export default {
  name: 'App',

  setup() {
    let person = reactive({
      name: '紫邪情',
      sex: '女',
      address: {
        value: '浙江杭州'
      }
    })

    return {
      person,
      // 2、使用toRef()函数
      // 使用toRef函数交出单个数据
      /* 
        第一个参数： 交出的数据是哪个对象中的
        第二个参数： 要交出的是对象中的哪个属性
      */
      name: toRef(person,'name'),
      sex: toRef(person,'sex'),
      // 这里需要注意一下：要交出的对象里面又套娃了，那么：第一个参数需要再进一步
      address: toRef(person.address,'value'),
    }
  },
}
</script>
```



#### toRefs() 函数

> 使用toRefs()函数

```html
<template>
  <h2>姓名: {{person.name}}</h2>
  <h2>性别: {{person.sex}}</h2>
  <h2>地址: {{person.address.value}}</h2>
  <!-- 上面这种方式并不好，简化 -->
  <br>
  <br>
  <h1>使用toRefs函数进行简化</h1> <br>
  <!-- 下面就可以直接简写了 -->
  <h2>姓名: {{name}}</h2>
  <h2>性别: {{sex}}</h2>
  <!-- 但是：美中不足就是，这里是里面套的娃，所以还得需要xxx.xxx一下 -->
  <h2>地址: {{address.value}}</h2>
</template>

<script>
// 1、组合式还是逃不开引入的问题
import { reactive, toRefs } from '@vue/reactivity'


export default {
  name: 'App',

  setup() {
    let person = reactive({
      name: '紫邪情',
      sex: '女',
      address: {
        value: '浙江杭州'
      }
    })

    return {
      person,
      // 利用toRef()交出数据，需要写多次toRef，所以还是不喜欢，那就用toRefs()函数
      /* 
        直接说要交出哪个对象即可
        注意点：return{}是一个对象，所以：使用toRefs就是对象中套对象，因此注意写法
      */
      ...toRefs(person)
    }
  },
}
</script>
```





#### 为何 toRef() 和 toRefs() 函数可做到数据简化？

现在回过来看一下，为什么通过 `toRef() ` 和 `toRefs()` 函数可以做到数据简化？

使用 `toRef()` 举例，去看一下它长什么样子？toRefs()函数是一样的原理。

```javascript
console.log( toRef(person,'name') );
```

![image](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240624140026506-1760429252.png)







#### Vue3的 toRef() 和 toRefs() 函数总结

作用：创建一个RefImpl引用对象，而此对象的value属性是指向你要交出数据的那个对象的某个属性 。

- 解读：创建的这个RefImpl引用对象就相当于中间商，这个中间商代表的就是你要交出去的那个数据 ，注意：是代表啊，意思就相当于是此中间商的引用指向的地址 和 原本你要交出去的对象的某个属性指向的是同一个地方，因此：**此中间商RefImpl可以修改原数据，即：你把数据交出去之后，被修改了，那么原数据中的数据也会被修改，数据保持同步嘛**。



语法：`const name = toRef(person,'name')`  **记得先引入对应的函数**。

应用场景：要将响应式对象中的某个属性单独提供给外部使用时就可以用这两个函数

扩展：`toRefs`与`toRef`功能一致，但：可以批量创建多个RefImpl引用对象

- toRefs的语法：`toRefs(person)`







## 其它 Composition API

### shallowReactive 与 shallowRef

shallowReactive：只处理对象最外层属性的响应式（浅响应式）。

shallowRef：只处理基本数据类型的响应式, 不进行对象的响应式处理。

什么时候使用?

-  如果有一个对象数据，结构比较深, 但变化时只是外层属性变化 ===> shallowReactive。
-  如果有一个对象数据，后续功能不会修改该对象中的属性，而是生新的对象来替换 ===> shallowRef。



### readonly 与 shallowReadonly

readonly: 让一个响应式数据变为只读的（深只读）。

shallowReadonly：让一个响应式数据变为只读的（浅只读）。

应用场景: 不希望数据被修改时。



### toRaw 与 markRaw

toRaw：

- 作用：将一个由```reactive```生成的<strong style="color:red">响应式对象</strong>转为<strong style="color:red">普通对象</strong>。
- 使用场景：用于读取响应式对象对应的普通对象，对这个普通对象的所有操作，不会引起页面更新。



markRaw：

- 作用：标记一个对象，使其永远不会再成为响应式对象。
- 应用场景:
  1. 有些值不应被设置为响应式的，例如复杂的第三方类库等。
  2. 当渲染具有不可变数据源的大列表时，跳过响应式转换可以提高性能。



### customRef

作用：创建一个自定义的 ref，并对其依赖项跟踪和更新触发进行显式控制。

实现防抖效果：

```html
<template>
	<input type="text" v-model="keyword">
	<h3>{{keyword}}</h3>
</template>

<script>
	import {ref,customRef} from 'vue'
	export default {
		name:'Demo',
		setup(){
			// let keyword = ref('hello') //使用Vue准备好的内置ref
			// 自定义一个myRef
			function myRef(value,delay){
				let timer
				// 通过customRef去实现自定义
				return customRef((track,trigger)=>{
					return{
						get(){
							track() // 告诉Vue这个value值是需要被“追踪”的
							return value
						},
						set(newValue){
							clearTimeout(timer)
							timer = setTimeout(()=>{
								value = newValue
								trigger() // 告诉Vue去更新界面
							},delay)
						}
					}
				})
			}
			let keyword = myRef('hello',500) // 使用程序员自定义的ref
			return {
				keyword
			}
		}
	}
</script>
```



### provide 与 inject

作用：实现<strong style="color:red">祖与后代组件间</strong>通信。

套路：父组件有一个 `provide` 选项来提供数据，后代组件有一个 `inject` 选项来开始使用这些数据。

具体写法：

1. 祖组件中：

   ```js
   setup(){
   	......
       let car = reactive({name:'奔驰',price:'40万'})
       provide('car',car)
       ......
   }
   ```

2. 后代组件中：

   ```js
   setup(props,context){
   	......
       const car = inject('car')
       return {car}
   	......
   }
   ```





## 响应式数据的判断

- isRef:：检查一个值是否为一个 ref 对象。
- isReactive:：检查一个对象是否是由 `reactive` 创建的响应式代理。
- isReadonly： 检查一个对象是否是由 `readonly` 创建的只读代理。
- isProxy:：检查一个对象是否是由 `reactive` 或者 `readonly` 方法创建的代理。





## Composition API 的优势

### Options API 存在的问题

使用传统OptionsAPI中，新增或者修改一个需求，就需要分别在data，methods，computed里修改 。

<div style="width:600px;height:370px;overflow:hidden;float:left">
    <img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f84e4e2c02424d9a99862ade0a2e4114~tplv-k3u1fbpfcp-watermark.image" style="width:600px;float:left" />
</div>
<div style="width:300px;height:370px;overflow:hidden;float:left">
    <img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5ac7e20d1784887a826f6360768a368~tplv-k3u1fbpfcp-watermark.image" style="zoom:50%;width:560px;left" /> 
</div>




















### Composition API 的优势

我们可以更加优雅的组织我们的代码，函数。让相关功能的代码更加有序的组织在一起。

<div style="width:500px;height:340px;overflow:hidden;float:left">
    <img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc0be8211fc54b6c941c036791ba4efe~tplv-k3u1fbpfcp-watermark.image"style="height:360px"/>
</div>
<div style="width:430px;height:340px;overflow:hidden;float:left">
    <img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6cc55165c0e34069a75fe36f8712eb80~tplv-k3u1fbpfcp-watermark.image"style="height:360px"/>
</div>




















## 新的组件

### Fragment

- 在Vue2中:：组件必须有一个根标签。
- 在Vue3中:：组件可以没有根标签, 内部会将多个标签包含在一个Fragment虚拟元素中。
- 好处:：减少标签层级, 减小内存占用。



### Teleport

- 什么是Teleport？—— `Teleport` 是一种能够将我们的<strong style="color:#DD5145">组件html结构</strong>移动到指定位置的技术。

  ```html
  <teleport to="移动位置">
  	<div v-if="isShow" class="mask">
  		<div class="dialog">
  			<h3>我是一个弹窗</h3>
  			<button @click="isShow = false">关闭弹窗</button>
  		</div>
  	</div>
  </teleport>
  ```



### Suspense

- 等待异步组件时渲染一些额外内容，让应用有更好的用户体验

- 使用步骤：

  - 异步引入组件

    ```js
    import {defineAsyncComponent} from 'vue'
    const Child = defineAsyncComponent(()=>import('./components/Child.vue'))
    ```

  - 使用```Suspense```包裹组件，并配置好```default``` 与 ```fallback```

    ```html
    <template>
    	<div class="app">
    		<h3>我是App组件</h3>
    		<Suspense>
    			<template v-slot:default>
    				<Child/>
    			</template>
    			<template v-slot:fallback>
    				<h3>加载中.....</h3>
    			</template>
    		</Suspense>
    	</div>
    </template>
    ```





## 其他

### 全局API的转移

Vue 2.x 有许多全局 API 和配置。

例如：注册全局组件、注册全局指令等。

```js
//注册全局组件
Vue.component('MyButton', {
  data: () => ({
    count: 0
  }),
  template: '<button @click="count++">Clicked {{ count }} times.</button>'
})

//注册全局指令
Vue.directive('focus', {
  inserted: el => el.focus()
}
```



Vue3.0中对这些API做出了调整：将全局的API，即：```Vue.xxx```调整到应用实例（```app```）上

| 2.x 全局 API（```Vue```） | 3.x 实例 API (`app`)                        |
| ------------------------- | ------------------------------------------- |
| Vue.config.xxxx           | app.config.xxxx                             |
| Vue.config.productionTip  | <strong style="color:#DD5145">移除</strong> |
| Vue.component             | app.component                               |
| Vue.directive             | app.directive                               |
| Vue.mixin                 | app.mixin                                   |
| Vue.use                   | app.use                                     |
| Vue.prototype             | app.config.globalProperties                 |





### 其他改变

data选项应始终被声明为一个函数。

过度类名的更改：

- Vue2.x写法

  ```css
  .v-enter,
  .v-leave-to {
    opacity: 0;
  }
  .v-leave,
  .v-enter-to {
    opacity: 1;
  }
  ```

- Vue3.x写法

  ```css
  .v-enter-from,
  .v-leave-to {
    opacity: 0;
  }
  
  .v-leave-from,
  .v-enter-to {
    opacity: 1;
  }
  ```



<strong style="color:red">移除</strong>keyCode作为 v-on 的修饰符，同时也不再支持```config.keyCodes```。

- <strong style="color:#DD5145">移除</strong>```v-on.native```修饰符

  - 父组件中绑定事件

    ```html
    <my-component
      v-on:close="handleComponentEvent"
      v-on:click="handleNativeClickEvent"
    />
    ```

  - 子组件中声明自定义事件

    ```javascript
    <script>
      export default {
        emits: ['close']
      }
    </script>
    ```

  

  <strong style="color:red">移除</strong>过滤器（filter）：过滤器虽然这看起来很方便，但它需要一个自定义语法，打破大括号内表达式是 “只是 JavaScript” 的假设，这不仅有学习成本，而且有实现成本！建议用方法调用或计算属性去替换过滤器。