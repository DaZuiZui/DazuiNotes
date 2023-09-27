# Bug处理总结

## 案例

访问此url https://sicnu-adms.htcube.top/#/user/permissionSharing?fileId=47&status=1

## 出现的问题？

无法获取到url中的参数，getQueryVariable("status")

## 希望成为什么样子？

通过getQueryVariable("status")获取到url参数。

## 如何造成的问题？

URL包含哈希路由（`#`），

##  为什么会造成此问题?

在开发版本我所使用传统的URL结构下，而线上环境我们使用含有hash路由

## 如何解决？

因为需要`window.location.hash` 获取整个哈希部分，包括路径和查询字符串，然后使用 `URLSearchParams` 来解析查询字符串，并从中获取参数值。这种方式适用于处理包含哈希路由的URL。

## 通过此bug学到了什么？

掌握了如何解析hash url下的参数。