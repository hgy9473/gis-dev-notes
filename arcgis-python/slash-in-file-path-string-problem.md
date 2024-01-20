# 文件路径字符串中的反斜杠问题

Python 是在 Unix/Linux 环境中开发的，所以它支持的`/`拼接构成的文件路径字符串。而 Windows 中的文件路径字符串是 `\` 拼接构成的。

在 Windows 平台，解决这个问题有三种方法：

1、在字符串前加上`r`将其转为生字符串（raw string）。
```python
path = r'C:\test\test1.shp'
```

2、将字符串中的 `\` 全部换为 `/`。

```python
path = 'C:/test/test1.shp'
```

3、将字符串中的 `\` 全部换为 `\\`。

```python
path = 'C:\\test\\test1.shp'
```
