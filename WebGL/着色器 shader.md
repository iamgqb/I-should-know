存在两个着色器

- 顶点着色器 vertex shader
- 片段着色器 fragment shader

一对 顶点着色器 和 片段着色器 组成 **着色程序** `program`

每个着色器是含有一个 main 函数

```
void main() {

}
```

#### 顶点着色器 

重点是需要将某个值赋给 `gl_Position` 

用来定位顶点的位置

```
void main() {
  // 一些计算
  gl_Position = someValue;
}
```


#### 片段着色器 

重点是需要将某个值赋给 `gl_FragColor`

为当前图形提供颜色，每个像素点执行一次

```
void main() {
  // 一些计算
  gl_FragColor = someValue;
}
```
