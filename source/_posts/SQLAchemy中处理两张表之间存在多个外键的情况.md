---
title: SQLAchemy中处理两张表之间存在多个外键的情况
date: 2019-01-24 23:51:17
categories: Flask
tags:
    - Flask
    - Flask-SQLAchemy
    - SQLAchemy
---
在Flask的开发中，我们势必会遇到两张表之间存在多个外键的情况。例如，现在有两张表，一张表是`User`，另一张表是`Article`。一篇文章的作者`author_id`可以设置外键关联`User`表，同时文章的审稿人`reviewer_id`也可以设置外键关联`User`表。当我们以SQLAchemy多对一(many to one)的设计方法来添加`relationship`关系映射时，程序会抛出一个`AmbiguousForeignKeysError`错误，这篇文章我们就来解决这个问题。
<!--more-->
### 出现Error的代码写法 
先来看以SQLAchemy多对一的常规设计方法处理这个问题时我的代码写法。  

```python
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)  
    email = db.Column(db.String(64), nullable=False, unique=True)  
    name = db.Column(db.String(32), nullable=False)  
    ...  
    
class Article(db.Model):
    __tablename__ = 'articles'
    id = db.Column(db.Integer, primary_key=True)  
    title = db.Column(db.String(64), nullable=False) 
    author_id = db.Column(db.Integer, db.ForeignKey("users.id"))
    reviewer_id = db.Column(db.Integer, db.ForeignKey("users.id"))
    author = db.relationship('User', backref='articles')  
    reviewer = db.relationship('User', backref='review_articles')  
```


看起来程序的设计应该是没问题，可运行的结果真的跟我们预想的一样吗？  
当我们运行代码后，程序抛出了一个错误：  

```
sqlalchemy.exc.AmbiguousForeignKeysError: 
Could not determine join condition between parent/child tables on relationship Article.author 
- there are multiple foreign key paths linking the tables.  
Specify the 'foreign_keys' argument, providing a list of those columns 
which should be counted as containing a foreign key reference to the parent table. 
```

可以看到SQLAchemy提示无法确定`Article.author`的父子表之间的关联，原因在于两张表之间存在多个外键。需要我们指定`foreign_keys`参数，提供一个包含关联了父表（即`User`表）外键的字段列表（`list`)  

### 解决办法  

查询了很多博客资料后这个问题依旧没有得到解决，只好去阅读[SQLAchemy的官方文档](https://docs.sqlalchemy.org/en/latest/index.html)，在SQLAchemy ORM > Relationship Configuation > Configuring how Relationship Joins下有关于[Handling Multiple Join Paths](https://docs.sqlalchemy.org/en/latest/orm/join_conditions.html#handling-multiple-join-paths)的介绍。  

文档中说，在遇到两表之间存在多外键关联时，需要给`relationship()`指定`foreign_keys`参数。需要对我们的代码进行修改，添加`foreign_keys`参数，所以将代码修改为：  

```python
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)  
    email = db.Column(db.String(64), nullable=False, unique=True)  
    name = db.Column(db.String(32), nullable=False)  
    ...

    def __repr__(self):
        return '<User id={0}, name={1}>'.format(self.id, self.name)  
    
class Article(db.Model):
    __tablename__ = 'articles'
    id = db.Column(db.Integer, primary_key=True)  
    title = db.Column(db.String(64), nullable=False) 
    author_id = db.Column(db.Integer, db.ForeignKey("users.id"))
    reviewer_id = db.Column(db.Integer, db.ForeignKey("users.id"))
    author = db.relationship('User', backref='articles', foreign_keys=[author_id])  
    reviewer = db.relationship('User', backref='review_articles', foreign_keys=[reviewer_id])

    def __repr__(self):
        return '<Article id={0}, title={1}>'.format(self.id, self.title)     
```


同时，在指定`foreign_keys`时，我们也可以使用字符串来指定。但如果使用列表，则列表必须是字符串的一部分。  

```python
author = db.relationship('User', backref='articles', foreign_keys="[author_id]")
```


在我们这个具体的例子中，不需要列表，所以可以写成： 


```python
author = db.relationship('User', backref='articles', foreign_keys="author_id")
```


### 测试  

在修改过后，我们运行程序，测试一下代码  
我们先给`User`表添加两条数据   


```
>>> zhangsan = User(email='zhangsan@123.com', name='张三')
>>> lisi = User(emaill='lisi@123.com', name='李四')
>>> db.session.add_all([zhangsan,lisi])
>>> db.session.commit()
```


接着给`Article`表添加一条记录，指定`Author`为`张三(users.id=1)`，`Reviewer`为`李四(usersid)`   


```
>>> article = Article()
>>> article.title = "Test"
>>> article.author_id = 1
>>> article.reviewer_id = 2
>>> db.session.add(article)
>>> db.session.commit()
```



我们来做查询操作  


```
>>> article = Article.query.get(1)
>>> article
<Article id=1, title=Test>
>>> article.author
<User id=1, name=张三>
>>> article.reviewer
<User id=2, name=李四>
```


可以看到我们可以正确的查询到`article.author`和`article.reviewer`，关于[SQLAchemy中处理两张表之间存在多个外键的情况]()这个问题我们已经解决。  

### 扩展  


在`relationship()`中我们添加了`backref`参数来对关系提供反向引用，这样更加方便了我们的查询操作。示例： 


```
>>> zhangsan = User.query.filter_by(name='张三').first()
>>> zhangsan
<User id=1, name=张三>
>>> zhangsan.articles
[<Article id=1, title=Test>]
>>> zhangsan.review_articles
[]
```


因为我们给`artice.author`添加了`articles`的反向引用，给`article.reviewer`添加了`review_articles`的反向引用。  
所以对于`User 张三`来说，他是`article Test`的`Author`，可以通过`article.author`来查询得到`张三`。也可以通过`zhangsan.articles`反向查询得到`Test`这篇文章。  
同时，因为`张三`不是任何一篇文章的`reviewer`，所以通过`zhangsan.review_articles`查询到结果为空列表。  
同样的，我们来看`李四`的查询操作：


```
>>> lisi = User.query.filter_by(name='李四').first()
>>> lisi
<User id=2, name=李四>
>>> lisi.articles
[]
>>> lisi.review_articles
[<Article id=1, title=Test>]
```


结果其实跟`张三`的查询是类似的，只是两人的角色`author`和`reviewer`不同，这里不再啰嗦。  