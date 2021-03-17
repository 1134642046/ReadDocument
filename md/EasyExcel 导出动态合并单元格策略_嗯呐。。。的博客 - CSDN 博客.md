> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_50067580/article/details/111559637)

EasyExcel 导出动态合并单元格策略
=====================

1. 导入依赖
-------

```
<dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>easyexcel</artifactId>
        <version>2.2.6</version>
    </dependency>
```

2. 合并单元格工具类
-----------

```
import com.alibaba.excel.metadata.CellData;
import com.alibaba.excel.metadata.Head;
import com.alibaba.excel.write.handler.CellWriteHandler;
import com.alibaba.excel.write.metadata.holder.WriteSheetHolder;
import com.alibaba.excel.write.metadata.holder.WriteTableHolder;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.CellType;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.util.CellRangeAddress;
import java.util.List;

public class ExcelMergeUtil implements CellWriteHandler {
private int[] mergeColumnIndex;
private int mergeRowIndex;

public ExcelMergeUtil() {
}

public ExcelMergeUtil(int mergeRowIndex, int[] mergeColumnIndex) {
    this.mergeRowIndex = mergeRowIndex;
    this.mergeColumnIndex = mergeColumnIndex;
}

@Override
public void beforeCellCreate(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder, Row row, Head head, Integer columnIndex, Integer relativeRowIndex, Boolean isHead) {

}

@Override
public void afterCellCreate(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder, Cell cell, Head head, Integer relativeRowIndex, Boolean isHead) {

}

@Override
public void afterCellDataConverted(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder, CellData cellData, Cell cell, Head head, Integer relativeRowIndex, Boolean isHead) {

}

@Override
public void afterCellDispose(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder, List<CellData> cellDataList, Cell cell, Head head, Integer relativeRowIndex, Boolean isHead) {

    //当前行
    int curRowIndex = cell.getRowIndex();
    //当前列
    int curColIndex = cell.getColumnIndex();

    if (curRowIndex > mergeRowIndex) {
        for (int i = 0; i < mergeColumnIndex.length; i++) {
            if (curColIndex == mergeColumnIndex[i]) {
                mergeWithPrevRow(writeSheetHolder, cell, curRowIndex, curColIndex);
                break;
            }
        }
    }
}


/**
 * 当前单元格向上合并
 *
 * @param writeSheetHolder
 * @param cell             当前单元格
 * @param curRowIndex      当前行
 * @param curColIndex      当前列
 */
private void mergeWithPrevRow(WriteSheetHolder writeSheetHolder, Cell cell, int curRowIndex, int curColIndex) {
    Object curData = cell.getCellTypeEnum() == CellType.STRING ? cell.getStringCellValue() : cell.getNumericCellValue();
    Cell preCell = cell.getSheet().getRow(curRowIndex - 1).getCell(curColIndex);
    Object preData = preCell.getCellTypeEnum() == CellType.STRING ? preCell.getStringCellValue() : preCell.getNumericCellValue();
    // 将当前单元格数据与上一个单元格数据比较
    Boolean dataBool = preData.equals(curData);
    //此处需要注意：因为我是按照订单号确定是否需要合并的，所以获取每一行第二列数据和上一行第一列数据进行比较，如果相等合并
    Boolean bool = cell.getRow().getCell(0).getStringCellValue().equals(cell.getSheet().getRow(curRowIndex - 1).getCell(0).getStringCellValue());
    if (dataBool && bool) {
        Sheet sheet = writeSheetHolder.getSheet();
        List<CellRangeAddress> mergeRegions = sheet.getMergedRegions();
        boolean isMerged = false;
        for (int i = 0; i < mergeRegions.size() && !isMerged; i++) {
            CellRangeAddress cellRangeAddr = mergeRegions.get(i);
            // 若上一个单元格已经被合并，则先移出原有的合并单元，再重新添加合并单元
            if (cellRangeAddr.isInRange(curRowIndex - 1, curColIndex)) {
                sheet.removeMergedRegion(i);
                cellRangeAddr.setLastRow(curRowIndex);
                sheet.addMergedRegion(cellRangeAddr);
                isMerged = true;
            }
        }
        // 若上一个单元格未被合并，则新增合并单元
        if (!isMerged) {
            CellRangeAddress cellRangeAddress = new CellRangeAddress(curRowIndex - 1, curRowIndex, curColIndex, curColIndex);
            sheet.addMergedRegion(cellRangeAddress);
        }
    }
}
```

}

3、导出方法（公司导出的工具类）
----------------

```
public static ExcelWriter getExcelWriterMerge(HttpServletResponse response, String excelName,int mergeRowIndex,int[] mergeColumeIndex) throws Exception{
    response.setContentType("application/vnd.ms-excel");
    response.setCharacterEncoding("utf-8");
    // 这里URLEncoder.encode可以防止中文乱码
    excelName = URLEncoder.encode(excelName, "UTF-8");
    response.setHeader("Content-disposition", "attachment;filename=" + excelName + ExcelTypeEnum.XLSX.getValue());

    ExcelWriter build = EasyExcel.write(response.getOutputStream()).registerWriteHandler(new ExcelMergeUtil(mergeRowIndex, mergeColumeIndex)).build();

    return build;
}
```

4、业务代码
------

```
try {
            //需要合并的列
            int[] mergeColumeIndex = {0,4,5,6,7,8,9,10};
            // 从那一行开始合并
            int mergeRowIndex = 1;
            //先执行合并策略
            excelWriter = ExcelUtil.getExcelWriterMerge(response, fileName, mergeRowIndex, mergeColumeIndex);
            //业务代码
            for (int i = 0 ; i < totalPage;i++){
                selectOrderDto.setPageNum(i);
                selectOrderDto.setPageSize(10000);
                String sheetName = "sheet"+i;
                PageInfo<ShowOrderByKitchen> page = selectOrderByKitchenId(selectOrderDto);
                List<ShowOrderByKitchen> list = page.getList();
                if (!list.isEmpty()){
                //进行写入操作
                    WriteSheet sheetWriter = EasyExcel.writerSheet(i,sheetName).head(ShowOrderByKitchen.class).build();
                    excelWriter.write(list,sheetWriter);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
            log.error("导出订单失败，{}",e);
        }finally {
            // 千万别忘记finish 会帮忙关闭流
            if (excelWriter != null) {
                excelWriter.finish();
            }
        }
    }
```

5、实体
----

```
public class ShowOrderByKitchen implements Serializable {

@ExcelProperty(value = "编码",index = 0)
private String orderId;

@ExcelProperty(value = "名称",index = 1)
private String dishName;

@ExcelProperty(value = "价格",index = 2)
private BigDecimal dishPrice;

@ExcelProperty(value = "数量",index = 3)
private Integer dishNumber;
```

}

6、测试结果
------

![](https://img-blog.csdnimg.cn/20201222154033253.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81MDA2NzU4MA==,size_16,color_FFFFFF,t_70#pic_center)  
这是根据订单号进行合并，工具类里进行两次判断，合并后就会呈现正确的格式，建议根据哪一列数据合并，就将这列数据放在 excel 的第一列，否则会出现空指针异常。