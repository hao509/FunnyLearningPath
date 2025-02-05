---
title: VueRouter简介
date: 2022-06-10 14:32:10
tags:

categories:
- 前端
---


### 标准通用模板

**src/router/index.ts**
先从vuerouter中导入三大件；  
设置对应component的路由地址；  
export整个路由模块，createRouter里面分别要写路由模式和路由信息；  

> 请避免router routes route 这些元素的混淆，他们各自代表不同的意思

```ts
import { createRouter,createWebHistory,RouteRecordRaw } from "vue-router";

const routes:Array<RouteRecordRaw> = [
    {
        path:'/',
        component:()=>import('../components/xxx.vue')
    }
]

export const router = createRouter({
    history:createWebHistory(),
    routes
})

```

<br>

**App.vue**
1. router-link配合router-view使用，故一个template里面可以有多个router-link，但只能有一个router-view
2. router-link 和面的to表示需要连接到那个路由页面，路径就写我们在path里面定义的路径
3. 每次让router-link指定一个路径后，指定路径的页面就会显示到router-view插入的地方！！！

```html
<template>
  <div>
    <h1>点击下方按钮切换对应的页面</h1>
    <router-link to="/">login</router-link>
    <router-link style="margin-left: 20px;" to="/reg">reg</router-link>
    <hr>
  </div>

    
  <router-view></router-view>
</template>
```

<br>

### 脚本式导航
导入模块useRouter后，通过使用push方法来跳转到对应的路由页面，而不是用router-link标签！！！  
下面展示了通过路由的name来跳转，当然也可以通过path来跳转

```html
<template>
  <div>
    <h1>点击下方按钮切换对应的页面</h1>
    <button @click="toPage('login')">sign in</button>
    <button @click="toPage('reg')">reg</button>
    <hr>
  </div>

  <router-view></router-view>
</template>

<script setup lang='ts'>

import { useRouter } from 'vue-router';
const router = useRouter()  // 导入路由使用模块

const toPage = (url:string)=>{
  router.push({
    name:url  // 根据路由名称打开对应页面
  })
}
</script>
```

<br>

### 历史记录
router-link标签添加replace属性，即可不记录跳转历史
```html
<router-link replace to="/">login</router-link>
```

脚本跳转模式，把push替换成replace也可以达到不记录跳转历史的效果！
```ts
const toPage = (url:string)=>{
  router.replace({
    name:url  // 根据路由名称打开对应页面
  })
}
```

<br>

使用go()或者back()来向前或者向后跳转一个网页
```ts
const toPage = (url:string)=>{
  router.go(1)      // 网页前进一步
  router.back(1)    // 网页后退一步
}
```

<br>

### 路由传参

**query传参**
上层代码为发送参数的组件，下层代码为接收参数的组件；
- 传参的方式就是在Push中写入一个属性query，之后query接受一个对象，该对象内写入你需要传递过去的属性；
- query传参法可以使用path或者name来指定路由页面
- 接收组件需要导入useRoute并实例化它来接收参数
- 接收参数格式为{ {route.query.xxx} }

```html
<!-- 发出参数的组件 -->
<script setup lang='ts'>
import { useRouter } from 'vue-router';
const router = useRouter()  // 导入路由使用模块

const toPage = (url:string)=>{
  router.push({
    path:'/reg',
    query:{         // query传入一个对象
      id:1,
      job:'none'
    }
  })
}
</script>


<!-- 接收参数的组件 -->
<template>
    <div>
        <div>{{ route.query.id }}</div>
        <div>{{ route.query.job }}</div>
    </div>
</template>

<script setup lang='ts'>
import { useRoute } from 'vue-router';
const route = useRoute()
</script>
```

<br>

**params传参**
和`query`传参类似，但是他传入的参数是放在请求文件里面的，所以不会明明白白的显示在地址栏里面；  
在push中必须且只能使用name来配合 `params` 使用，不可以使用path指定路由页面；  
接收者组件只需要按照以下格式获取即可：`{{route.params.xxx}}`

<br>

### 嵌套路由
需要定义定义路由中的route；  
添加children属性，然后可以无限嵌套下去，但要注意最终path是拼接而来的，即一层层向上链接！！
```ts
const routes:Array<RouteRecordRaw> = [
    {
        path:'/',
        component:()=>import('../components/footer.vue'),
        children:[
            {
                path:'',
                component:()=>import('../components/login.vue')
            },
            {
                path:'reg',
                component:()=>import('../components/reg.vue')
            }
        ]
    }
]
```

<br>

### 组件命名
当一个component具有多个成员时，可以对route-view添加属性name，来指定需要显示那个成员！
```ts
// 父组件定义导向的
const routes:Array<RouteRecordRaw> = [
    {
        path:'/',
        component:{
            login:import('../components/login.vue'),
            reg:import('../components/reg.vue')
        }
    }
]


// 接收者使用name来指定接收哪一个component
<router-view name="rv"></router-view>
```

<br>

### 路由别名
在routes中加入参数alias即可指定别名；  
别名的作用：当使用path调用路由页面时，也可以直接使用别名代替；  
```ts
const routes:Array<RouteRecordRaw> = [
    {
        path:'/',
        alias:['/a','/b'],
        component:{
            login:import('../components/login.vue'),
            reg:import('../components/reg.vue')
        }
    }
]
```

<br>

### 前置路由守卫
定义：路由守卫可以看做是一个筛选器，即根据访问者具有的权限来将其导航向不同的地址；  
例如某访问者想访问后台，但很明显其权限不够，路由守卫立刻阻挡并驳回请求（或者将其导向另一个网址）；  

以下是前置路由守卫的主要格式：
- 单独使用next()表示允许允许访问通过
- next('/')表示指向该访问到一个新的地址'/'
- 配合if...else...对用户的权限进行判断，来指向性导航到某个网址

```ts
const whiteList = ['/']

router.beforeEach((to, from, next) => {
  if(whiteList.includes("xxx")){
    next()
  }else{
    next('/')
  }
})
```

<br>

### 后置路由守卫
使用方式和前置路由守卫差不多，只是方法名变了一下而已，他表示在路由next()完成后执行的响应

<br>

### 路由元信息
即路由自带的一个数据包，里面可以存储变量等内容；  
使用meta定义，对象的形式！  
```ts
export const router = createRouter({
  history:createWebHistory(),
  routes:[
    {
      path:'/',
      component:()=>import('view/login.vue'),
      meta:{
        title:'这是一个标题'
      }
    }
  ]
})
```

<br>

### 
