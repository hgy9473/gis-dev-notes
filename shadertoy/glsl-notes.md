
## smoothstep
```glsl
genType smoothstep(float edge0, float edge1, genType x);

// 公式定义
genType t;
t = clamp((x - edge0) / (edge1 - edge0), 0.0, 1.0);
return t * t * (3.0 - 2.0 * t);
```
当 edge0 < edge1 时：
当 x < edge0, 返回 0；当 x > edge1，返回 1；当 edge0 < x < edge1，返回值从 0 到 1 递增。
![[Pasted image 20240322142659.png]]

当 edge0 > edge1 时：
当 x < edge0, 返回 1；当 x > edge1，返回 0；当 edge1 < x < edge0，返回值从 1 到 0 递减。
![[Pasted image 20240322143231.png]]


## clamp
```glsl
genType clamp(genType x,float minVal,float maxVal);
```
返回三个值中的中值。在 Excel 中可以有 MEDIAN 函数代替。


