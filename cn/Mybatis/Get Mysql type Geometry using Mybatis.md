# Get Mysql type Geometry using Mybatis



## 方案A 使用ST_AsText(l.coordinates)  查询速度会慢因转换字符串数据大小会大

### mapper层

~~~sql
select ST_AsText(coordinates) as 'coordinates' from table1
~~~

### domain 层

```java
public class Entry implements Serializable {
    private Long id;
    private String coordinates;
    private String name;
  	//省略set get 构造方法
}
```

## 方案B 获取geometry的字节流

**首先来说我们Mybatis是不支持获取Geometry的**，如果我们想要获取到那么就要以byte[]字节流的方式获取这个Geometry字段,然后在将字节流删除前4位转换为十六进制，然后在通过Geo工具如	org.geotools.data.postgis.WKBReader or  com.vividsolutions.jts.io.WKBReader or org.locationtech.jts.io.WKBReader 进行转换为我们数据库所对应的Geometry数据。

### 为什么删除前4位

因为Mysql的Geometry存放的是WKB格式，前4位是SRID值，所以需要删除前4位再去进行处理。

SRID是“空间参考标识符” (Spatial Reference Identifier)

### 数据库查询代码

```java
@Select("SELECT coordinates FROM locations")
public List<Entry> getall();
```

### 实体类方法

```java
public class Entry implements Serializable {
    private Long id;
    private byte[] coordinates;
    private String name;
    //省略set get 构造方法
}
```

###  转换业务方法

```java
		//测试
    @GetMapping("/test")
    public String test() throws SQLException, IOException, ParseException {
				//从数据库获取Geometry
        List<Entry> getall = testMapper.getall();
      	//封装好返回给前端的Geometry
        List<Geometry> list = new ArrayList<>();
      	//处理数据转换从字节流转换为我们想要的Geometry
        for (Entry entry : getall) {
            byte[] coordinates = entry.getCoordinates();
            String string = byteArrayToHexString(coordinates);
            Geometry geometry = bytesToGeometry(string);
            list.add(geometry);
        }
      
        return list.toString();
    }

		//下面的代码不要碰
		private static String byteArrayToHexString(byte[] bytes) {
        StringBuilder result = new StringBuilder();
        for (int i = 4; i < bytes.length; i++) {
            result.append(String.format("%02X", bytes[i]));
        }

        return result.toString();
    }

    private static org.locationtech.jts.geom.Geometry bytesToGeometry(String geomBytes) {
        byte[] bytes =  WKBReader.hexToBytes(geomBytes);

        //用org.geotools.data.postgis.WKBReader 或者com.vividsolutions.jts.io.WKBReader、org.locationtech.jts.io.WKBReader转都ok
        org.locationtech.jts.geom.Geometry geo = null;
        try {
            WKBReader wkbReader = new  WKBReader();
            geo = wkbReader.read(bytes);
            //System.out.println("geotools: " + geo.getCoordinates().length);
        } catch (ParseException e) {
            //System.err.println("geotoolsError: " + e.getMessage());
        }
//
//        try {
//            com.vividsolutions.jts.io.WKBReader wkbReader1 = new com.vividsolutions.jts.io.WKBReader();
//            Geometry geometry = wkbReader1.read(bytes);
//            System.out.println("vividsolutions: " + geometry.getCoordinates().length);
//        } catch (ParseException e) {
//            System.err.println("vividsolutionsError: " + e.getMessage());
//        }
//
//        try {
//            org.locationtech.jts.io.WKBReader wkbReader2 = new org.locationtech.jts.io.WKBReader();
//            org.locationtech.jts.geom.Geometry geometry = wkbReader2.read(bytes);
//            System.out.println("locationtech: " + geometry.getCoordinates().length);
//        } catch (org.locationtech.jts.io.ParseException e) {
//            System.err.println("locationtechError: " + e.getMessage());
//        }
        return geo;
    }
```