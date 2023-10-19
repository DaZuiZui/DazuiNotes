# 解决EasyExcel3.1.1动态位数字列设置保留2位小数不生效 #3503

## 您的需求

​	动态位数字列设置保留2位小数不生效，想要保留两位小数生效。

## 我的想法

​	您好，在我复现过您的问题后，我发现了您的问题在您的

~~~java
        String stringValue = cellData.getStringValue();
~~~

代码`stringValue` 是一个 `String` 类型的变量，这意味着它包含的数据会被当作字符串处理。如果您使用这个字符串值来设置 `cellData` 的数据类型为 `NUMBER` 并尝试设置数据格式，这可能无法正确工作，因为数据类型是字符串而不是数字。您应该把它正确的设置为小数类型。

如果是小数请使用进行获取，或您将它设置为正确的数据类型

~~~java
cellData.getNumberValue()
~~~

![image-20231017135811807](/Users/yangyida/Library/Application Support/typora-user-images/image-20231017135811807.png)

## 我的尝试解决方案代码

~~~java
    @Override
    protected void setContentCellStyle(CellWriteHandlerContext context) {
        if (stopProcessing(context) || CollectionUtils.isEmpty(contentWriteCellStyleList)) {
            return;
        }
        WriteCellData<?> cellData = context.getFirstCellData();

        if (context.getRelativeRowIndex() == null || context.getRelativeRowIndex() <= 0) {
            WriteCellStyle.merge(contentWriteCellStyleList.get(0), cellData.getOrCreateStyle());
        } else {
            WriteCellStyle.merge(
                    contentWriteCellStyleList.get(context.getRelativeRowIndex() % contentWriteCellStyleList.size()),
                    cellData.getOrCreateStyle());
        }
        if (context.getColumnIndex() == 0) {
            // 是第一列 左对齐
            WriteCellStyle cellStyle = cellData.getOrCreateStyle();
            cellStyle.setHorizontalAlignment(HorizontalAlignment.LEFT);
            cellData.setWriteCellStyle(cellStyle);
        }

        // 是否是小数，小数需要格式化
        if (cellData.getNumberValue() != null) {
            cellData.setType(CellDataTypeEnum.NUMBER);
            WriteCellStyle cellStyle = cellData.getOrCreateStyle();
            DataFormatData dataFormatData = new DataFormatData();
            dataFormatData.setFormat("0.00");
            cellStyle.setDataFormatData(dataFormatData);
        }

        // 清空字符串值
        cellData.setStringValue(null);
    }

  
~~~



祝您生活愉快，工作顺利。