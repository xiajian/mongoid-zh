---
layout: post
title: 安装 
---

## Getting Started
----

    Prerequisites
    Installation
    Configuration
    Logging
    Replica Sets
    Sharding 

## Prerequisites
----

There are a few things you need to have in your toolbox before tackling a web application using Mongoid.在将Mongoid加入到自己的web开发工具箱之前，需要一些前置的条件：

-  A good to advanced knowledge of Ruby. Ruby程序高级知识。
-  Have good knowledge of your web framework if using one.深入了解web框架。
-  A thorough understanding of MongoDB. 对MongoDB的整体性理解。
-  Aug 5,上述的这三个条件都不满足。

This may seem like a "thank you Captain Obvious" moment, however if you believe that you can just hop over to Mongoid because you read a blog post on how cool Ruby and MongoDB were, you are in for a world of pain.

如果仅仅因为在博客的文章上看到Ruby和MongoDB是如何的酷，就认为自己可以直接跳到Mongoid上，哈哈，有的你受的。

Mongoid leverages many aspects of the Ruby programming language that are not for beginner use, and sending the core team into a frenzy tracking down a bug for a common Ruby mistake is a waste of our time, and all of the other users of the framework as well.

Mongoid利用了Ruby中很多不为认知的方面，所以不要因为一些Ruby语法错误就将其当作bug发送给核心开发团队，这非常的浪费时间。
	

> ![](/images/achtung.png)  THE DATABASE IS NOT A BLACK BOX.数据库不是黑盒  
> Mongoid is an abstraction to make application developers' lives easier, however the internals leverage the power of MongoDB and it is truly important to know what is going on under the covers. This is why the documentation provides the exact queries that Mongoid is executing against the database when you call a persistence operation. If we took the time to tell you, you should listen. :)  
> Mongoid是让开发者更加轻松的数据库抽象，了解黑盒之下的东西，并利用内部杠杆启动MongoDB的能力不可或缺，这也是为什么文档提供了Mongoid可以对在数据库上执行精确的查询。

## Introduction
----

The preferred method for installing Mongoid is with bundler. Simply add Mongoid to your `Gemfile`.

> gem "mongoid", "~> 3.1.6"

Alternatively you can get the Mongoid gem direcly from rubygems.org:

> $ gem install mongoid

推荐使用bundle安装mongoid，这只要简单的将gem添加到Gemfile中。当然，也可以直接从rubygems.org中安装mongoid.
	
The minimum version of MongoDB that is required for you to run Mongoid is 2.0.0 for 3.0.x and 2.2.0 for 3.1.x.
一般而言，Mongodb 2.0对应 mongoid 3.0.x，2.2.0对应3.1.x。

## Configuration
----

Mongoid configuration can be done through a mongoid.yml that specifies your options and database sessions. The simplest configuration is as follows, which sets the default session to "localhost:27017" and provides a single database in that session named "mongoid".

Mongoid的配置可以通过mongoid.yml来指定，最简配置如下：

    development:
      sessions:
        default:
          database: mongoid
          hosts:
            - localhost:27017

### Rails Applications（Rails应用程序）

You can generate a config file by executing the generator and then editing myapp/config/mongoid.yml to your heart's desire. Mongoid will then handle everything else from there.

$ rails g mongoid:config  通过Rails生成mongoid的配置文件

When Mongoid loads its configuration, it chooses the environment to used based on the following order:

Mongoid将会加载配置，并基于如下的流程选择相应的运行环境：

- `Rails.env` if using Rails. Rails则考虑Rails.env。
-  `Sinatra::Base.environment` if using Sinatra. Sinatra则使用Base.environment环境
-  `RACK_ENV` environment variable. RACK_ENV的环境变量
-  `MONGOID_ENV` environment variable.  

If you are not using any rack based application and want to override the environment programatically, you can pass a second paramter to load! and Mongoid will use that.如果没有使用任何基于Rack的应用程序，并且想要通过编程覆盖环境。可以通过向load!方法传递第二个参数：

> Mongoid.load!("path/to/your/mongoid.yml", :production) #Mongoid加载对应的运行环境。

### Anatomy of a Mongoid Config（解析Mongoid的配置）

Let's have a look at a full-blown mongoid.yml and explain the full power of what Mongoid can do. The following configuration has its explanation in the comments above each key. 

考虑一个完整的mongoid.yml的配置，并完整的解释Mongoid的全部的能力。如下的配置文件以及其中注释的清晰的解释了Mongodb的每一点。

{% highlight ruby %}
# Tell Mongoid which environment this configuration is for.标明哪个环境对应哪个配置
production:
  # This starts the session configuration settings. You may have as
  # many sessions as you like, but you must have at least 1 named
  # 'default'. 配置设置会话，必须有一个命名为default。


  sessions:
    # Define the default session.
    default:
      # A session can have any number of hosts. Usually 1 for a single
      # server setup, and at least 3 for a replica set. Hosts must be
      # an array of host:port pairs. This session is single server.
      #一个会话可以拥有多个主机。通常一个主机对应一个启动服务器，以及至少3个replica集合。主机必须是host:port数组对。如下是单个服务器会话。
      hosts:
        - flame.mongohq.com:27017
      # Define the default database name. 定义默认数据库的名字
      database: mongoid
      # Since this database points at a session connected to MongoHQ, we must
      # provide the authentication details. 指向MongoHQ会话链接，需要提供一些权限细节（就是用户名和密码）
      username: user
      password: password
    # This defines a secondary session at a replica set. 定义replica集合会话
    replica_set:
      # This configuration is a 3 node replica set.
      hosts:
        - dedicated1.myapp.com:27017
        - dedicated2.myapp.com:27017
        - dedicated3.myapp.com:27017
      database: mongoid
      # We can set session specific options, like reads executing
      # on secondary nodes, and defaulting the session to safe mode. 可以指定特定会话的选项，比如在第二个节点上执行读，并设置为safe模式。

      options:
        consistency: :eventual #一致性设置，最终一致性
        safe: true #打开安全模式
    # This defines a tertiary session at a Mongos fronted shard. 第三个session是前端分片。
    shard:
      # This configuration is a Mongos shard server.
      hosts:
        - mongos.myapp.com:27017
      database: mongoid
    # This configuration shows an authenticated replica set via a uri. 通过uri展示权限分片集。
    another:
      uri: mongodb://user:pass@59.1.22.1:27017,59.1.22.2:27017/mongoid
  # Here we put the Mongoid specific configuration options. These are explained
  # in more detail next. 最后放置了Mongoid的特定的配置选项。
  options:
    allow_dynamic_fields: false
    identity_map_enabled: true
    include_root_in_json: true
    include_type_for_serialization: true
    # Note this can also be true if you want to preload everything, but this is
    # almost never necessary. Most of the time set this to false. 预加载模型。
    preload_models:
      - Canvas
      - Browser
      - Firefox
    scope_overwrite_exception: true
    raise_not_found_error: false #没有发现时，抛出异常。
    skip_version_check: false
    use_activesupport_time_zone: false
    use_utc: true
{% endhighlight %}

### Configuration options（配置选项）

Mongoid currently supports the following configuration options, either provided in the mongoid.yml or programatically (defaults in parenthesis).

Mongoid当前支持如下的配置选项，可以通过mongoid.yml或者编程实现(括号中为默认参数)

-  `allow_dynamic_fields`(true): When attributes are not defined as fields but added to an object, they will get fields added for them dynamically and will get persisted. If set to false an error will get raised when attempting to set a value that has no field defined.
-  `allow_dynamic_fields`(true): 当属性没有定义为类的域但被添加到对象中时，mongoid江自动添加相应的域并将其持久化。如果该属性设置false时，则在尝试给未定义的属性设置值时，会引发异常。
-  `identity_map_enabled`(false): When set to true Mongoid will store documents loaded from the database in the identity map by their ids, so subsequent database queries for the same document in the same unit of work do not hit the database. This is only for relation queries at the moment. See the identity map documentation for more info.
-  `identity_map_enabled`(false): 该属性设置为true时，将从数据库中加载的文档，以其标记映射(identity map)的ids进行保存，所以后续在同一单元中，查询相同的文档不必访问数据库。这仅仅适用统一时刻的关系查询，更多信息参考映射文件。
-  `include_root_in_json`(false): When set to true mongoid will include the name of the root document and the name of each association as the root element when calling #to_json on a model.
-  `include_root_in_json`(false): 该属性设置为true时，mongoid将包含根文档的名字并将每个关联名作为根元素调用模型上的to_json方法。
-  `include_type_for_serialization`(false): When set to true this will tell Mongoid to include the "_type" field when serializing to JSON and XML.
-  `include_type_for_serialization`(false): 该属性设置为true时，将在序列化对象为JSON和XML时，加上_type属性域。
-  `preload_models`(false): Tells Mongoid to preload application model classes on each request in environments where classes are not being cached. Specify an array of class names when enabling, only to the classes that use inheritance.
-  `preload_models`(false): 设置Mongoid将会对环境中没有缓存的类，在每个请求上预渲染应用程序模型类。仅仅针对那些继承类。
-  `protect_sensitive_fields`(true): Mongoid by default will auto protect `_id` and `_type` from mass assignment. Set this to false if you are daring with your application's security.
-  `protect_sensitive_fields`(true): Mongoid默认自动保护`_id`和`_type`的错误赋值。如果对应用程序的安全性相当自信，可以设置为false。
-  `raise_not_found_error`(true): Will raise a Mongoid::Errors::DocumentNotFound when attempting to find a document by an id that doesnt exist. When set to false will only return nil for the same query.
-  `raise_not_found_error`(true): 该属性设置当尝试寻找id不存在的文档时，是否会抛出Mongoid::Errors::DocumentNotFound。设置为false时，则仅仅返回为nil。
-  `skip_version_check`(false): If you are having issues authenticating against MongoHQ or MongoMachine because of access to the system collection being not allowed, set this to true.
-  `skip_version_check`(false): 如果由于接入系统，而对MongoHQ和MongoMachine的权限认证抱有问题，可以将其设置为真。
-  `scope_overwrite_exception`(false): This will instruct Mongoid to raise an error if you define a scope with the same name as an existing method.
-  `scope_overwrite_exception`(false): 该属性设置为true时，当定义和已存在的方法同名的scope时，会抛出异常。
-  `use_activesupport_time_zone`(true): When in a Rails app will tell Mongoid to convert all times in the application to the local defined time zone in Active Support. 设置时区转换。
-  `use_utc`(false): Instructs Mongoid to convert all times to UTC times in all cases。指导Mongoid是否将所有时间转换为UTC时间。 

If you would like to see samples, there is one in the [Mongoid repository](https://github.com/mongoid/mongoid/blob/master/spec/config/mongoid.yml) and one in the [Echo sample application](https://github.com/mongoid/echo/blob/master/config/mongoid.yml).

如果想看配置文件的例子，可以参考在[Mongoid的版本库](https://github.com/mongoid/mongoid/blob/master/spec/config/mongoid.yml)或者[Echo sample application](https://github.com/mongoid/echo/blob/master/config/mongoid.yml)。

### Getting Rid of Active Record（脱离Active Record）

Now that you have a mongoid.yml you can't wait to delete that pesky database.yml, right? Do it and you'll start getting ActiveRecord errors all over the place. You don't need ActiveRecord unless you're trying to use Mongo in concert with a SQL database. Here's how you remove ActiveRecord from the most recent version of Rails 3... 

现在，有了mongoid.yml，你迫不及待的删除那令人讨厌的database.yml，不是吗？ 如果你真这么做了，将会被ActiveRecord错误所淹没。除非mongo和SQL数据库同唱一台戏，否则并不需要ActiveRecord。但是如何从已有Rails 3中移除Active Record? 

Open `myapp/config/application.rb` and near the top, remove the line require "rails/all" and add the following lines so you end up with this:

打开config/application.rb，并在其中将“rails/all”替换为如下的的这些gem包，并在环境文件中注释掉涉及ActiveRecord的配置：

> require "action_controller/railtie"  
> require "action_mailer/railtie"   
> require "active_resource/railtie"  
> require "rails/test_unit/railtie"  
> # require "sprockets/railtie"  
> # Uncomment this line for Rails 3.1+

For Rails 3.2+ you'll also need to remove configuration options for Active Record that reside in your environments, ie myapp/config/environments/development.rb. Make sure the lines are commented out like as follows.

对于Rails 3.2+，需要在环境中移除关于Active Record的配置选项。例如，在myapp/config/environments/development.rb中，确保注释如下的行:

> # config.active_record.mass_assignment_sanitizer = :strict   
> # config.active_record.auto_explain_threshold_in_seconds = 0.5

For Rails 3.2.3+ you'll also need to comment out the following line in myapp/config/application.rb.

对于Rails 3.2.3+，还需要注释myapp/config/application.rb中的如下行:

> # config.active_record.whitelist_attributes = true

You can also generate your new rails app sans Active Record like so.

如果想要在生成新的rails程序时跳过Active Record，其命令如下:

> rails new app_name --skip-active-record

### Sinatra, Padrino, and others

You can create your mongoid.yml and place it anywhere you like. Just be sure that on application initialization you do the following:

在程序的初始化加入Mongoid.load!("path/to/your/mongoid.yml")，就可以将创建的配置文件放置到任意的地方。

Mongoid.load!("path/to/your/mongoid.yml") 

## Logging
----

Changing logging options is done simply by telling Mongoid or Moped's logger to have a different level. Logging is turned off by default.

可以通过设置Mongoid的记录等级来改变日志选项。默认情况下，日志是关闭的。
{% highlight ruby %}
module MyApplication
  class Application < Rails::Application
    Mongoid.logger.level = Logger::DEBUG #设置记录为debug模式
    Moped.logger.level = Logger::DEBUG
  end
end
{% endhighlight %}

If you want to change the logger instance, you can simply just set a new one.
如果想要改变日志实例(logger), 只要简单的这是一个新的即可。

> Mongoid.logger = Logger.new($stdout)  
> Moped.logger = Logger.new($stdout)

## Replica Sets(复制集)
----

For replica sets, you only need to put each member of the replica set under the `hosts` section in your `mongoid.yml` - Mongoid and Moped will take care of the rest. The default consistency is `:eventual`, which means that reads will attempt to go to secondaries. If you don't want this, switch this option to `:strong`, which will send everything to the master node.

对于复制集，只要在mongoid.yml中将所有的复制集的成员放置到host选项下，然后放心的交给Mongoid和Moped处理。 默认的一致性选项是最终一致性，这可能有些读写不一致的现象，可以通过:strong选项，从而保证所有的数据和Master节点保持一致。

{% highlight ruby %}
sessions:
  default:
    hosts:
      - repl0.myapp.com:27017
      - repl1.myapp.com:27017
      - repl3.myapp.com:27017
    database: mongoid
    options:
      consistency: :strong
{% endhighlight %}

## Sharding（分片）
----

If you are using Mongoid in a sharded MongoDB environment and want to tell Mongoid to include the shard keys in its updates, specify this at the model class level.

如果在分片的Mongodb的环境中使用Mongoid，可以在模型层包含分片的键。
{% highlight ruby %}
class Person
  include Mongoid::Document
  field :first_name, type: String
  field :last_name, type: String
  shard_key :first_name, :last_name
end
{% endhighlight %}

In your mongoid.yml, just ensure that you are pointed at the mongos server in your hosts.在配置文件中，只要确保在主机中指向了正确的mongo 服务器。
{% highlight ruby %}
sessions: 
  default: 
    hosts: 
      - mongos.myapp.com:27017 
    database: mongoid 
    options: 
      consistency: :eventual 
{% endhighlight %}

