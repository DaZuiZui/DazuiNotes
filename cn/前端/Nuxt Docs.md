# Nuxt Docs 

## install 

~~~yml
 npx nuxi@latest init demo
~~~

## Create pages

创建pages文件夹，里面vue file name就是我们的路由名字，然后更改我们app.vue的路由显示

~~~java
<template>
  <div>
    <NuxtPage />
  </div>
</template>
~~~

此时我们的page下的vue名字

~~~c++
- pages 
  - Hello.vue
~~~

访问http://localhost:3000/Hello

### What is Nuxt router

Nuxt采用的是FileBased文件方式自动根据文件名创建路由规则，此外还支持路由嵌套，路由参数。

比如我们想Hello带上id参数

~~~yml
- Hello
  - [id].vue
  - index.vue
~~~

**访问链接：**http://localhost:3000/Hello/2

## use Seo

如设置文字标题

~~~vue
<script setup>
	useSeoMeta({
    title: "hello world",
    description: "Seo info"
  })
</script>
~~~

## web server api

~~~yml
- demo
  - server
    - api
    	Helloa.ts
~~~

**Hello.ts code**

~~~java
// server/api.js
export default defineEventHandler ( async(event) => {
    let res = [{name:"hello"},{name: "hi"}]
    return res;
})
~~~

访问：http://localhost:3000/api/Helloa

~~~w
<template>
    <div>
        Hello3
        <div v-for="obj in apiData">
            <span style="color:red">{{obj}}</span>
        </div>
    </div>
  </template>

<script setup>
  useSeoMeta({
    title: "hello world",
    description: "Seo info"
  })
  //请求api获取数据
  const {data: apiData} = await useFetch("http://localhost:3000/api/Helloa");
  console.log("data",apiData);
</script>
~~~

最后渲染