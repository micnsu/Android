### LitePal For Android（替代SQLite传统操作）

#### [Github地址](https://github.com/LitePalFramework/LitePal)

#### LitePal配置

1. 引用

   **jar包引用** 

   * [**litepal-1.6.0.jar**](https://github.com/LitePalFramework/LitePal/raw/master/downloads/litepal-1.6.0.jar)
   * [**litepal-1.6.0-src.jar**](https://github.com/LitePalFramework/LitePal/raw/master/downloads/litepal-1.6.0-src.jar)

   **Gradle引用(非GitHub项目最新版本)**

    > ```
    > dependencies {
    >     compile 'org.litepal.android:core:1.6.0'
    > }
    > ```

2. xml文件设置

   > ```
   > <?xml version="1.0" encoding="utf-8"?>
   > <litepal>
   >     <!--
   >     	Define the database name of your application. 
   >     	By default each database name should be end with .db. 
   >     	If you didn't name your database end with .db, 
   >     	LitePal would plus the suffix automatically for you.
   >     	For example:    
   >     	<dbname value="demo" />
   >     -->
   >     <dbname value="demo" />
   >
   >     <!--
   >     	Define the version of your database. Each time you want 
   >     	to upgrade your database, the version tag would helps.
   >     	Modify the models you defined in the mapping tag, and just 
   >     	make the version value plus one, the upgrade of database
   >     	will be processed automatically without concern.
   > 			For example:    
   >     	<version value="1" />
   >     -->
   >     <version value="1" />
   >
   >     <!--
   >     	Define your models in the list with mapping tag, LitePal will
   >     	create tables for each mapping class. The supported fields
   >     	defined in models will be mapped into columns.
   >     	For example:    
   >     	<list>
   >     		<mapping class="com.test.model.Reader" />
   >     		<mapping class="com.test.model.Magazine" />
   >     	</list>
   >     -->
   >     <list>
   >     </list>
   >     
   >     <!--
   >         Define where the .db file should be. "internal" means the .db file
   >         will be stored in the database folder of internal storage which no
   >         one can access. "external" means the .db file will be stored in the
   >         path to the directory on the primary external storage device where
   >         the application can place persistent files it owns which everyone
   >         can access. "internal" will act as default.
   >         For example:
   >         <storage value="external" />
   >     -->
   >     
   > </litepal>
   > <!-- 
   > 	dbname:数据库名字
   > 	version：数据库版本号
   > 	list：映射模型
   > 	storage：配置数据库存储位置，只有internal（内部）和external（外部）两种配置
   > -->
   > ```


3. 配置litepalApplication

   * 直接在**AndroidManifest**文件中设置

     > ```
     > <manifest>
     >     <application
     >         android:name="org.litepal.LitePalApplication"
     >         ...
     >     >
     >         ...
     >     </application>
     > </manifest>
     > ```

   * 自定义**MyApplication**

     * 首先在AndroidManifest中指定自定义class

     > ```
     > <manifest>
     >     <application
     >         android:name="com.example.MyOwnApplication"
     >         ...
     >     >
     >         ...
     >     </application>
     > </manifest>
     > ```

     * 在代码中自定义MyApplication

     > ```
     > public class MyOwnApplication extends AnotherApplication {
     >
     >     @Override
     >     public void onCreate() {
     >         super.onCreate();
     >         LitePal.initialize(this);
     >     }
     >     ...
     > }
     > ```

#### 使用

1. 建表

   > ```
   > public class Album extends DataSupport {
   > 	//id主键可以不写，默认生成
   >     @Column(unique = true, defaultValue = "unknown")//不允许重复，默认值位unknown
   >     private String name;
   > 	
   >     private float price;
   > 	
   >     private byte[] cover;
   > 	
   >     private List<Song> songs = new ArrayList<Song>();
   >
   >     // generated getters and setters.
   >     ...
   > }
   > public class Song extends DataSupport {
   > 	
   >     @Column(nullable = false)//允许为null
   >     private String name;
   > 	
   >     private int duration;
   > 	
   >     @Column(ignore = true)//允许忽略
   >     private String uselessField;
   > 	
   >     private Album album;
   >
   >     // generated getters and setters.
   >     ...
   > }
   > ```

   修改litepal.xml文件中的映射

   > ```
   > <list>
   >     <mapping class="org.litepal.litepalsample.model.Album" />
   >     <mapping class="org.litepal.litepalsample.model.Song" />
   > </list>
   > ```

2. 更改表属性

   在继承DataSupport的类中修改相关属性后，在xml文件中将数据库版本+1

3. 数据保存

   > ```
   > Album album = new Album();
   > album.setName("album");
   > album.setPrice(10.99f);
   > album.setCover(getCoverImageBytes());
   > album.save();
   > Song song1 = new Song();
   > song1.setName("song1");
   > song1.setDuration(320);
   > song1.setAlbum(album);
   > song1.save();
   > Song song2 = new Song();
   > song2.setName("song2");
   > song2.setDuration(356);
   > song2.setAlbum(album);
   > song2.save();
   > ```

4. 数据修改

   ```
   Album albumToUpdate = DataSupport.find(Album.class, 1);
   albumToUpdate.setPrice(20.99f); // raise the price
   albumToUpdate.save();
   ```

   或者

   ```
   Album albumToUpdate = new Album();
   albumToUpdate.setPrice(20.99f); // raise the price
   albumToUpdate.update(id);
   ```

   或者

   ```
   Album albumToUpdate = new Album();
   albumToUpdate.setPrice(20.99f); // raise the price
   albumToUpdate.updateAll("name = ?", "album");
   ```

5. 删除数据

   ```
   DataSupport.delete(Song.class, id);
   ```

   或者

   ```
   DataSupport.deleteAll(Song.class, "duration > ?" , "350");
   ```

6. 查找数据

   ```
   Song song = DataSupport.find(Song.class, id);
   ```

   或者

   ```
   List<Song> allSongs = DataSupport.findAll(Song.class);
   ```

   或者

   ```
   List<Song> songs = DataSupport.where("name like ?", "song%").order("duration").find(Song.class);
   ```

7. 异步查询

   ```
   DataSupport.findAllAsync(Song.class).listen(new FindMultiCallback() {
       @Override
       public <T> void onFinish(List<T> t) {
           List<Song> allSongs = (List<Song>) t;
       }
   });
   ```

8. 支持代码生成数据库

   ```
   LitePalDB litePalDB = new LitePalDB("demo2", 1);
   litePalDB.addClassName(Singer.class.getName());
   litePalDB.addClassName(Album.class.getName());
   litePalDB.addClassName(Song.class.getName());
   LitePal.use(litePalDB);
   ```



#### 后记

以上总结并不完全，还有很多其他操作没有总结，作者个人CSDN博客更全——[作者CSDN](http://blog.csdn.net/guolin_blog/article/details/38556989)

