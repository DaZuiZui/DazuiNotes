# 缺少GeoTools依赖且无法下载 解决文档
y51288033@outlook.com

官网：https://docs.geotools.org/latest/userguide/tutorial/quickstart/maven.html

如果直接在Maven仓库会出现无法下载，请在pom文件告诉Maven去哪里下载这个jar包

```xml
 <repositories>
    <repository>
    <id>osgeo</id>
    <name>OSGeo Release Repository</name>
    <url>https://repo.osgeo.org/repository/release/</url>
    <snapshots><enabled>false</enabled></snapshots>
    <releases><enabled>true</enabled></releases>
    </repository>
    <repository>
    <id>osgeo-snapshot</id>
    <name>OSGeo Snapshot Repository</name>
    <url>https://repo.osgeo.org/repository/snapshot/</url>
    <snapshots><enabled>true</enabled></snapshots>
    <releases><enabled>false</enabled></releases>
    </repository>
  </repositories>
```

<repositories> 放在dependencies标签下面例如

~~~xml
<project
  xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.geotools.tutorial</groupId>
  <artifactId>quickstart</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>
  <name>GeoTools Demo</name>
  <url>http://maven.apache.org</url>
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <geotools.version>31-SNAPSHOT</geotools.version>
    <maven.deploy.skip>true</maven.deploy.skip>
  </properties>
  <dependencies>
 			........
  </dependencies>
  <repositories>
    <repository>
    <id>osgeo</id>
    <name>OSGeo Release Repository</name>
    <url>https://repo.osgeo.org/repository/release/</url>
    <snapshots><enabled>false</enabled></snapshots>
    <releases><enabled>true</enabled></releases>
    </repository>
    <repository>
    <id>osgeo-snapshot</id>
    <name>OSGeo Snapshot Repository</name>
    <url>https://repo.osgeo.org/repository/snapshot/</url>
    <snapshots><enabled>true</enabled></snapshots>
    <releases><enabled>false</enabled></releases>
    </repository>
  </repositories>
</project>
~~~

## 依然无法安装点击Maven的刷新

请手动在Terminal输入

~~~xml
mvn install -U
~~~

## 公司项目中依然无法下载该依赖？

请创建一个新的空白Maven项目，在新的空白Maven项目下载好后，在切回公司项目刷新Maven项目即可

## 依然存在问题？

将淘宝镜像或其他第三方镜像切换为国际Maven默认的仓库。