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
``` python
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
    author_id = db.Column(db.Integer, ForeignKey("users.id")
    reviewer_id = db.Column(db.Integer, ForeignKey("users.id")
    author = db.relationship('User', backref='articles')  
    reviewer = db.relationship('User', backref='review_articles')
    
```  
看起来程序的设计应该是没问题，可运行的结果真的跟我们预想的一样吗？  
当我们运行代码后，程序给抛出一个错误：
```
sqlalchemy.exc.AmbiguousForeignKeysError: 
Could not determine join condition between parent/child tables on relationship Article.author - there are multiple foreign key paths linking the tables.  Specify the 'foreign_keys' argument, providing a list of those columns which should be counted as containing a foreign key reference to the parent table.  
```
可以看到SQLAchemy提示无法确定`Article.author`的父子表之间的关联，原因在于两张表之间存在多个外键。需要我们指定`foreign_keys`参数，提供一个包含关联了父表（即`User`表）外键的字段列表（`list`)  
看起来非常让人迷惑，只好去阅读[SQLAchemy的官方文档](https://docs.sqlalchemy.org/en/latest/index.html)，在SQLAchemy