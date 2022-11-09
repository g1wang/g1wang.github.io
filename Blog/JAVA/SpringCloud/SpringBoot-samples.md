# SpringBoot-samples

## Excel导出

### SXSSFWorkbook POI 
[项目地址](https://github.com/g1wang/springboot-samples/tree/master/excel-sample)

excel 导出单个sheet 限制100万条

#### poi tempfile
生成excel时，SXSSFWorkbook POI 临时文件夹“poifiles”问题处理
- code
```
@Component  
public class ExcelConfig {  
  
    @Value("${application.tmp.path}")  
    private String applicationTmpPath;  
  
    /**  
     * 设置使用SXSSFWorkbook对象导出excel报表时，TempFile使用的临时目录，代替{java.io.tmpdir}  
     */    @PostConstruct  
    public void setExcelSXSSFWorkbookTmpPath() {  
        String excelSXSSFWorkbookTmpPath = applicationTmpPath + "/poifiles";  
        File dir = new File(excelSXSSFWorkbookTmpPath);  
        if (!dir.exists()) {  
            dir.mkdirs();  
        }  
        TempFile.setTempFileCreationStrategy(new DefaultTempFileCreationStrategy(dir));  
    }  
}

```
- 配置文件
```
application:  
  tmp:  
    path: ./excel-sample/tmp/poifile
```