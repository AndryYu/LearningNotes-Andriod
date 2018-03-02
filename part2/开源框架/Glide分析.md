## Glide分析
### 1.Glide基本使用
Glide的一个完整的请求至少需要三个参数，代码如下
```
  Glide.with(context)
       .load(url)
       .into(imageView)
```
* with(Context context) - 需要上下文
* load(String url) - 网络图片URL、本地路径等等
* into(ImageView imageView) - 显示图片的目标ImageView

**占位图设置**<br/>
placeholder()和error()的参数都是只支持int和Drawable类型的参数。
* placeholder(R.drawable.place_image) //图片加载出来前，显示的图片
* error(R.drawable.error_image) //图片加载失败后，显示的图片

**缩略图**<br/>
* thumbnail(0.2f) //参数是float类型，作为其倍数大小

**动画开关**<br/>
Glide是可以自定义动画效果的，这个在后面会讲解
* crossFade() //开启Glide默认的图片的淡出淡入动画
* crossFade(int duration) //可以控制动画的持续时间，单位是ms

**图片大小与裁剪**<br/>
* override(width,height) //这里的单位是px
* CenterCrop() //将图片按比例缩放到足以填充ImageView的尺寸，但是图片可能会显示不完整
* FitCenter()  //图片缩放到小于等于ImageView的尺寸，这样图片是显示完整了，但是ImageView就可能不会填满了

**图片的缓存处理**<br/>
* skipMemoryCache(true) //告诉Glide跳过内存缓存
* diskCacheStrategy(diskCacheStrategy.NONE) //设置磁盘缓存类别

DiskCacheStrategy 的枚举意义：
```
  DiskCacheStrategy.NONE 什么都不缓存
  DiskCacheStrategy.SOURCE 只缓存全尺寸图
  DiskCacheStrategy.RESULT 只缓存最终的加载图
  DiskCacheStrategy.ALL 缓存所有版本图（默认行为）
```

**图片的优先级**<br/>
* priority(priority.HIGH) //配合Priority枚举来设置图片加载的优先级
```
  Priority.LOW
  Priority.NORMAL
  Priority.HIGH
  Priority.IMMEDIAT
```

**显示Gif和Video**<br/>
* asGif()

Glide只显示本地视频
```
  String filePath = "/storrage/emulated/0/Pictures/video.mp4";
  Glide.with( context )
      .load( Uri.fromFile( new File( filePath ) ) )
      .into( imageView );
```

### 2.Glide进阶使用
#### 2.1 Target篇
Target其实就是整个图片的加载的生命周期，所以我们就可以通过它在图片加载完成之后获取到Bitmap。如果我们使用 Target 的话，这个 Target 就有可能独立于普通的Activity/Fragment的生命周期以外，这时候我们就需要使用 context.getApplicationContext() 的上下文了，这样只有在应用完全停止时 Glide 才会杀死这个图片请求。
* SimpleTarget
* ViewTarget (自定义View)

#### 2.2 Transformations篇
如果你只是想要对图片（不是 Gif 和 video）做常规的 bitmap 转换，我们推荐你使用抽象类 BitmapTransformation。它简化了很多的实现，这应该能覆盖 95% 的应用场景啦。<br/>
调用 .transform() 方法，将自定义的 Transformation 的对象作为参数传递进去就可以使用你的 Transformation 了，这里也可以使用 .bitmapTransform() 但是它只能用于 bitmap 的转换。
```
  Glide.with(context)
      .load(mUrl)
      .transform(new RoundTransformation(context , 20))
      //.bitmapTransform( new RoundTransformation(context , 20) )
      .into(mImageView);
```

#### 2.3 Animate篇
两种动画类型：Xml动画和实现了ViewPropertyAnimation.Animator接口的类对象。
* animate(R.anim.zoom_in)
* animate(animator)




#### 2.4 Modules篇
