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

## connect mysql

参考文档：https://blog.51cto.com/u_16213318/11506041

安装mysql

~~~bash
npm install mysql mysql2
~~~

edit mysql conf 

~~~javascript
const mysql = require('mysql2/promise')
const connection = mysql.createPool({
  host: 'localhost',
  user: 'username',
  password: 'password',
  database: 'database'
})

module.exports = connection;
~~~

use mysql in vue file

~~~vue
<template>
  <div>
      Hello3
      <div v-for="obj in apiData" :key="obj.id">
          <span style="color:red">{{ obj }}</span>
      </div>
  </div>
</template>

<script setup>
import { useRoute } from 'vue-router';
 


const route = useRoute();
useSeoMeta({
  title: "hello world",
  description: "Seo info"
});

// 获取 url 参数
const id = route.params.id;
console.log(id); // 这里应该打印出 2

// 请求 api 获取数据
const { data: apiData } = await useFetch(`http://localhost:3000/api/Helloa/${id}`);
console.log("data", apiData);
</script>

~~~

this is js file

~~~bash
- server
  - Helloa
    - [id].ts
~~~

~~~tsx
// server/api/Helloa/[id].js
import db from '~/conf/db';

export default defineEventHandler(async (event) => {
    const { id } = event.context.params; // 获取 id 参数
    console.log(`Fetching user with id: ${id}`);

    return db.query("SELECT * FROM user WHERE id = ?", [id])
        .then(([rows]) => {
            return rows; // 返回数据行
        })
        .catch(error => {
            return { error: error.message }; // 返回错误信息
        });
});
~~~

## 传递

## 防止sql注入的方法

~~~java
 return db.query("SELECT * FROM user WHERE id = ?", [id])
~~~

这里的 `?` 是占位符，`id` 的值会被安全地转义，防止恶意输入影响 SQL 语句的结构，从而避免 SQL 注入攻击。

## 防止xxs攻击

### A. Vue特性

Nuxt和Vue都是默认对插值的（如 `{{ variable }}`）进行 HTML 转义，防止 XSS 攻击。确保始终使用 Vue 的插值语法来显示用户输入的数据。

### B. 尽量避免v-html渲染

如果你使用 `v-html` 来渲染 HTML 内容，请确保内容是可信的，并且没有用户输入的恶意代码。避免直接使用来自用户输入的内容。

但是我们的业务需求是把语雀的html拿过来渲染，如果我们语雀中的doc存在攻击代码那么就麻烦了，虽然也可以使用DOMPurify进行处理第三方html，但是处理后的数据是否正常？还没有得到认证

### C.设置安全头设计

在nuxt.config.ts

~~~java
export default {
    head: {
        meta: [
            {
                'http-equiv': 'Content-Security-Policy',
                content: "default-src 'self'; script-src 'self';"
            }
        ]
    }
}
~~~

### D. use https

通过 HTTPS 保护数据传输，防止中间人攻击（MITM），确保数据的安全性。



