## Android图片的三级缓存实现
#### 三级缓存
1. 网络缓存 从网络获取资源(异步加载)
2. 本地缓存 从本地获取数据(DiskLruCache file存储)
3. 内存缓存 从内存获取数据(LruCache)

### 采用LRU算法缓存：LruCache和DiskLruCache
#### LruCache
LruCache是一个泛型类，它内部采用一个LinkedHashMap以强引用的方式存储外界的缓存对象，其提供了get和put方法来完成缓存的获取和添加操作。当缓存满时，LruCache会移除较早使用的缓存对象，然后再添加新的缓存对象。
* 强引用： 直接的对象引用
* 软引用： 当一个对象只有软引用存在时，系统内存不足时此对象会被gc回收
* 弱引用： 当一个对象只有弱引用存在时，此对象会随时被gc回收
* 虚引用：

另外LruCache是线程安全的。

#### DiskLruCache
DiskLruCache用于实现存储设备缓存，即磁盘缓存，它通过将缓存对象写入文件系统从而实现缓存的效果。DiskLruCache得到了Android官方文档的推荐，但它不属于Android SDK的一部分。
**1.DiskLruCache的创建**<br/>
DiskLruCache并不能通过构造方法来创建，它提供了open方法用于创建自身，如下所示。
```
public static DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)
```
open方法有四个参数，其中第一个参数表示磁盘缓存在文件系统中的存储路径。第二个参数表示应用的版本号，一般设为1即可。第三个参数表示单个节点所对应的数据的个数，一般设为1即可。第四个参数表示缓存的总大小，比如50MB，当缓存大小超出这个设定值后，DiskLruCache会清除一些缓存从而保证总大小不大于这个设定值。
**2.DiskLruCache的缓存添加**<br/>
DiskLruCache的缓存添加的操作是通过Editor完成的，Editor表示一个缓存对象的编辑对象。这里仍然以图片缓存举例，首先需要获取图片url所对应的key，然后根据key就可以通过edit()来获取Editor对象。如果这个缓存正在被编辑，那么edit()会返回null，即DiskLruCache不允许同时编辑一个缓存对象。之所以要把url转换成key，是因为图片的url中很可能有特殊字符，这将影响url在Android中直接使用，一般采用rul的md5值作为key
```
 public void setLocalBitmap(String url, Bitmap bitmap) {
      DiskLruCache.Editor editor = null;
      String key = ImageUtil.hashKeyForDisk(url);
      try {
          editor = diskLruCache.edit(key);
          if (editor != null) {
              OutputStream outputStream = editor.newOutputStream(0);
              Bitmap2OutputStream(bitmap, outputStream);
              if (outputStream != null) {
                  editor.commit();
              } else {
                  editor.abort();
              }
          }
          diskLruCache.flush();
      } catch (IOException e) {
          e.printStackTrace();
      }
  }
```
**3.DiskLruCache的缓存查找**<br/>
和缓存的添加过程类似，缓存查找过程也需要将url转换为key，然后通过DiskLruCache的get方法得到一个Snapshot对象，接着再通过Snapshot对象即可得到缓存的文件输入流，有了文件输出流，自然就可以得到Bitmap对象了。为了避免加载图片过程中导致的OOM问题，一般不建议直接加载原始图片。可以通过文件流来得到它所对应的文件描述符，然后通过BitmaFactory.decodeFileDescriptor方法来加载一个缩放后的图片。
```
  public Bitmap getLocalBitmap(String url) {
        Bitmap bitmap = null;
        String key = ImageUtil.hashKeyForDisk(url);
        try {
            DiskLruCache.Snapshot snapshot = diskLruCache.get(key);
            FileInputStream fileStream = (FileInputStream)snapshot.getInputStream(0);
            FileDescriptor descriptor = fileStream.getFD();
            bitmap = BitmapFactory.decodeFileDescriptor(descriptor);
            /*if (snapshot != null) {
                bitmap = BitmapFactory.decodeStream(snapshot.getInputStream(0));
            }*/
        } catch (IOException e) {
            e.printStackTrace();
        }
        return bitmap;
    }
```
