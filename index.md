---
layout: post
---

## 文档目录
----

- [第1章 安装 ](/mongoid-zh/zh/00-install.html)
- [第2章 文档类型](/mongoid-zh/zh/01-documents.html)
- [第3章 持久化](/mongoid-zh/zh/02-persistence.html)
- [第4章 查询](/mongoid-zh/zh/03-querying.html)
- [第5章 关系](/mongoid-zh/zh/04-relations.html)
- [第6章 嵌套属性](/mongoid-zh/zh/05-nested_attributes.html)
- [第7章 回调](/mongoid-zh/zh/06-callbacks.html)
- [第8章 验证](/mongoid-zh/zh/07-validation.html)
- [第9章 索引](/mongoid-zh/zh/08-indexing.html)
- [第10章 rails](/mongoid-zh/zh/09-rails.html)
- [第11章 extras](/mongoid-zh/zh/10-extras.html)
- [第12章 升级](/mongoid-zh/zh/11-upgrading.html)
- [第13章 贡献](/mongoid-zh/zh/12-contributing.html)
- [第14章 性能](/mongoid-zh/zh/13-performance.html)
- [第15章 技巧&FAQ](/mongoid-zh/zh/14-tips.html)

> 备注: 开发中使用的3.1.3的Mongoid的gem包，所以，翻译打算以3.X为准。

## Mongoid
----

Mongoid (pronounced mann-goyd) is an Object-Document-Mapper (ODM) for MongoDB written in Ruby. It was conceived in August, 2009 during a whiskey-induced evening at the infamous Oasis in Florida, USA by Durran Jordan.

诞生在一个充满威斯基的夜晚。

The philosophy of Mongoid is to provide a familiar API to Ruby developers who have been using Active Record or Data Mapper, while leveraging the power of MongoDB's schemaless and performant document-based design, dynamic queries, and atomic modifier operations.

Mongoid的哲学是为Ruby开发者提供和Active Record或Data Mapper相似的API，并得利于MongoDB的无模式，基于文档的高性能，动态查询以及原子修饰操作。


This is the site for Mongoid 3 and 4 documentation, along with Origin and Moped. If you want the Mongoid 2 docs, please go here.

该文档主要讨论了Mongoid 3和 4的文档。如果想要Mongoid 2的文档，移步这里。
Mongoid是API接口类似Active Record的Mongodb的Ruby驱动库，感觉好复杂的表述，大体就这么个节奏。

## Sample Syntax
----

> ![](/images/achtung.png)Mongoid 支持 MRI 1.9.3+, JRuby 1.6.0+的1.9模式，所有的代码使用1.9的语法

{% highlight ruby %}
class Artist
  include Mongoid::Document
  field :name, type: String
  embeds_many :instruments
end

class Instrument
  include Mongoid::Document
  field :name, type: String
  embedded_in :artist
end

syd = Artist.where(name: "Syd Vicious").between(age: 18..25).first
syd.instruments.create(name: "Bass")
syd.with(database: "bands", session: "backup").save!
{% endhighlight %}

