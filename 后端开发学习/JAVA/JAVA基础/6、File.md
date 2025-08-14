**File对象就表示一个路径**

|​**​方法名称​**​|​**​说明​**​|
|---|---|
|`File(String pathname)`|根据​**​完整文件路径字符串​**​（如 `"/data/test.txt"`）创建文件对象。|
|`File(String parent, String child)`|通过​**​父目录路径字符串​**​ + ​**​子文件/目录名​**​（如 `"/data"`, `"test.txt"`）创建文件对象。|
|`File(File parent, String child)`|通过​**​父目录的 `File`对象​**​ + ​**​子文件/目录名​**​（如 `parentDir`, `"test.txt"`）创建文件对象。|

```java
String str ="路径"
File f1 = new File(str);

File f2 = new File(父级（字符串或File对象）,子路径);
```

|**​方法名称​**​|​**​说明​**​|
|---|---|
|`public boolean isDirectory()`|判断此路径名表示的 `File`是否为​**​文件夹​**​（目录）。|
|`public boolean isFile()`|判断此路径名表示的 `File`是否为​**​文件​**​（非目录）。|
|`public boolean exists()`|判断此路径名表示的 `File`​**​是否存在​**​于文件系统中。|
|`public long length()`|返回文件的​**​大小​**​（字节数量），对目录无效（返回 `0`）。|
|`public String getAbsolutePath()`|返回文件的​**​绝对路径​**​（完整路径，如 `"/data/test.txt"`）。|
|`public String getPath()`|返回定义文件时使用的​**​原始路径​**​（可能是相对路径，如 `"./test.txt"`）。|
|`public String getName()`|返回文件的​**​名称​**​（含后缀，如 `"test.txt"`）。|
|`public long lastModified()`|返回文件的​**​最后修改时间​**​（毫秒值，需配合 `Date`类转换可读格式）。|

| ​**​方法签名​**​                     | ​**​功能说明​**​                                 |
| -------------------------------- | -------------------------------------------- |
| `public boolean createNewFile()` | ​**​创建新空文件​**​（需确保父目录已存在，否则抛出 `IOException`） |
| `public boolean mkdir()`         | ​**​创建单级文件夹​**​（仅当父目录存在时成功）                  |
| `public boolean mkdirs()`        | ​**​创建多级文件夹​**​（自动补全不存在的父目录）                 |
| `public boolean delete()`        | ​**​删除文件或空文件夹​**​（非空文件夹删除失败）                 |

| **方法签名​**​                                       | ​**​功能说明​**​                                       |
| ------------------------------------------------ | -------------------------------------------------- |
| `public static File[] listRoots()`               | 列出系统所有可用的​**​文件系统根目录​**​（如 Windows 的 ``C:\`、``D:`） |
| `public String[] list()`                         | 获取当前路径下的​**​所有文件和子目录名称​**​（仅名字，不包含路径）              |
| `public String[] list(FilenameFilter filter)`    | 通过​**​文件名过滤器​**​获取当前路径下符合条件的文件和子目录名称               |
| `public File[] listFiles()`                      | 获取当前路径下的​**​所有文件和子目录的 `File`对象​**​（包含完整路径信息）       |
| `public File[] listFiles(FileFilter filter)`     | 通过​**​文件对象过滤器​**​获取当前路径下符合条件的 `File`对象             |
| `public File[] listFiles(FilenameFilter filter)` | 通过​**​文件名过滤器​**​获取当前路径下符合条件的 `File`对象              |