# Fix EsayExcel #3481

## English

​	Hello This is Bryan yang(Dazui)，

### Your needs:

 Hello, I understand your needs. Your needs are to set different colors for each title.

### My solution

 First, let esayexcel create a template file, then insert data according to the template file and set no header.

### code	

~~~java
package com.example.easyexcelisusse.excel.Issues3491;

import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.write.metadata.WriteSheet;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * @author Bryan yang(Dazui) 2023-10-9
 * 解决方案思路，首先生成一个模版文件，然后在根据模版文件进行插入
 */
public class SetCellColorExample {
    public static void main(String[] args) throws IOException {
        String excelFilePath = "colored_cells.xlsx";

        //创建模版文件
        //create template file
        Workbook workbook = new XSSFWorkbook();
        Sheet sheet = workbook.createSheet("Sheet1");

        //创建模版文件标题头样式
        //create a template file header style
        CellStyle redStyle = workbook.createCellStyle();
        redStyle.setFillForegroundColor(IndexedColors.RED.getIndex());
        redStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);

        CellStyle greenStyle = workbook.createCellStyle();
        greenStyle.setFillForegroundColor(IndexedColors.GREEN.getIndex());
        greenStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);

        CellStyle yellowStyle = workbook.createCellStyle();
        greenStyle.setFillForegroundColor(IndexedColors.YELLOW.getIndex());
        greenStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);

        //创建标题头
        //create title style
        Row row = sheet.createRow(0);

        Cell cell1 = row.createCell(0);
        cell1.setCellValue("Cell1");
        cell1.setCellStyle(redStyle);

        Cell cell2 = row.createCell(1);
        cell2.setCellValue("Cell2");
        cell2.setCellStyle(greenStyle);

        Cell cell3 = row.createCell(2);
        cell3.setCellValue("Cell3");
        cell3.setCellStyle(yellowStyle);

        // 保存模版文件
        // save template file
        try (FileOutputStream fos = new FileOutputStream(excelFilePath)) {
            workbook.write(fos);
        }

        System.out.println("Excel save to" + excelFilePath);

        //use template file to insert data
        List<TeamInfo> list = new ArrayList<>();

        for (int i = 0; i < 10; i++) {
            TeamInfo teamInfo = new TeamInfo();
            teamInfo.setC1(i+"");
            teamInfo.setC2(i+"");
            teamInfo.setC3(i+"");
            list.add(teamInfo);
        }


        WriteSheet writeSheet = EasyExcel.writerSheet("Sheet1").head((List<List<String>>) null).build();
        EasyExcel.write("example1.xlsx", TeamInfo.class)
                .withTemplate(excelFilePath)
                .sheet("Sheet1").needHead(false).doWrite(list);

        System.out.println("Excel save to" + excelFilePath);
    }
}
~~~

I wish you a happy life and smooth work.

## Chinese       

​	您好，这里是Bryan Yang(Dazu)

### 您的需求：

​	您好我了解到您的需求，您的需求是为每一个标题设置不同的颜色。

### 我的解决方案

​	首先让esayexcel去创建一个模版文件，然后在根据模版文件去插入数据并且设置不需要标题头。

### code

~~~java
package com.example.easyexcelisusse.excel.Issues3491;

import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.write.metadata.WriteSheet;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * @author Bryan yang(Dazui) 2023-10-9
 * 解决方案思路，首先生成一个模版文件，然后在根据模版文件进行插入
 */
public class SetCellColorExample {
    public static void main(String[] args) throws IOException {
        String excelFilePath = "colored_cells.xlsx";

        //创建模版文件
        //create template file
        Workbook workbook = new XSSFWorkbook();
        Sheet sheet = workbook.createSheet("Sheet1");

        //创建模版文件标题头样式
        //create a template file header style
        CellStyle redStyle = workbook.createCellStyle();
        redStyle.setFillForegroundColor(IndexedColors.RED.getIndex());
        redStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);

        CellStyle greenStyle = workbook.createCellStyle();
        greenStyle.setFillForegroundColor(IndexedColors.GREEN.getIndex());
        greenStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);

        CellStyle yellowStyle = workbook.createCellStyle();
        greenStyle.setFillForegroundColor(IndexedColors.YELLOW.getIndex());
        greenStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);

        //创建标题头
        //create title style
        Row row = sheet.createRow(0);

        Cell cell1 = row.createCell(0);
        cell1.setCellValue("Cell1");
        cell1.setCellStyle(redStyle);

        Cell cell2 = row.createCell(1);
        cell2.setCellValue("Cell2");
        cell2.setCellStyle(greenStyle);

        Cell cell3 = row.createCell(2);
        cell3.setCellValue("Cell3");
        cell3.setCellStyle(yellowStyle);

        // 保存模版文件
        // save template file
        try (FileOutputStream fos = new FileOutputStream(excelFilePath)) {
            workbook.write(fos);
        }

        System.out.println("Excel save to" + excelFilePath);

        //use template file to insert data
        List<TeamInfo> list = new ArrayList<>();

        for (int i = 0; i < 10; i++) {
            TeamInfo teamInfo = new TeamInfo();
            teamInfo.setC1(i+"");
            teamInfo.setC2(i+"");
            teamInfo.setC3(i+"");
            list.add(teamInfo);
        }


        WriteSheet writeSheet = EasyExcel.writerSheet("Sheet1").head((List<List<String>>) null).build();
        EasyExcel.write("example1.xlsx", TeamInfo.class)
                .withTemplate(excelFilePath)
                .sheet("Sheet1").needHead(false).doWrite(list);

        System.out.println("Excel save to" + excelFilePath);
    }
}
~~~

 

祝您 生活愉快 工作顺利。