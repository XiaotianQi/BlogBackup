Django Model 是以 Python 代码形式表示数据在数据库中的定义。对数据层来说，Python 代码等同于 SQL 语句，只不过执行的是 Python 代码而不是 SQL。这也是 OBM 的好处之一，只要专注 Model 即可，避免大脑在不同领域来回切换。

***

### **Django Models 简介**

模型是数据的唯一的、确定的信息源。它包含储存数据的字段和行为。Django 使用一种直观的方式把数据库表中的数据表示成 Python 对象。

* 每个模型都是 django.db.models.Model 的一个Python 子类。 父类 Model 包含了所有必要的和数据库交互的方法，并提供了一个简洁漂亮的定义数据库字段的语法。
* 一个模型类代表数据库中的一个表，一个模型类的实例代表这个数据库表中的一条特定的记录。
* 提供一个自动生成的访问数据库抽象的 API，可以创建、检索、更新和删除对象。

***

### **Model 示例**

创建 Blog 所需的三个简单要素：类别、标签和博文。模型中的每个字段都是 Field 子类的某个实例。

```python
class Category(models.Model):
    name = models.CharField(max_length=100)
    def __str__(self):
        return self.name

class Tag(models.Model):
    name = models.CharField(max_length=100)
    def __str__(self):
        return self.name

class Post(models.Model):
    title = models.CharField(max_length=70)
    body = models.TextField()
    created_time = models.DateTimeField(auto_now_add=True)
    modified_time = models.DateTimeField(auto_now=True)
    excerpt = models.CharField(max_length=200, blank=True)

    category = models.ForeignKey(Category)
    tag = models.ManyToManyField(Tag, blank=True)
    author = models.ForeignKey(User)

    # 阅读量
    views = models.PositiveIntegerField(default=0)

    def increase_views(self):
        self.views += 1
        self.save(update_fields=['views'])

    def __str__(self):
        return self.title

    # 以创建时间排序
    class Meta:
        ordering = ['-created_time']

```

***

### **对应SQL语句**

查看 MySQL CREATE TABLE 语句：

```shell
python manage.py sqlmigrate blog 0001
```

MySQL 语句如下：

```SQL
BEGIN;
--
-- Create model Category
--
CREATE TABLE "blog_category" (
    "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    "name" varchar(100) NOT NULL
    );
--
-- Create model Post
--
CREATE TABLE "blog_post" (
    "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
     "title" varchar(70) NOT NULL,
     "body" text NOT NULL,
     "created_time" datetime NOT NULL,
     "modified_time" datetime NOT NULL,
     "excerpt" varchar(200) NOT NULL,
     "author_id" integer NOT NULL REFERENCES "auth_user" ("id"),
     "category_id" integer NOT NULL REFERENCES "blog_category" ("id")
     );
--
-- Create model Tag
--
CREATE TABLE "blog_tag" (
    "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    "name" varchar(100) NOT NULL
    );
--
-- Add field tag to post
--
CREATE TABLE "blog_post_tag" (
    "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    "post_id" integer NOT NULL REFERENCES "blog_post" ("id"),
    "tag_id" integer NOT NULL REFERENCES "blog_tag" ("id")
    );
CREATE INDEX "blog_post_author_id_dd7a8485" ON "blog_post" ("author_id");
CREATE INDEX "blog_post_category_id_c326dbf8" ON "blog_post" ("category_id");
CREATE UNIQUE INDEX "blog_post_tag_post_id_tag_id_ba2a5f83_uniq" ON "blog_post_tag" ("post_id", "tag_id");
CREATE INDEX "blog_post_tag_post_id_a5c00319" ON "blog_post_tag" ("post_id");
CREATE INDEX "blog_post_tag_tag_id_2bbd31e4" ON "blog_post_tag" ("tag_id");
COMMIT;
```

由此可见，Django 把 model 中内容封装为 MySQL 事务。每个类相当于单个数据库表，每个类属性都被指定成一个字段，每个属性映射到一个数据库的列。属性名即字段名，属性的类型（例如 CharField ）相当于数据库的字段类型 （例如 varchar ）。

需要注意，我们并没有显式地为这些模型定义任何主键。如果一个模型里没有显示声明哪一个字段（field）是主键（即在某个字段里声明 primary_key=True），则 Django 自动为每个模型生成一个自增长的整数主键字段，为每个模型自动添加了一个 id 主键。除非单独声明，否则 Django 自动为每个模型生成主键字段，因为每个 Django 模型都要求有单独的主键。

***

### **null 与 blank**

文章可以没有标签 Tag，因此为 tag 指定了 blank=True。默认情况下 CharField 要求我们必须存入数据，否则就会报错。指定 CharField 的 blank=True 参数值后就可以允许空值了。

> * null，Field.null。如果为True，Django将在数据库中将空值存储为NULL。默认值是 False。如果字符串字段的null=True，那意味着对于“无数据”有两个可能的值：NULL 和空字符串。Django 的惯例是使用空字符串而不是NULL。
> * blank，Field.blank。注意它与null不同。null 纯粹是数据库范畴的概念，而blank 是数据验证范畴的。如果字段设置blank=True，表单验证时将允许输入空值。如果字段设置blank=False，则该字段为必填。
> * 无论是字符串字段还是非字符串字段，如果你希望在表单中允许空值，你将还需要设置blank=True，因为null 仅仅影响数据库存储。

### **关系**

Category 与 Tag 的类我们已经定义在上面。我们在这里把 Post 类对应的数据库表 和 Category、Tag 类对应的数据库表关联起来。但是，关联形式不同: ForeignKey，即多对一的关联关系。ManyToManyField，表明这是多对多的关联关系。

Django 提供了三种最常见的数据库关系：多对一(ForeignKey)，多对多(ManyToManyField)，一对一(OneToOneField)

* ForeignKey

> class ForeignKey(othermodel[, **options])

多对一关系。需要一个位置参数：与该模型关联的类。

* ManyToManyField

> class ManyToManyField(othermodel[, **options])

多对多关系。需要一个关键字参数：与该模型关联的类。

在幕后，Django 创建一个中间表来表示多对多关系。默认情况下，这张中间表的名称使用多对多字段的名称和包含这张表的模型的名称生成，如示例中 blog\_post\_tag。

* OneToOneField

> class OneToOneField(othermodel[, **options])

一对一关系。需要一个位置参数：与该模型关联的类。当某个对象想扩展自另一个对象时，最常用的方式就是在这个对象的主键上添加一对一关系。

***

### **update_fields**

在统计阅读量时，使用了 update\_fields，其用来指定哪些字段需要更新，其余字段不更新。默认参数是 None，这样所有字段都会更新一遍。在字段很多而我们只需要更新很少的字段情况下，可以用修改参数来提高更新效率。注意参数是一个可迭代对象（比如list等）。如果给它一个空的可迭代对象，那么就什么都不更新（注意和None不同，如果等于None是更新全部字段）。一旦 update_fields 参数不使用默认值None，那么save()语句就是强制执行UPDATE的。

在显式调用 save() 之前，Django 是不会访问数据库。

***

### **class Meta**

class Meta 内嵌于 Post 这个类的定义中，设置一些与特定模型相关的选项。设置了 ordering 选项，使用 Django 的数据库 API 去检索时，Post 对象的相关返回值默认地都会按 \-created\_time 字段排序。

***

### **QuerySet**

一旦建立好数据模型，Django 会自动为你生成一套数据库抽象的 API，可以创建、检索、更新和删除对象，这四种操作亦是数据库常用操作。

QuerySet 方法是封装了对数据库的操作。从数据库中查询出来的结果一般是一个集合，并且可迭代支持切片。可在Django Shell中操作。

创建一个 QuerySet 对象的时候，Django 并不会立即向数据库发出查询命令，只有在你需要用到这个 QuerySet 的时候才会这样做。