# Fix 3510 easyexcel使用registerWriteHandler方法自定义单元格样式不生效

## 您的需求

使用了EasyExcel.write(excelFilePath, Person.class)来设置表头后，再通过registerWriteHandler方法自定义单元格样式（增加了单元格边框线），此时自定义样式好像会被easyexcel中设置表头的方法所覆盖，变成默认的样式（没有单元格边框线），想要不被覆盖。

## 我的想法

你代码中发现一个小问题：在`afterCellCreate`方法中设置单元格样式之后，你需要调用`cell.setCellStyle(cellStyle)` 来将样式应用到单元格。你的代码缺少这一步。

## code

~~~java
package com.example.easyexcelisusse.issue3511;

import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.metadata.Head;
import com.alibaba.excel.write.handler.CellWriteHandler;
import com.alibaba.excel.write.metadata.holder.WriteSheetHolder;
import com.alibaba.excel.write.metadata.holder.WriteTableHolder;
import org.apache.poi.ss.usermodel.*;

import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        // 指定输出的Excel文件路径
        String excelFilePath = "example-write.xlsx";

        // 设置要输出的数据内容
        List<Person> personList = new ArrayList<>();
        personList.add(new Person("a01", "张三", "男", 10));
        personList.add(new Person("a02", "李四", "男", 20));
        personList.add(new Person("a03", "王五", "女", 30));

        // 将数据内容写入excel文件
        EasyExcel.write(excelFilePath, Person.class)
                .registerWriteHandler(new CellWriteHandler() {
                    @Override
                    public void afterCellCreate(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder, Cell cell, Head head, Integer relativeRowIndex, Boolean isHead) {
                        // 设置单元格边框
                        Workbook workbook = writeSheetHolder.getSheet().getWorkbook();
                        CellStyle cellStyle = workbook.createCellStyle();
                        cellStyle.setBorderTop(BorderStyle.THIN);
                        cellStyle.setBorderBottom(BorderStyle.THIN);
                        cellStyle.setBorderLeft(BorderStyle.THIN);
                        cellStyle.setBorderRight(BorderStyle.THIN);

                        // 设置标题字体和背景色
                        if (cell.getRowIndex() == 0) {
                            Font font = workbook.createFont();
                            font.setBold(true);
                            cellStyle.setFont(font);
                            cellStyle.setFillForegroundColor(IndexedColors.GREY_25_PERCENT.getIndex());
                            cellStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);
                        }
                        cell.setCellStyle(cellStyle); // 设置单元格样式
                    }
                })
                .sheet("Sheet1")
                .doWrite(personList);
    }
}

~~~

祝您生活愉快，工作顺利。

I wish you a happy life and smmoth work