# ShaderToy 笔记

## 结构

```glsl
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
	// 图形全部渲染为红色
    fragColor = vec4(1., 0., 0.,1.0);
}
```

图形渲染时每个像素的着色都会调用 mainImage。mainImage 的参数中，fragColor 用于存储用于给像素着色的颜色，fragCoord 存储需要着色的像素的坐标，坐标以像素为单位。

fragColor 里, 最后一位被接受端忽略。

## 系统自带变量

### iResolution

iResolution.x 为画布宽度。iResolution.y 为画布高度。fragCoord 的取值范围由 iResolution 决定。

通常不直接用 fragCoord，在图形学里常做的一个操作是归一化，需要将像素坐标转换为 0 ~ 1 范围内：

```glsl
vec2 uv = fragCoord / iResolution.xy;
```

于是像素从画布左下角为零点，然后 x 向右增加至 1，y 向上增加至 1

## 黑红渐变

```glsl
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // 归一化
    vec2 uv = fragCoord/iResolution.xy;

    // 黑-红渐变
    fragColor = vec4(uv.x, 0., 0., 0.5);
}
```
画板左边颜色为黑色 `(0, 0, 0, 1)`，右边为红色`(1, 0, 0, 1)`。颜色随着 x 的增长逐渐从 `(0, 0, 0, 1)`到`(1, 0, 0, 1)`。

## 四角渐变
```glsl
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // 归一化
    vec2 uv = fragCoord/iResolution.xy;

    // 四角渐变
    fragColor = vec4(uv.x, uv.y, 0., 0.5);
}
```
左下角为黑色`(0,0,0,1)`，左上角为绿色`(0, 1, 0, 1)`，右下角为红色`(1, 0, 0, 1)`，右上角为黄色`(1, 1, 0, 1)`。

## 绘制圆形
```glsl
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // 归一化
    vec2 uv = fragCoord/iResolution.xy;
    
    // 计算向量长度
    float d = length(uv);

    // 向量长度大于等于1为白色，小于1为黑色
    fragColor = vec4(vec3(d), 0.5);
}
```
## 让圆形居中
```glsl
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // 归一化
    vec2 uv = fragCoord/iResolution.xy;

	// 平移
    uv -= .5;
    
    // 计算向量长度
    float d = length(uv);

    // 向量长度大于等于1为白色，小于1为黑色
    fragColor = vec4(vec3(d), 0.5);
}
```

## 设置圆形半径
```glsl
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // 归一化
    vec2 uv = fragCoord/iResolution.xy;
    
    uv -= .5;
    
    // 计算向量长度
    float d = length(uv);
    
    float color;
    if(d < .3){
        color = 1.;
    } else {
        color = 0.;
    }

    fragColor = vec4(vec3(color), 1.0);
}
```

## 绘制正圆
```glsl
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // 归一化
    vec2 uv = fragCoord/iResolution.xy;
    
    uv -= .5;
    
    // 计算宽高比
    float aspect = iResolution.x / iResolution.y;
    
    // 补偿宽高比：还原到x的尺度，缩放到y的尺度
    uv.x *= aspect;
    
    // 计算向量长度
    float d = length(uv);
    
    float color;
    if(d < .3){
        color = 1.;
    } else {
        color = 0.;
    }

    fragColor = vec4(vec3(color), 1.0);
}
```

## 消除锯齿
```glsl
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // 归一化
    vec2 uv = fragCoord/iResolution.xy;
    
    uv -= .5;
    
    // 计算宽高比
    float aspect = iResolution.x / iResolution.y;
    
    // 补偿宽高比：还原到x的尺度，缩放到y的尺度
    uv.x *= aspect;
    
    // 计算向量长度
    float d = length(uv);
    
    // 半径
    float r = 0.3;
    
    // 平滑处理
    // 在 r-0.01 ~ r 之间的像素采取平滑处理
    float color = smoothstep(r, r-0.01, d);

    fragColor = vec4(vec3(color), 1.0);
}
```

## 添加画圆函数
```glsl
float drawCircle(vec2 uv, vec2 center, float radius, float blur){
    float d = length(uv - center);
    float color = smoothstep(radius, radius - blur, d);
    
    return color;
}


void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // 归一化
    vec2 uv = fragCoord/iResolution.xy;
    
    uv -= .5;
    
    // 计算宽高比
    float aspect = iResolution.x / iResolution.y;
    
    // 补偿宽高比：还原到x的尺度，缩放到y的尺度
    uv.x *= aspect;
    
    float color = drawCircle(uv, vec2(.0, .0), .4, .01);
    color += drawCircle(uv, vec2(0.5, .3), .1, .01);

    fragColor = vec4(vec3(color), 1.0);
}
```

## 绘制矩形
```glsl
float Bound(float p, float start, float end, float blur)
{
    float step1 = smoothstep(start, start + blur, p);
    float step2 = smoothstep(end + blur, end, p);
    return step1 * step2;
}

float Rect(vec2 uv, float left, float right, float bottom, float up, float blur)
{
    float bound1 = Bound(uv.x, left, right, blur);
    float bound2 = Bound(uv.y, bottom, up, blur);
    return bound1 * bound2;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // 归一化
    vec2 uv = fragCoord/iResolution.xy;
    
    uv -= .5;
    
    // 计算宽高比
    float aspect = iResolution.x / iResolution.y;
    
    // 补偿宽高比：还原到x的尺度，缩放到y的尺度
    uv.x *= aspect;
    
    float mask = Rect(uv, -.2, .2, -.3, .3, .01);
    
    vec3 color = vec3(1.0, 1.0, 1.0) * mask;
    
    fragColor = vec4(vec3(color), 1.0);
    
}
```

## 会动的波浪线
```glsl
/**
 * 绘制边界
 * p 输入值
 * start 边界起始值
 * end 边界结束值
 * blur 模糊区宽度
 */
float Bound(float p, float start, float end, float blur)
{
    float step1 = smoothstep(start, start + blur, p);
    float step2 = smoothstep(end + blur, end, p);
    return step1 * step2;
}

/**
 * 绘制矩形
 * uv 归一化之后的像素坐标
 * left 左侧值
 * right 右侧值
 * bottom 下侧值
 * up 上侧值
 * blur 模糊区宽度
 */
float Rect(vec2 uv, float left, float right, float bottom, float up, float blur)
{
    // 绘制左右边界
    float bound1 = Bound(uv.x, left, right, blur);
    // 绘制上下边界
    float bound2 = Bound(uv.y, bottom, up, blur);
    return bound1 * bound2;
}

float Remap01(float a, float b, float t)
{
    return (t - a) / (b - a);
}

float Remap(float a, float b, float c, float d, float t)
{
    return Remap01(a, b, t) * (d - c) + c;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // 归一化
    vec2 uv = fragCoord/iResolution.xy;
    
    // 调整原点为中心
    uv -= .5;
    
    // 计算宽高比
    float aspect = iResolution.x / iResolution.y;
    
    // 补偿宽高比：还原到x的尺度，缩放到y的尺度
    uv.x *= aspect;
    
    // 系统时间：单位秒
    float t = iTime;
    
    float x = uv.x;
    float y = uv.y;
    
    // 计算 y 偏移值
    // x 取值为 [-0.5, 0.5], * 8.0 之后变为 [-4, 4]，才能出现 sin 的一个周期
    // y 取值为 [-0.5, 0.5], sin 结果取值为 [-1, 1], * 0.1 才能使 y - m 不超出绘图区域
    float m = sin(x*8.0 + t*5.0) * 0.1;
    
    // 偏移 y 值，产生波浪线效果
    y -= m;
    
    float blur = Remap(-.5, .5, .01, .25, x);
    blur *= pow(blur * 4., 2.);
    
    float mask = Rect(vec2(x, y), -.5, .5, -.1, .1, blur);
    
    vec3 color = vec3(1.0, 1.0, 1.0) * mask;
    
    fragColor = vec4(vec3(color), 1.0);
    
}
```

## 参考资料

1. [面向新手的 ShaderToy —— 入门篇](https://zydiii.github.io/ShaderToy-For-Beginners0)
2. 
