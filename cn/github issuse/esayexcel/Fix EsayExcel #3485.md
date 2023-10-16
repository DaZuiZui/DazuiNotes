# My idea

我没有想到更好的解决方案，我的思路就是使用POI进行解决。

i haven`t thought of a better solution, My idea is to user POI to solve it.

```java
package com.example.easyexcelisusse.todaydemo;

import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFRichTextString;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.FileOutputStream;
import java.lang.reflect.Field;

public class ExcelExport {

    public static void main(String[] args) {
        try {
            Workbook workbook = new XSSFWorkbook();
            Sheet sheet = workbook.createSheet("Sheet1");

            // 创建一个行
            Row row = sheet.createRow(0);

            // 创建一个单元格样式
            CellStyle style = workbook.createCellStyle();
            Font redFont = workbook.createFont();
            redFont.setColor(IndexedColors.RED.getIndex()); // 设置字体颜色为红色
            style.setFont(redFont);

            // 获取包含数据的对象
            YourDataObject dataObject = new YourDataObject();

            Field[] fields = dataObject.getClass().getDeclaredFields();

            for (Field field : fields) {
                field.setAccessible(true);

                // 创建一个单元格
                Cell cell = row.createCell(0);

                String fieldValue = (String) field.get(dataObject);

                // 创建一个带格式的富文本字符串
                XSSFRichTextString richText = new XSSFRichTextString(fieldValue);
                richText.applyFont(0, 1, redFont); // 设置第一个字符为红色

                cell.setCellValue(richText);
            }

            // 将工作簿保存到文件
            FileOutputStream fileOut = new FileOutputStream("excel_with_red_first_char.xlsx");
            workbook.write(fileOut);
            fileOut.close();

            workbook.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

class YourDataObject {
    private String name = "* 昵称";
}
```

祝你生活愉快，工作顺利。

wish you a happy life and smooth work