# day4 model

## 一、配置数据库

**settings.py**

数据库默认为sqlite数据库  更改成mysql数库

**实例:**

settings.py 77行

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'djangopython1807',
        'USER':'root',
        'PASSWORD':'123456',
        'HOST':'127.0.0.1',
        'PORT':3306,
    }
}
```

helloworld/init.py文件

```python
import pymysql
pymysql.install_as_MySQLdb()
```

## 二、ORM

**概述：**

对象->关系->映射

**任务：**

1. 根据对象的类型生成表结构
2. 将对象、列表的操作 转换为sql语句
3. sql语句查询到的结果转换为对象和列表

**优点：**

极大的减轻了开发人员的工作量  不会因为数据库的改变而修改代码



## 三、模型的字段和可选条件

#### (1) 字段类型

| 字段名称         | 字段说明                                                     | 参数                                                         |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| AutoField        | 一个根据实际ID自动增长的Integer field 通常不指定(自动创建主键id字段) |                                                              |
| CharField        | varchar类型字段                                              | max_length存储值的最大长度                                   |
| TextField        | longtext类型 长文本                                          |                                                              |
| IntegerField     | int 类型字段  存储整形                                       |                                                              |
| DecimailField    | 存储浮点形 更加精准（存钱）                                  | max_digits=None 位数长度,decimal_places=None 小数的位数      |
| FloatField       | 浮点类型                                                     |                                                              |
| BooleanField     | 存储Bool值 True/False                                        |                                                              |
| NullBolleanField | 存储 null/True/False                                         |                                                              |
| DateField        | date字段                                                     | auto_now = False 如果对数据进行修改则会自动保存修改的时间  auto_now_add=False 会自动添加第一次保存的时间  **俩个参数不能同时设置** |
| TimeField        | time字段                                                     | 参数同上                                                     |
| DateTimeField    | datetimefield                                                | 参数同上                                                     |

#### (2) 字段选项

| 可选参数    | 参数说明                                           |
| ----------- | -------------------------------------------------- |
| null        | 如果设置为True 则当前字段值可以为null              |
| blank       | 如果设置为True  则当前字段可以为空（什么值都没有） |
| db_column   | 设置字段名称  不设置 字段名称默认为属性名          |
| db_index    | 常规索引                                           |
| unique      | 唯一索引                                           |
| primary_key | 主键索引                                           |
| default     | 默认值                                             |



## 四、定义模型

#### (1) 模型、属性、表之间的关联

一个模型类 对应数据库中的一张表  一个类属性 对应 表中的一个字段

#### (2) 创建我们测试的模型

models.py

```python
from django.db import models

# Create your models here.

class Test(models.Model):
    char = models.CharField(max_length=20,default='char的默认值',db_index=True)
    text = models.TextField(null=True)
    intger = models.IntegerField(blank=True,db_column='inte')
    deci = models.DecimalField(max_digits=5,decimal_places=2)
    float = models.FloatField()
    bool = models.BooleanField()
    null = models.NullBooleanField()
    date = models.DateField(auto_now=True)
    time = models.TimeField(auto_now=True)
    dateTime = models.DateTimeField(auto_now_add=True)
```

#### (3) 元选项

在模型类中 定义一个Meta类

```python
class Test(models.Model):
    ...
    class Meta:
        db_table = '表名称' #表名默认为 应用名称_类名的小写（App_test）
        #ordering = ['id'] #查询出来的数据 按照id升序查询
        #ordering = ['-id'] #查询出来的数据 按照id降序查询
```

**注意：**

如果是表已经创建完成 在次更改表名 需要重新生成迁移文件 执行迁移文件

**原生sql改表名:** alter table a rename b;

#### (4) 创建表

生成迁移文件

```python
python3 manage.py makemigrations
```

执行迁移文件

```python
python3 manage.py migrate
```



## 五、测试数据库

#### (1) 进入到python shell进行测试

python3 manage.py shell

#### (2) 倒包

```python
from App.models import Test
```

#### (3) 添加数据

```python
t = Test()
t.char = 'char'
t.text = 'text'
t.intger = 1
t.deci = 12
t.float = 12
t.bool = True
t.null = None
t.save()
t = Test(char='',text=''...) #也可以使用关键字参数添加数据
```

#### (4) 查询数据

```python
#查询数据
def showData(req):
    t = Test.objects.get(pk=1) #id=1
    print(t.char)
    print(t.text)
    return HttpResponse('查询数据')
```

#### (5) 修改

```python
#修改数据
def update(req):
    t = Test.objects.get(pk=1) #id=1
    t.char = '修改后的内容'
    t.save()
    return HttpResponse('修改数据')
```

#### (6) 删除

```python
#删除数据
def delete(req):
    t = Test.objects.get(pk=2)
    t.delete()
    return HttpResponse('删除数据')
```



## 六、模型成员

#### 类属性

#### (1) objects

是Manager类的一个对象 作用是与数据库进行交互

当定义模型的时候 没有指定模型管理器 则Django会默认为当前模型类创建一个名为objects的管理器

#### (2) 自定义模型管理器

```python
class User(models.Model):
    ...
    userobj = models.Manager() #自定义一个userobj的模型管理器 objects默认的就不存在了
```

**views.py**中使用

```python
#查询数据
def showData(req):
    u = User.userobj.get(pk=1)
    u = User.objects.get(pk=1) #报错 属性错误 不存在
    print(u)
    return HttpResponse('查询数据')
```

#### (3) 自定义管理器 Manager类

模型管理器是Django模型与数据库进行数据交互的接口 一个模型可以有多个模型管理器

**作用：**

1. 像管理器类中添加额外的方法
2. 修改管理器返回的原始查询集
3. 重写 get_queryset()方法

**实例：**

```python
#重写get_queryset方法 修改原始查询集
class UserManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(isDelete=False)
    
#创建用户表
class User(models.Model):
    ...
    objects = UserManager()
```

**使用**

```python
#查询数据
def showData(req):
    u = User.objects.all()
    return HttpResponse('查询数据')
```



#### (4) 创建对象

**目的：**

像数据库中添加数据

**注意：** 不能够在自定义的模型中 使用init构造方法 原因是已经在父类中models.Model中使用了

**1) 在模型类中添加一个方法**

**实例**

**models.py**

```python
class User(models.Model):
    ...
    #自定义添加数据的类方法 调用方法为 User.addUser([args...]).save()
    @classmethod
    def addUser(cls,username='',sex=True,age=20,info='info'):
        obj = cls(username=username,sex=sex,age=age,info=info)
        return obj
```

**views.py**使用

```python
def addUser(req):
    username = random.randrange(1,100000)
    age = random.randint(1,100)
    sex = [True,False]
    #使用自定义类方法 进行数据的添加
    u = User.addUser(username,sex[random.randint(0,1)],age)
    u.save()
    return HttpResponse('添加user模型的数据')
```

**2) 在自定义管理器中添加方法**

**models.py**

```python
#重写get_queryset方法 修改原始查询集
class UserManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(isDelete=False)
    def addUser(self,username='',sex=True,age=20,info='info'):
        # u = self.model() #代表当前调用的模型
        u = User()
        u.username = username
        u.sex = sex
        u.age = age
        u.info = info
        return u
```

user模型中

```python
class User(models.Model):
    ...
    objects = UserManager()
```

**views.py**中使用

```python
def addUser(req):
    username = random.randrange(1,100000)
    age = random.randint(1,100)
    sex = [True,False]
    u = User.objects.addUser(username,sex[random.randint(0,1)],age)
    u.save()
    return HttpResponse('添加user模型的数据')
```



## 七、模型查询

**概述：**

1. 查询集表示从数据库拿到的对象的集合
2. 查询集可以有多个过滤器
3. 过滤器就是一个函数 根据所给的参数 限制返回的查询集
4. 从sql角度来说  查询集合和select语句等价 过滤器就是sql的where条件

#### (1) all() 返回查询集中的所有数据  

类名.objects.all()

返回 query_set 查询集

```python
#all 返回所有
def all(req):
    u = User.userobj.all()
    print(u) 
    return render(req,'show.html',{'data':u})
```

**实现分页**

```python
#all 返回所有
def all(req):
    try:
        page = int(req.GET.get('page'))
    except:
        page = 1ss
    u = User.userobj.all()[(page-1)*5:page*5] #0:5  5 10
    print(u)
    return render(req,'show.html',{'data':u})
```

#### (2) filter()  将符合条件的数据进行返回

类名.objects.filter(属性名=属性值....)

如果条件多个，逗号隔开为and模式

and操作

类名.objects.filter(属性名=属性值，属性名=值) == 类名.objects.filter(属性名=属性值).filter(属性名=属性值)

**实例：**

```python
def filter(req):
    u = User.userobj.filter(sex=True,username__contains='1')
    u = User.userobj.filter(sex=True).filter(username__contains='1')
    return render(req,'show.html',{'data':u})
```

#### (3) exclude() 过滤掉符合条件的数据

类名.objects.exclude(属性名=属性值....)

```python
def filter(req):
    u = User.userobj.exclude(sex=True,username__contains='1')
    u = User.userobj.exclude(sex=True).exclude(username__contains='1')
    return render(req,'show.html',{'data':u})
```

#### (4) order_by() 排序

1. order_by('id')
2. order_by('-id')

**实例：**

```python
def orderBy(req):
    u = User.userobj.filter().order_by('-id')
    u = User.userobj.order_by('-id')
    u = User.userobj.order_by('id')
    return render(req,'show.html',{'data':u})
```

#### (5) reverse() 反转

对order_by的反转

````python
def reverse(req):
    u = User.userobj.order_by('id').reverse()
    u = User.userobj.order_by('-id').reverse()
    u = User.userobj.order_by('-age').reverse()
    return render(req,'show.html',{'data':u})
````

#### (6) values()  返回一个列表 每条数据是一个字典

```python
def values(req):
    u = User.userobj.values() #默认返回所有
    u = User.userobj.values('id','age') #只返回id和age字段的值
    print(u) #[{'username':'张三','age':18}]
    return render(req,'show.html',{'data':u})
```

#### (7) value_list()  得到一个元组格式的数据  只有值

````python
def value_list(req):
    u = User.userobj.values_list() #默认返回所有
    u = User.userobj.values_list('id','age')#只返回id和age字段的值
    print(u)
    return render(req,'show.html',{'data':u})
````

### 返回一条数据（1个对象）

#### (1) get() 返回一个对象

**注意：**

1. 只能匹配一条数据 如果多条则抛出 MultipleObjectsReturned 异常
2. 如果匹配数据失败 则抛出 DoesNotExist 异常
3. 只能匹配一条数据

**实例：**

```python
def myGet(req):
    u = User.userobj.get(pk=1)
    print(u)
    u = User.userobj.get(id=1)
    print(u)
    u = User.userobj.get(age=43) #报错 原因：只能返回一个值 但是匹配到了多个 MultipleObjectsReturned
    u = User.userobj.get(id=20) #报错 DoesNotExist 匹配失败
    print(u)
    return HttpResponse('get')
```

#### (2) count() 返回统计条数

```python
#返回数据条数
def count(req):
    u = User.userobj.count()
    u = User.userobj.filter(sex=True).count()
    print(u)
    return HttpResponse('返回数据的条数')
```

#### (3) first()  取出第一条数据

```python
def first(req):
    u = User.userobj.order_by('id').first() #第一条
    print(u)
    return HttpResponse('first')
```

#### (4) last() 取出最后一条数据

```python
def first(req):
    u = User.userobj.order_by('id').last() #最后一条
    print(u)
    return HttpResponse('first')
```

#### (5) exists() 判断数据是否存在 返回bool值

```python
def exists(req):
    u = User.userobj.filter(age=400).exists()
    print(u)
    return HttpResponse('exists')
```

### 比较运算符

#### (1) 完全匹配运算符

1. __exact   对大小写敏感
2. __iexact  对大小写不敏感

**实例：**

```python
def exact(req):
    u = User.userobj.filter(username__exact='abc')
    u = User.userobj.filter(username__iexact='abc')
    u = User.userobj.filter(username='abc')
    return render(req,'show.html',{'data':u})
```

#### (2) _contains  包含 大小写敏感(真敏感)

```python
def contains(req):
    #contains
    # u = User.userobj.filter(username__contains='a')
    u = User.userobj.filter(username__contains='A')
    return render(req,'show.html',{'data':u})
```

#### (3) \_\_startswith \_\_endswith 以...开头 以...结尾 区分大小写

 \_\_istartswith 不区分大小写的 以...作为开头

\_\_iendswith   不区分大小写 以...作为结尾

**实例：**

```python
#以...作为开头和结尾
def startend(req):
    u = User.objects.all() #查所有
    u = User.objects.filter(username__startswith='3')
    u = User.objects.filter(username__startswith='a')
    u = User.objects.filter(username__startswith='A')
    u = User.objects.filter(username__istartswith='a')
    u = User.objects.filter(username__iendswith='C')
    u = User.objects.filter(username__endswith='9')
    u = User.objects.filter(username__startswith='5',username__endswith='5')
    return render(req,'show.html',{'data':u})
```

#### (4) null的数据的查询

\_\_isnull  值为 True/False

**实例**

```python
#查询为空的
def null(req):
    #查询为null的数据
    u = User.objects.filter(username__isnull=True)
    u = User.objects.filter(username=None)
    #查询不为null的数据
    u = User.objects.exclude(username__isnull=True)
    u = User.objects.filter(username__isnull=False)
    return render(req,'show.html',{'data':u})
```

#### (5) in 是否在...里

\_\_in = [值1,值2...]

**实例**

```python
def In(req):
    #在...范围内
    u = User.objects.filter(id__in=[1,2,3,4])
    u = User.objects.filter(pk__in=[1,2,3,4])
    #不在...范围内
    u = User.objects.exclude(pk__in=[1,2,3,4])
    return render(req,'show.html',{'data':u})
```

#### (6) range 值的范围

\_\_range = [start,end]

```python
#range的使用
def Range(req):
    u = User.objects.filter(age__range=[20,40]) #between 20 and 40
    u = User.objects.filter(age__range=[20,33]) #between 20 and 33
    return render(req,'show.html',{'data':u})
```

#### (7) 比较运算符

1. \_\_gt  大于
2. \_\_gte 大于等于
3. \_\_lt 小于
4. \_\_lte  小于等于

**实例**

```python
#比较运算符
def compare(req):
    u = User.objects.filter(id__gt=2)
    u = User.objects.filter(id__gte=2)
    u = User.objects.filter(id__lt=5)
    u = User.objects.filter(id__lte=5)
    u = User.objects.filter(id__exact=5)
    u = User.objects.exclude(id__exact=5)
    return render(req,'show.html',{'data':u})
```

### **聚合函数**

**导入：**

`from django.db.models import Avg,Max,Min,Sum,Count

#### (1) Avg 平均数

```python
#聚合函数
def jh(req):
    u = User.objects.aggregate(Avg('age'))
    return HttpResponse('值为{}'.format(u))
```

#### (2) Count 统计

#### (3) Max 最大

```python
from django.db.models import Avg,Count,Sum,Max,Min
#聚合函数
def jh(req):
    u = User.objects.aggregate(Avg('age'))
    u = User.objects.aggregate(Sum('age'))
    u = User.objects.aggregate(Max('age'))
    return HttpResponse('值为{}'.format(u))
```

#### (4) Min 最小

#### (5) Sum 求和

#### Q对象

作为or查询来使用

```python
def q(req):
    u = User.objects.filter(Q(age__gt=20)) #这个没有and和or一说 因为就一个条件
    u = User.objects.filter(Q(age__gt=20)|Q(sex=True)) #查询age大于20 或 性别为True的
    # print(u)
    # return HttpResponse('q')
    # return HttpResponse('值为{}'.format(u))
    return render(req,'show.html',{'data':u})
```

#### F对象

**作用：**  可以使用模型A的属性 与 模型B的属性 进行比较

**实例：**

```python
def f(req):
    u = User.objects.filter(id__gte=F('age'))
    print(u)
    return HttpResponse('f')
```



## 八、数据的修改

1. save()
2. update()

**实例：**

##### save值的修改

```python
def update(req):
    u = User.objects.get(pk=11)
    u.username = 'zhangsan'
    u.save()
    return HttpResponse('save修改')
```

#### update的值的修改

```python
def update(req):
    u = User.objects.filter(id__in = [1,2,3])
    u.update(sex=True)
    return HttpResponse('update修改多个值')
```

**区别：**

save适用于单个对象的值的修改 

update 适用于多条数据的值的修改



## 九、模型对应关系

1. 1:1
2. 1:N
3. M:N

**关系：**

**1:1和1:N共同使用的属性**

on_delete:

+ models.CASCADE 默认值 当主表的数据删除 则从表数据 默认删除
+ models.PROTECT 保护模式  一但被删除 从表数据为保护模式 不删除
+ models.SET_NULL 当主表数据被删除 则从表外键的字段的值 设置为null 一定将这个字段 设置为null=True

#### (1) 1对1

使用OneToOneField创建1对1的模型关系 将要创建对应关系的模型添加OneToOneField

**创建1:1模型**

```python
#创建用户表
class User(models.Model):
    username = models.CharField(max_length=20,db_index=True,null=True)
    sex = models.BooleanField(default=True)
    age = models.IntegerField(default=20)
    info = models.CharField(max_length=100,default='info')
    icon = models.CharField(max_length=60,default='default.jpg')
    isDelete = models.BooleanField(default=False)
    createTime = models.DateTimeField(auto_now_add=True)
    def __str__(self):
        return self.username
    class Meta:
        db_table = 'user'


#1对1的表关系
class IdCard(models.Model):
    num = models.CharField(max_length=18)
    name = models.CharField(max_length=8)
    sex = models.BooleanField(default=True)
    birth = models.DateTimeField(auto_now_add=True)
    address = models.CharField(max_length=100,default='xxxxxx')
    # user = models.OneToOneField(User) #1对1的外键 默认删除
    # user = models.OneToOneField(User,on_delete=models.PROTECT) #保护模式 如果删除主表中与从表对应关系的数据 则不能删除
    user = models.OneToOneField(User,on_delete=models.SET_NULL,null=True) #保护模式
    def __str__(self):
        return self.name
    class Meta:
        db_table = 'idcard'
```

#### 1对1数据的添加

```python
#添加用户信息
def adduser(req):
    u = User.addUser('王五')
    u.save()
    return HttpResponse('添加用户信息')

#添加1对1的idcard数据
def addIdCard(req):
    u = User.objects.get(pk=2)
    idcard = IdCard(num=random.randrange(1000000000,10000000000),name='李四',user=u)
    idcard.save()
    return HttpResponse('添加卡的信息')
```

#### 1对1数据的查询

```python
#数据查询
def showOneToOne(req):
    idcard = IdCard.objects.first()
    print(idcard.user)  #查询卡对应用户的对象
    print(idcard.user.icon) #查询卡对应用户的对象的icon属性
    u = User.objects.first()
    print(u.idcard) #idcard关联的数据
    print(u.idcard.num) #取出idcard对象的num属性
    return HttpResponse('1对1数据的查询')
```

#### 1对1数据的删除

```python
#数据的删除
def deleteOneToOne(req):
    #删除主表数据   从表数据随着主表数据而删除
    # u = User.objects.first()
    u = User.objects.get(pk=2)
    u.delete()
    #删除从表数据  主表数据 没有改变
    # idcard = IdCard.objects.first()
    # idcard.delete()
    return HttpResponse('删除了第一个用户的数据')
```

#### (2) 1对多

使用ForeignKey创建1对1的模型关系 将要创建对应关系的模型添加ForeignKey

也就是主表和从表的关系  将外键添加到从表

**创建模型 grade和students**

```python
#班级表
class Grade(models.Model):
    gname = models.CharField(max_length=15)
    gnum = models.IntegerField()
    ggirlnum = models.IntegerField()
    gboynum = models.IntegerField()
    class Meta:
        db_table = 'grade'

#学生表
class Students(models.Model):
    sname = models.CharField(max_length=10)
    ssex = models.BooleanField(default=True)
    sage = models.IntegerField(default=20)
    sgrade = models.ForeignKey(Grade) #1对多的外键 默认删除
    # sgrade = models.ForeignKey(Grade,on_delete=models.PROTECT) #保护模式 如果删除主表中与从表对应关系的数据 则不能删除
    # sgrade = models.ForeignKey(Grade,on_delete=models.SET_NULL, null=True)  # 保护模式
    def __str__(self):
        return self.sname
    class Meta:
        db_table = 'students'
```

**添加数据**

```python
#添加grade信息
def addGrade(req):
    Grade(gname='python1807',gnum=63,ggirlnum=6,gboynum=57).save()
    Grade(gname='python1808',gnum=50,ggirlnum=2,gboynum=48).save()
    return HttpResponse('添加grade信息')
#添加students信息
def addStudents(req):
    Students(sname='梁义',sgrade=Grade.objects.get(pk=1)).save()
    Students(sname='贾江龙',sgrade=Grade.objects.get(pk=1)).save()
    Students(sname='尚子义',sgrade=Grade.objects.get(pk=2)).save()
    Students(sname='王鹏飞',sgrade=Grade.objects.get(pk=2)).save()
    return HttpResponse('添加students信息')
```

**查询**

```python
#1对多查询
def oneToManyShow(req):
    g = Grade.objects.get(pk=1)
    s = g.students_set.all()
    for i in s:
        print(i)
    return HttpResponse('查询1对多')
```

**删除**

```python
#删除主表数据  默认还是 主表数据删除 从表对应数据也被删除
def deleteGrade(req):
    g = Grade.objects.first()
    name = g.gname
    g.delete()
    return HttpResponse('删除主表数据{}'.format(name))
```

#### (3) 多对多

使用ManyToManyField创建1对1的模型关系 将要创建对应关系的模型添加ManyToManyField

**创建模型**

```python
#创建用户表
class User(models.Model):
    username = models.CharField(max_length=20,db_index=True,null=True)
    sex = models.BooleanField(default=True)
    age = models.IntegerField(default=20)
    info = models.CharField(max_length=100,default='info')
    icon = models.CharField(max_length=60,default='default.jpg')
    isDelete = models.BooleanField(default=False)
    createTime = models.DateTimeField(auto_now_add=True)
    def __str__(self):
        return self.username
    class Meta:
        db_table = 'user'
        
#以下为多对多  posts关联用户
class Posts(models.Model):
    title = models.CharField(max_length=20,default='标题')
    article = models.CharField(max_length=200,default='article')
    createtime = models.DateTimeField(auto_now=True)
    users = models.ManyToManyField(User)
    def __str__(self):
        return self.title
    class Meta:
        db_table = 'posts'
```

#### 添加数据

**正常添加**

```python
#添加grade信息
def addPosts(req):
    Posts(title='以后的你感谢现在拼搏的自己').save()
    Posts(title='阿甘正传').save()
    Posts(title='斗罗大陆').save()
    return HttpResponse('添加Posts信息')
```

**添加多对多的数据** add

**添加一个多对多的数据**

```python
#添加收藏
def addColl(req):
    u1 = User.objects.first()
    p1 = Posts.objects.first()
    p1.users.add(u1)
    return HttpResponse('添加多对多的数据')
```

**添加多个多对多的数据**

```python
#添加收藏
def addColl(req):
    u1 = User.objects.first()
    u2 = User.objects.last()
    p = Posts.objects.get(pk=2)
    p.users.add(u1,u2) #添加多个对象
    return HttpResponse('添加多对多的数据')
```

**查询**

```python
#查询 用户收藏了哪些帖子
def showPosts(req):
    u = User.objects.first()
    data = u.posts_set.all()
    for i in data:
        print(i)
    return HttpResponse('查询收藏了哪些帖子')


#查看帖子对应多的关系
def showUsers(req):
    p1 = Posts.objects.first()
    data = p1.users.all()
    for i in data:
        print(i)
    return HttpResponse('查询帖子被哪个用户收藏了')
```

**删除** remove

##### remove1条数据

```python
#删除
def deleteData(req):
    p1 = Posts.objects.first()
    u1 = User.objects.first()
    p1.users.remove(u1)
    return HttpResponse('1号用户取消收藏1号帖子')
```

remove多条数据

```python
#删除
def deleteData(req):
    p2 = Posts.objects.get(pk=2)
    u1 = User.objects.first()
    u2 = User.objects.last()
    p2.users.remove(u1,u2)
    return HttpResponse('1号用户取消收藏1号帖子')
```





  























