---
layout:     post
title:      Android数据库神器之Room
subtitle:   Room的使用
date:       2018-08-07
author:     Anriku
header-img: img/2018_08_07_post.jpeg
catalog: true
tags:
    - Android
    - Android架构
    - Jetpack
    - Room
---

我们都知道原生的Android数据库API使用起来很恶心。特别是查询的参数是相信大家都觉得头疼的事情。那么今天的IO大会Google给Android开发带来了一件数据库神器，让我们的数据更加简单、更加清晰、使用更加容易。那么个容易法呢？通过这篇博客的例子就让它体验一下。



# Room概述

首先要使用基本的Room引入如下依赖：

```
implementation 'androidx.room:room-runtime:2.0.0-rc01'
kapt "androidx.room:room-compiler:2.0.0-rc01" //Java使用annotationProcessor替代kapt
```

至于androidx相关的内容已经在前面的博客中讲到了，这里就不重复了。



#### Database

DataBase是一个数据库的容器。用**类级@Database**来进行标明。

用@DataBase表明的类具有下面三个特点：

* 这个类必须是一个继承自RoomDataBase的抽象类
* 在@DataBase注解中有许多的实体(也就是表)
* 在这个抽象类中通过一系列的抽象方法来返回后面会进行说明的@Dao修饰的类



当我们对@DataBase修饰的数据库类进行实例化的时候有两个Builder。它们的区别如下

* Room.databaseBuilder():这个方法是真正的**本地化**的数据库
* Room.inMemoryDatabaseBuilder():这个方法是用于创建存储在**内存中**的数据库。当前进程被杀掉就不复存在了。



#### Entity

Entity是用来表示一个表的



#### DAO(Data Access Object)

DAO是包含一系列的用于进行表操作的方法



# Entity的使用



Entity是用来代表一个表的：

直接来看个例子吧：

```kotlin
@Entity(tableName = "student")
class Student(@PrimaryKey 
              @ColumnInfo
              (name = "stu_id") var stuId: String,
              @ColumnInfo(name = "stu_name") 
              var stuName: String?,
              @ColumnInfo(name = "stu_sex") 
              var stuSex: String?) 
```

上面用@Entity来表示这个类是一个表。

在Entity中通过@Primary来指定主键，主键也可以在@Entity中进行指定。

在Entity中通过@ColumnInfo来对表中的列进行设置。如果无任何设置，还可以将@ColumnInfo注解去掉。默认的列名是类中的属性名。如果你不想让类中的属性存入数据库中，你可以使用@Ignore注解来进行忽略。



### 索引创建

数据库中通过创建索引可以加快以本列为条件的查询的速度。

**索引的创建有两种方式：**

* 在Room中通过在@Entity中进行索引的添加
* 在@ColumnInfo中进行定义

```kotlin
@Entity(tableName = "student", indices = [(Index(value = ["stu_name"]))])
...

or
...
class Student(...
              @ColumnInfo(name = "stu_name", index = true) 
              ...)

```



#### 外键的添加

外键用于一个表建立于另一个表的关系。

这里要有两个概念要弄清楚，**一个是parent表表示被关联的表(也就是主键被关联的表这里是Student表)，一个是child表指定我们定义外键的表**。

**child表的外键必须是要在另一个被关联的parent表的主键的取值范围内的。**

下面来个例子：

```kotlin
@Entity(foreignKeys =
[(ForeignKey(entity = Student::class,parentColumns = ["stu_id"],childColumns = ["student_id"],
        onDelete = ForeignKey.CASCADE))])
class Book(
        @PrimaryKey
        @ColumnInfo(name = "book_name")
        var bookName: String,
        @ColumnInfo(name = "book_price")
        var bookPrice: Double,
        @ColumnInfo(name = "student_id", index = true)
        var studentId: String
)
```



其中Student为了稍微合理一点可以做如下个修改：

```kotlin
@Entity(tableName = "student", indices = [(Index(value = ["stu_name"]))],
        foreignKeys = [ForeignKey(entity = Book::class, parentColumns = ["book_name"], childColumns = ["b_name"])])
class Student(...
              @ColumnInfo(name = "b_name",index = true)
              var bookName: String)
```





从上面可以知道。**外键的定义是放在@Entity中的，这也只能在这里面进行定义。**

entity用于指定**parent表**。

然后，parentColumns和childColumns用于指定两表关联的属性。



两个表通过外键连接有下面几种方式：

* NO_ACTION: parent表中某行被删掉(更新)后。child表中与parent这一行发生映射的行不发生任何改变
* RESTRICT: parent表中想要删除(更新)某行。如果child表中有与这一行发生映射的行。那么改操作拒绝。
* SET_NULL/SET_DEFAULT:parent表中某行被删掉(更新)后。child表中与parent这一行发生映射的行设置为NULL(DEFAULT)值。
* CASCADE:parent表中某行被删掉(更新)后。child表中与parent这一行发生映射的行被删掉(其属性更新到对应设置)



外键在很多情况是有用的。比如上面的如果Book表中的某一本书被删掉了。那么通过onDelete的CASCADE方式可以将对应借了这本书的Student表中的对应记录删掉。



#### 设置嵌套对象

被嵌套类：

```kotlin
class BirthDay(
        @ColumnInfo(name = "birth_day")
        var birthYear: Int,
        @ColumnInfo(name = "birth_month")
        var birthMonth: Int,
        @ColumnInfo(name = "birth_day")
        var birthDay: Int)
```

Student表中的修改：

```kotlin
...
class Student(...
              @Embedded
              var birthDay: BirthDay)
```

通过@Embedded可以使用一个对象作为表的一个属性。**但是在数据库中会把BirthDay类中所有属性集成到Student表中。**



# DAO

DAO是一个用于进行数据库的CRUD操作的。**它可以是一个接口或者是一个抽象类。**

它们的区别在于：

* 接口很简单，就需要定义操作相关的方法就行
* 如果是抽象类的话，你可以给一个RoomDatabase类型参数的构造器



下面看一个例子：

```kotlin
@Dao
interface StudentDao {

    @Insert
    fun insertStudent(student: Student)

    @Query("SELECT * FROM student")
    fun queryAll(): List<Student>

    @Query("SELECT * FROM student WHERE stu_id = :stuId")
    fun queryByStuId(stuId: String)

    @Delete
    fun deleteStudent(user: User)

    @Update
    fun updateUser(user: User)
    
}
```

这里用@Dao注解来进行DAO接口的标注。这个注解没有任何的参数。只是纯粹的用来进行一个标志的。



然后其中的CRUD操作分别通过@Insert、@Query、@Delete、@Delete、@Update注解来进行标注。根据上面的例子来分别对这几个注解进行一个具体的说明：



#### @Insert的使用

@Insert的使用很简单就是用来标志接口中的方法是用来进行数据插入的。它里面有一个onConflict参数。用于指明当插入的数据与已有的数据发生了冲突之后该怎么办。



#### @Delete的使用

**@Delete修饰的方法的参数要么是用@Entity修饰的类的、要么是这个样的类的集合/数组。**

而且还有一个需要注意的是@Delete对没有定义主键为自增长的类是不管用的。对于主键是自增长的，可以改@Query注解来写sql语句进行删除。



#### @Update的使用

@Update的使用其实和@Delete的使用是差不多的，只不过一个是更新数据、一个是删除数据。



**上面的插入、删除、更新都是可以返回一个int类型的数据用于表示操作了表中的多少行**



#### @Query的使用

大家都知道在数据中的增、删、改、查操作中，最难同时也是最重要的操作就是查。在Room的使用中。这个仍然是重点的。也是平时大家用得最多的。



**@Query注解只有一个String参数。这个参数用来接受一个sql语句。**任何的查询语句都可以直接写sql语句了。不同像Android原生的数据库API那样。接受那么多贼烦人的操作。



##### 含有参数的sql语句

* 如果sql语句中有需要通过方法参数传递进行的参数。**如果这个参数不是集合**我们用**:variable**来进行参数的使用。这个在我们上面的例子中有体现。
* 如果这个参数是集合我们用**(:variable)**俩进行参数的使用。上面的DAO类没有对应的例子。这里举个例子：

```kotlin
@Dao
public interface MyDao {
    @Query("SELECT first_name, last_name FROM user WHERE region IN (:regions)")
    public List<NameTuple> loadUsersFromRegions(List<String> regions);
}
```



##### 返回指定的列

在上面我们的查询语句都是返回一个@Entity注解的类的List集合。那么如果我们不像返回所有列该如何？Room很人性化。**我们可以用@Entity修饰的类的属性的子集来组成一个类，然后返回这个类的集合**



##### 返回LiveData

当然Room不是单独战斗的。与我们前面的LiveData都是有沟通。在上面任何的一个查询返回的数据都是可以用LiveData来进行包装的。只不过这里返回的数据不能直接用value值来获取，只能同observe来获取值的改变。



##### 直接获取Cursor

如果我们想像原生的API那样直接获取一个表示数据的Cursor改怎么办。这时候我们可以直接让返回的值为Cursor对象就行。但是这种做法是不推荐的。除非你真的有必要使用Cursor外，其它情况不要使用Cursor。



##### 和RxJava2的使用

学Android都知道RxJava是一个很不错的开源库。那么Room也可以和RxJava一起使用



首先，要引入下面的依赖

```
implementation "android.arch.persistence.room:rxjava2:1.1.1"
```



然后，将我们查询要得到的数据通过Observable、Single、Flowable、Maybe来进行包装。这里通过Observable来进行数据的包装。

```kotlin
@Dao
interface UserDao {
...
    @Query("SELECT * FROM user")
    fun getAll(): Observable<List<User>>
...

}
```



最后，通过下面的代码我们就可以得到数据库中的数据了：

```kotlin
        Thread {
            val users = db.userDao().getAll()
            
            users.subscribeOn(Schedulers.newThread())
                    .observeOn(AndroidSchedulers.mainThread())
                    .subscribe {
                        for (i in it) {
                            Log.e("RoomActivity", i.toString())
                        }
                    }
		}.start()
```





# DataBase的使用

```kotlin
@Database(entities = [Student::class], version = 1)
abstract class StudentDatabase : RoomDatabase() {
    abstract fun studentDao(): StudentDao

    companion object {
        private var INSTANCE: StudentDatabase? = null

        fun getDatabase(context: Context): StudentDatabase? {
            if (INSTANCE == null) {
                synchronized(StudentDatabase::class) {
                    if (INSTANCE == null) {
                        INSTANCE = Room.databaseBuilder(context,
                                StudentDatabase::class.java, "student_database").build()
                    }
                }
            }
            return INSTANCE
        }
    }
}
```

Database这里用@Database注解来进行修饰，这里可以必须指定entities、version属性。entities属性接受一个数组，表示这个Database中有哪些表，version用来指定数据库版本，当Entity发生变化之后我们要进行版本升级，当然如果为了数据能够在升级后仍然能够保存下来，还需要下面讲的Migration。



上面还有一点就是通过单例模式来方式生成多个数据库对象同时对数据库进行操作。



# 使用数据库

**这里为了方便说明，使用的表示是上面讲Entity的时候最初创建的表。关于外键等内容没有添加到表中！！！**

```kotlin
class RoomActivity : AppCompatActivity() {

    @SuppressLint("CheckResult")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_room)

        Thread{
            val studentDatabase = StudentDatabase.getDatabase(applicationContext)

            studentDatabase!!.studentDao().insertStudent(Student("2016215039",
                    "anrikuwen","男"))
        }.start()

    }
}
```

上面是在Activity中进行代码的使用。**这里要注意的数据库的操作默认是在子线程中进行的**，因为当有大量数据进行读写的时候会很耗时。当然如果数据量小的话，也可以使用DatabaseBuilder的allowMainThreadQueries()来让其可以在主线程中进行读写操作。



# 自定义类型转换

在Room中，可以使用@TypeConverter和@TypeConverters来定义类型转换。通过一个例子来进行具体说明：

```kotlin
class StudentConverter {
    @TypeConverter
    fun userToStu(user: User): String? {
        return user.firstName + user.lastName
    }
}
```

从上面的代码可以看到@TypeConverter用于修饰我们要进行转换的方法。这里是将一个User对象转换为String对象。

```kotlin
@Dao
interface StudentDao {
...
    @Query("SELECT * FROM student WHERE stu_name LIKE :user")
    @TypeConverters(StudentConverter::class)
    fun queryByStu(user: User): List<Student>
}
```

这里用@TypeConverters来修饰要进行转换的部分。其中参数是一个Class数组，用于表示用于转换的类。



这是User类

```kotlin
data class User(var firstName: String, var lastName: String)
```



然后在Activity中你就可以传入User来进行按姓名查询的操作了。



@TypeConverters有下面几种修饰情况：

* 如果给Database修饰，那么Daos和Entities都可以使用
* 如果给Dao修饰，那么Dao中所有方法都能使用
* 如果给Entity修饰，那么Entity中的所有域都能使用
* 如果给Entity中的域修饰，那么只有这个域能使用
* 如果给Dao的方法使用，那么只有Dao中的这个方法能使用
* 如果给Dao中的方法参数使用，那么只有这个参数可以使用



# Migration

当数据库的Entity发生了改动后，进行版本升级是必要的。**但如果你还想将原本数据库中所有表的内容转移到新的数据库中就要通过Migration来进行转移。Migration中实现的修改表的语句**

下面看个例子：

首先，在Entity中我们添加一个属性

```kotlin
@Entity(tableName = "student")
class Student(stuId: String, stuName: String?, stuSex: String?, borrowNum: Int?,returnTime: String?) {
...
    @ColumnInfo(name = "return_time")
    var returnTime: String? = returnTime
}
```



下面我们修改一下StudentDatabase类:

```kotlin
@Database(entities = [Student::class], version = 2)
abstract class StudentDatabase : RoomDatabase() {
...

                        val migration = object :Migration(1,2){
                            override fun migrate(database: SupportSQLiteDatabase) {
                                database.execSQL("""
                                    ALTER TABLE student ADD COLUMN return_time TEXT
                                """.trimIndent())
                            }

                        }


                        INSTANCE = Room.databaseBuilder(context,
                                StudentDatabase::class.java, "student_database")
                                .addMigrations(migration)
                                .build()
 ...
}
```

这里要修改三个地方：**一个是version的升级；一个是实例化一个Migration对象里面通过写对应转移sql语句；最后在进行Database创建的时候使用addMigrations来添加Migration。**



**如果我找不到对应的可以进行数据库转移的方法。可以使用databaseBuilder的fallbackToDestructiveMigration()来进行数据的重建，这样数据库中的所有表的数据都会被删除掉**



# 总结

在这篇博客中主要是介绍了Room的使用。

主要有五点：

* Entity
* DAO
* Database
* 自定义转换
* Migration



其它更多相关的内容请查阅相关的文档。enjoy coding!!!

# 参考

[官方文档](https://developer.android.com/training/data-storage/room/)



*转载请注明链接*