# EsayExcel读取合并单元格

​	如果我们Excel读取合并单元格直接读出来是null的，我们必须做一些手段进行处理，

## Excel信息

| 主题     | 合并  | 合并 | 合并 | 合并  | 合并 | 合并 |
| -------- | ----- | ---- | ---- | ----- | ---- | ---- |
| 地址     | A信息 | 合并 | 合并 | B信息 | 合并 | 合并 |
| 与上合并 | A存   | 姓名 | 职务 | 区域  | 姓名 | 职务 |

## 实体类

```java
import com.alibaba.excel.annotation.ExcelProperty;

public class LeaderInfo {
    @ExcelProperty(value = "地址", index = 0)
    private String township;

    @ExcelProperty(value = {"A信息","A存"}, index = 1)
    private String village;

    @ExcelProperty(value = {"A信息","姓名"}, index = 2)
    private String fieldLeaderName;

    @ExcelProperty(value = {"A信息","职务"}, index = 3)
    private String fieldLeaderPosition;

    @ExcelProperty(value = {"B信息","区域"},index = 4)
    private String fieldLeaderRegion;

    @ExcelProperty(value = {"B信息","姓名"}, index = 5)
    private String gridLeaderName;

    @ExcelProperty(value = {"B信息","职务"},index = 6)
    private String gridLeaderPosition;

    // 构造方法、getter 和 setter 方法省略
}
```

## 基础写法-不可以读取合并内容

```java
import java.util.List;

public class ExcelReader {
    public static void main(String[] args) {
        String fileName = "xxxxx.xls"; // 替换为你的 Excel 文件路径
        String sheetName = "sheet1";

        LeaderInfo listener = new LeaderInfo();

        // 使用 EasyExcel 读取 Excel 文件
        List<LeaderInfo> list = EasyExcel.read(fileName).head(LeaderInfo.class).sheet(sheetName).headRowNumber(3).doReadSync();

        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
    }
}
```

打印出来：

~~~java
LeaderInfo{township='null', village='null', fieldLeaderName='null', fieldLeaderPosition='null', fieldLeaderRegion='测试数据', gridLeaderName='测试名字', gridLeaderPosition='测试职位'}
~~~

##  更改后可以读取合并内容

### Listener 监听文件	

```java
package com.example.easyexcelisusse.com.listen;
 
import com.alibaba.excel.context.AnalysisContext;
import com.alibaba.excel.enums.CellExtraTypeEnum;
import com.alibaba.excel.event.AnalysisEventListener;
import com.alibaba.excel.metadata.CellExtra;
import com.example.easyexcelisusse.com.domain.LeaderInfo;


import java.util.ArrayList;
import java.util.List;
 

public class CustomAnalysisEventListener extends AnalysisEventListener<LeaderInfo> {
 
    private int headRowNum;
 
    public CustomAnalysisEventListener(int headRowNum) {
        this.headRowNum = headRowNum;
    }
 
    private List<LeaderInfo> list = new ArrayList<>();
 
    private List<CellExtra> cellExtraList = new ArrayList<>();

    @Override
    public void invoke(LeaderInfo excelData, AnalysisContext analysisContext) {

        list.add(excelData);
    }
 
    @Override
    public void extra(CellExtra extra, AnalysisContext context) {
        CellExtraTypeEnum type = extra.getType();
        switch (type) {
            case MERGE: {
                if (extra.getRowIndex() >= headRowNum) {
                    cellExtraList.add(extra);
                }
                break;
            }
            default:{
            }
        }
    }

    @Override
    public void doAfterAllAnalysed(AnalysisContext analysisContext) {
 
    }
 
    public List<LeaderInfo> getList() {
        return list;
    }
 
    public List<CellExtra> getCellExtraList() {
        return cellExtraList;
    }
}
```

### 读取Excel业务类

```java
package com.example.easyexcelisusse.com.utils;
 
import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.annotation.ExcelProperty;
import com.alibaba.excel.enums.CellExtraTypeEnum;
import com.alibaba.excel.metadata.CellExtra;
import com.example.easyexcelisusse.com.domain.LeaderInfo;
import com.example.easyexcelisusse.com.listen.CustomAnalysisEventListener;
//import com.example.model.ExcelData;
//import lombok.extern.slf4j.Slf4j;
 
import java.lang.reflect.Field;
import java.util.List;
 
 

public class ExcelRead {
 
    private static final int HEAD_ROW_NUM = 2;
    private static final String FILEPATH = "/Users/yangyida/Downloads/桦甸市田长制“3+1”名单(1).xls";
    String sheetName = "村级田长、网格长名单";

    public List<LeaderInfo> list() {
        List<LeaderInfo> excelDataList;
        CustomAnalysisEventListener listener = new CustomAnalysisEventListener(HEAD_ROW_NUM);
        EasyExcel.read(FILEPATH, LeaderInfo.class, listener).extraRead(CellExtraTypeEnum.MERGE).sheet(sheetName).doRead();
        excelDataList = listener.getList();
        List<CellExtra> cellExtraList = listener.getCellExtraList();
        if (cellExtraList != null && cellExtraList.size() > 0) {
            mergeExcelData(excelDataList, cellExtraList, HEAD_ROW_NUM);
        }
        return excelDataList;
    }
 
    private void mergeExcelData(List<LeaderInfo> excelDataList, List<CellExtra> cellExtraList, int headRowNum) {
        cellExtraList.forEach(cellExtra -> {
            int firstRowIndex = cellExtra.getFirstRowIndex() - headRowNum;
            int lastRowIndex = cellExtra.getLastRowIndex() - headRowNum;
            int firstColumnIndex = cellExtra.getFirstColumnIndex();
            int lastColumnIndex = cellExtra.getLastColumnIndex();
            //获取初始值
            Object initValue = getInitValueFromList(firstRowIndex, firstColumnIndex, excelDataList);
            //设置值
            for (int i = firstRowIndex; i <= lastRowIndex; i++) {
                for (int j = firstColumnIndex; j <= lastColumnIndex; j++) {
                    setInitValueToList(initValue, i, j, excelDataList);
                }
            }
        });
    }
 
    private void setInitValueToList(Object filedValue, Integer rowIndex, Integer columnIndex, List<LeaderInfo> data) {
        LeaderInfo object = data.get(rowIndex);
 
        for (Field field : object.getClass().getDeclaredFields()) {
            field.setAccessible(true);
            ExcelProperty annotation = field.getAnnotation(ExcelProperty.class);
            if (annotation != null) {
                if (annotation.index() == columnIndex) {
                    try {
                        field.set(object, filedValue);
                        break;
                    } catch (IllegalAccessException e) {
                        System.out.println("error");
                    }
                }
            }
        }
    }
 
    private Object getInitValueFromList(Integer firstRowIndex, Integer firstColumnIndex, List<LeaderInfo> data) {
        Object filedValue = null;

        LeaderInfo object = data.get(firstRowIndex);
        for (Field field : object.getClass().getDeclaredFields()) {
            field.setAccessible(true);
            ExcelProperty annotation = field.getAnnotation(ExcelProperty.class);
            if (annotation != null) {
                if (annotation.index() == firstColumnIndex) {
                    try {
                        filedValue = field.get(object);
                        break;
                    } catch (IllegalAccessException e) {
                        System.out.println("error");
                    }
                }
            }
        }
        return filedValue;
    }
}
```

### 测试类

```java
package com.example.easyexcelisusse.com;

import com.example.easyexcelisusse.com.domain.LeaderInfo;
import com.example.easyexcelisusse.com.utils.ExcelRead;

import java.util.List;

public class Main {
    public static void main(String[] args) {
        ExcelRead excelRead = new ExcelRead();
        List<LeaderInfo> list = excelRead.list();
        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
    }
}

```

**打印结果:**

~~~java

LeaderInfo{township='a区域', village='b村', fieldLeaderName='测试姓名', fieldLeaderPosition='测试职位', fieldLeaderRegion='测试组织', gridLeaderName='测试姓名', gridLeaderPosition='测试职位'}

~~~

