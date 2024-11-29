# 给字段赋值为随机数

```python
# 给字段赋值随机数
ws = Workspace()  
ds = ws.get_datasource(0) 
# 或者 ds = Datasource.create("C:\Users\a1560\Downloads\贵阳市.udbx")
jg = ds['井盖']
jg.update_field_express("text", "round(abs(sin(random())) * 100, 0)")

```






# 参考资料
【1】 [https://iobjectspy.supermap.io/](https://iobjectspy.supermap.io/)
