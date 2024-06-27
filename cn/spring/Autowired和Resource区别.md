# Autowired和Resource区别

## Autowired（属于Spring框架）

默认按照类型自动装备，可以用于构造函数，方法和字段。

## Resource

Resource是Java标准注解，默认按照昵称装配，Resource只能用于字段和setter方法。

## 为什么Resource可以装配到IOC

因为在Spring设计的时候考虑到了Java标注注解的兼容性，因此在扫描和解析注解的时候，Spring能够识别和处理@Resource注解。因为在内部有一个注解处理器，他会对Resource进行处理，按照name进行装配。