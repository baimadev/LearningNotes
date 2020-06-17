
[色彩处理](#1)   
[图形变换](#2) 
<h3 id="1"></h3>

#色彩处理
## ColorMatrix

```kotlin
val matrix = ColorMatrix()  
mPaint.setColorFilter(ColorMatrixColorFilter(matrix))
canvas.drawBitmap(bm,0,0,paint)
```

### 色调
第一个参数，系统分别使用0、1、2来代表R、G、B；第二个参数是需要处理的值。

```kotlin
val hueMatrix = ColorMatrix()  
huiMatrix.setRotate(0,hue0)
```
### 饱和度
当值为0时，图像变为灰度图。

```kotlin
val saturationMatrix = ColorMatrix()  
saturationMatrix.setSaturation(saturation)
```
### 亮度
当值为0时，图像变为全黑。

```kotlin
val lumMatrix = ColorMatrix()  
lumMatrix.setScale(lum,lum,lum,1)
```
### postConcat
矩阵的乘法运算，可以将效果叠加。

### 直接使用矩阵赋值
ColorMatrix是一个4*5的矩阵，可直接讲一个size为20的Float数组赋值给它。

```kotlin
val mColorMatrix = = FloatArray(20)
val matrix = ColorMatrix()  
matrix.set(mColorMatrix)
```
 
## 像素点分析

```kotlin
 /**
     * @param pixels   接收位图颜色值的数组
     * @param offset   The first index to write into pixels[]
     * @param stride   pixels[]中的行间距
     * @param x        The x coordinate of the first pixel to read from
     *                 the bitmap
     * @param y        The y coordinate of the first pixel to read from
     *                 the bitmap
     * @param width    The number of pixels to read from each row
     * @param height   The number of rows to read
     *
     */
    bitmap.getPixels(oldpx,0,bm.width,0,0,width,height) 

```
获取每个像素点具体的ARGB值。

```kotlin
 val color=oldPx[i]
 val r = Color.red(color)
 val g = Color.green(color)
 val b = Color.blue(color)
 val a = Color.alpha(color)
```
将新的RGBA值合成像素点。

```kotlin
newPx[i]=Color.argb(a,r1,g1,b1)
```
处理后的像素点数组重新设置给bitmap。

```kotlin
bmp.setPixels(newPx,0,width,0,0,width,height)
```
<h3 id="2"></h3>
#图形变换
##Android变形矩阵-Matrix
图形变换是用一个3*3的矩阵来变形图片。

```kotlin
val mImageMetrix = FloatArray(9)
matrix.setValues(mImageMetrix)
canvas.draw(mBitmap,matrix,null)
```
或者使用封装好的API。

```kotlin
matrix.setRotate() //旋转
matrix.setTranslate() //平移
matrix.setScale() //缩放
matrix.setSkew() //错切
canvas.draw(mBitmap,matrix,null)
```
set会重置矩阵中的所有值，post和pre不会，这两个方法常用来实现矩阵的混合作用。  
矩阵乘法不满足交换律，前乘和后乘是两种不同的运算。pre插入到队列最前面执行，post插到最后面执行，set会清除之前的操作。

## 像素块分析

```kotlin
    /**
     * @param bitmap The bitmap to draw using the mesh
     * @param meshWidth 需要的横向网格数目
     * @param meshHeight 需要的纵向网格数目 
     * @param verts 网格交叉点坐标数组
     * @param vertOffset verts数组中开始跳过的（x,y）坐标对的数目
     * @param colors May be null. Specifies a color at each vertex, which is interpolated across the
     *            cell, and whose values are multiplied by the corresponding bitmap colors. If not
     *            null, there must be at least (meshWidth+1) * (meshHeight+1) + colorOffset values
     *            in the array.
     * @param colorOffset Number of color elements to skip before drawing
     * @param paint May be null. The paint used to draw the bitmap
     */
    public void drawBitmapMesh(@NonNull Bitmap bitmap, int meshWidth, int meshHeight,  @NonNull float[] verts, int vertOffset, @Nullable int[] colors, int colorOffset,
            @Nullable Paint paint) 
```
最重要的是verts数组，其中每两位用来保存一个交织点，x1,y1,x2,y2...xn,yn。  
改变交织点坐标 --> 扭曲图形

# Paint
## PorterDuffXfermode

```kotlin
mPaint.setXfermode(PorterDuffXfermode(PorterDuff.Mode.SRC_IN))
canvas.drawBitmap(bm,0,0,mPaint)
```
[详解](https://www.jianshu.com/p/19997b0b5b24)

## Shader

- BitMapShader 着色器、图像填充、clamp、repeat、mirror

```kotlin
 val shader = BitmapShader(originBitmap,Shader.TileMode.CLAMP,Shader.TileMode.CLAMP)
 paint.shader = shader
 canvas!!.drawCircle(100f,100f,100f,paint)
```
- LinearGradient 线性渐变

```kotlin
paint.shader = LinearGradient(startX,startY,endX,endY,Color1,Color2,Shader.TileMode.CLAMP)
```

- RadialGradient 光束
- SweepGradient	梯度
- ComposeShader 混合

## PathEffect

用各种笔触效果绘制一个路径。

- CornerPathEffect 拐角圆滑
- DiscretePathEffect 产生许多杂点
- DashPathEffect 绘制虚线，用一个数组来设置各个点之间的距离，此后绘制就不断重复，另一个参数用来控制偏移量，可实现动态效果。
- PathDashPathEffect plus版，还可设置绘制显示点的图形，方形点虚线、圆形点虚线
- ComposePathEffect 组合PathEffect

```kotlin
  val Effects = ArrayList<PathEffect?>(6)
        Effects[0] = null
        Effects[1] = CornerPathEffect(20f)
        Effects[2] = DiscretePathEffect(3f,5f)
        Effects[3] = DashPathEffect(floatArrayOf(10f,20f,30f,40f),5f)
        val path = Path()
        path.addRect(0f,0f,8f,8f,Path.Direction.CCW)
        Effects[4] = PathDashPathEffect(path,12f,0f,PathDashPathEffect.Style.ROTATE)
        Effects[5] = ComposePathEffect(Effects[0],Effects[2])
        for(i in 0 until 6){
            paint.setPathEffect(Effects[i])
            canvas.drawPath(mPath,paint)
            canvas.translate(0f,200f)
        }
```

