---
layout: post
title: 
---


Ruby 1.9.3

Are you getting an error like the following?

NoMethodError: undefined method `[]' for nil:NilClass
  from /.rvm/gems/ruby-1.9.2-p290/gems/moped-1.1.0/lib/moped/node.rb:74:in `block in command'
  from /.rvm/gems/ruby-1.9.2-p290/gems/moped-1.1.0/lib/moped/node.rb:522:in `[]'
  from /.rvm/gems/ruby-1.9.2-p290/gems/moped-1.1.0/lib/moped/node.rb:522:in `block (3 levels) in flush'
  from /.rvm/gems/ruby-1.9.2-p290/gems/moped-1.1.0/lib/moped/node.rb:521:in `map'
  from /.rvm/gems/ruby-1.9.2-p290/gems/moped-1.1.0/lib/moped/node.rb:521:in `block (2 levels) in flush'
  from /.rvm/gems/ruby-1.9.2-p290/gems/moped-1.1.0/lib/moped/node.rb:113:in `ensure_connected'
  from /.rvm/gems/ruby-1.9.2-p290/gems/moped-1.1.0/lib/moped/node.rb:517:in `block in flush'

If so, this is your issue. Until 1.9.3, Ruby did not have a way to decode binary string data accurately for both big endian and little endian systems across all the data types used in the MongoDB wire protocol. All data sent via the wire protocol, as well as the BSON specification itself use little endian and the ability to decode 32 bit and 64 bit signed longs and ints did not arrive in Ruby until this version. See String#unpack and Mongo Wire Protocol for more detailed information.

What this means is that prior to 1.9.3, the data received back from big endian systems such as SPARC or PPC would get incorrect data decoded when receiving replies from the database. We have thus made 1.9.3 the minimum requirement to be able to support all architectures.

For those of you on Heroku bamboo, you will need to do two things in order to run on 1.9.3 in your applications. First, you will need to upgrade to the Cedar stack, and then you will need to specify 1.9.3 as your Ruby in your application's Gemfile. See Multiple Rubies on Heroku for instructions on making this migration.
Count Performance

MongoDB uses non counting B-Tree indexes which causes issuing count commands to be extremely slow when providing a selector with the command. For example:

Band.where(name: "Depeche Mode").count
session[:bands].find(name: "Depeche Mode").count

This doesn't affect queries on small data sets, but large data sets (millions of documents) can experience count queries in the neighborhood of seconds, which can easily cripple application performance.

    See: SERVER-1752
    See: SERVER-2274

Reordering Embedded Documents

Due to only a partially implemented feature in MongoDB, the $ positional operator, it is not possible to accurately update embedded documents that are nested more than one level deep based on a query selector. For example:

session[:bands].find(
  "_id" => 1,
  "albums._id" => 2,
  "albums.tracks._id" => 3,
).update(
  "$set" => { "albums.$.tracks.$.name" => "Sober" }
)

The above query does not work, and is not possible in MongoDB. To get around this, Mongoid stores the index of the embedded documents in memory, and would execute the same query like so:

session[:bands].find(
  "_id" => 1,
  "albums._id" => 2,
  "albums.tracks._id" => 3,
).update(
  "$set" => { "albums.0.tracks.2.name" => "Sober" }
)

This works, however it comes with a caveat in that you cannot reorder your embedded documents in another process. If you were to do so, Mongoid would be updating the incorrect field. It is recommended until this is fixed that you sort your documents in memory and leave the underlying order untouched. See SERVER-831 for more information.
No GridFS Support

GridFS is marketed as a core database feature, when in fact it is not. It is simply a pattern for storing chunked file data as documents in a collection, just like any other document. The implementation of this behaviour is handled in the client drivers, not in the core database itself, which can lead to discrepencies in how this is handled across platforms.

Even if having this behaviour in the client is acceptable, the effects of this on application performance where you are not just storing file data is quite large. Since files are stored as documents, they consume RAM just as any other document in the database would, and can easily cause memory consumption on your server to max out. There are also limitations in chunking the data, such as you do not have the ability to update a file - you must delete the file and replace it with a new one.

Given this, we did not prioritize any work with GridFS at the front, but there is a gem in the pipeline for those who can wait a bit to upgrade. In the meantime you have a few options.

    Use the 10gen driver in conjunction with Mongoid 3 and Moped - the namespaces will not collide. 

    Take advantage of Ara Howard's mongoid-grid_fs gem: mongoid-grid_fs 

Relational Associations

Mongoid provides relational-style associations as a convenience for application developers who are used to dealing with relational databases, but we do not recommend you use these extensively.

MongoDB provides no transactions, and no join support, so it is impossible to ensure that the database is kept in a consistent state with regards to referencing documents from one collection to another. Also, without support for joins, you will find that the number of database queries executed grows in order to retrieve documents with their associations, and can have a huge affect on performance since this is not where MongoDB's power lies.

If you find that you have more relational associations in your application than embedded ones, it is recommended that you not use MongoDB and move to a relational database. Also, you will probably find this feature removed in future versions of Mongoid, in order to not allow developers to use the database in an incorrect fashion.
Safe Mode

Safe mode is turned off by default, as this is MongoDB's recommendation. However, we recommend that users, especially first timers, have this setting enabled in development and test mode. Doing so will allow you to better understand what is going on, rather than assuming your data was actually persisted.

To ensure safe mode is on in development and test, you should simply configure this in your mongoid.yml:

development:
  sessions:
    default:
      database: my_app_dev
      hosts:
        - localhost:27017
      options:
        safe: true
test:
  sessions:
    default:
      database: my_app_test
      hosts:
        - localhost:27017
      options:
        safe: true

Also we recommend keeping an eye on the MongoDB logs if you are getting started, as safe mode itself won't even catch all errors - sometimes MongoDB will simply log an error on the server but send a message back to the client that the operation succeeded.

    On OSX (with Homebrew): /usr/local/var/log/mongodb/output.log
    On Ubuntu (with apt-get): /var/log/mongodb/mongodb.log

Working With Other Mappers

It's not always the case that you are using MongoDB as your only database, and some apps like to combine databases for different use cases. Below is an example of accessing an Active Record model from Mongoid manually.

class Band
  include Mongoid::Document

  field :user_id, type: Integer

  def user
    @user ||= User.find(user_id)
  end

  def user=(user)
    self.user_id = user.id
    @user = user
  end
end

class User < ActiveRecord::Base
end

Sidekiq

If you are using Sidekiq, you need to ensure that MongoDB sessions are diconnected after each worker runs, or you will quickly overload MongoDB with more active connections than it can handle. In order to handle this seamlessly for you, we have provided a gem with a Sidekiq middleware that handles this - Kiqstand. You will need to be on Mongoid version 3.0.3 or higher.

To use Kiqstand, first add it to your Gemfile.

gem "kiqstand"

When you intialize Sidekiq, add the middleware to the Kiqstand server stack.

Sidekiq.configure_server do |config|
  config.server_middleware do |chain|
    chain.add Kiqstand::Middleware
  end
end

After this, all connection handling is taken care of for you.

