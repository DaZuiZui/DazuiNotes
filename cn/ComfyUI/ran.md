# Nuxt3 学习日志

## 简介

Nuxt提供了机遇Node.js服务器渲染的方案，可以让vue在服务器SSR进行渲染，提高了加载速度和SEO，只要你愿意你也可以用它写全栈，Nuxt同时支持SSR和SPA。

适合场景C端领域，

## 项目创建

~~~bash
npx nuxi@latest init <project-name>
npm install
npm run dev
~~~

## What are SSR and SPA

SSR就是生成html发给客户端 优点 对搜索引擎友好

SPA就是动态渲染，                 优点流畅

## 为什么使用Nuxt3

因为搜索引擎都是根据html进行获取的，在一些网站都是动态获取的数据导致搜索引擎无法搜索到，所以用Nuxt3

让Vue在服务器SSR进行渲染可以解决这个问题，可以把所有内容实打实渲染出来。