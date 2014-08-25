---
layout: post
title: 标识映射 
---

## Identity Map
----

Mongoid's identity map is an implementation of the Identity Map Pattern.

    Usage
    Unit of Work
    Testing

Usage

The identity map in Mongoid is a current aid to assist with excessive database queries in relations, and is necessary for eager loading to work.

To enable the identity map, simply change the configuration option in your mongoid.yml.

identity_map_enabled: true

You can enable the identity map programatically as well.

Mongoid.identity_map_enabled = true

When a document is now loaded from the database, it is automatically added to the identity map by its class and id. Subsequent request for that document by its id will not hit the database, but rather pull the document back from the identity map itself.

When performing Model.find queries with the identity map enabled, Mongoid will automatically look for the document in the identity map first and return it if it exists before hitting the database.

Band.find(id) #=> Fetch the document from the database.
Band.find(id) #=> Gets the document from the identity map.

Band.find([ id_one, id_two ]) #=> Fetch the documents from the database.
Band.find([ id_one, id_two ]) #=> Gets the documents from the identity map.
Band.find([ id_one, id_three ])
  #=> Gets the first document from the identity map, the second from the db.

Note the identity map does not store nil values if the query to the database did not return any results. So subsequent finds for the same id will hit the database again. This also includes eager loading.
The Unit of Work

To prevent database objects from becoming stale, the documents in the identity map should only exist in a single unit of work, which is usually a single request to the application.
Rails

No extra work is needed for Rails users, Mongoid will automatically clear out the identity map after each request.
Sinatra

You can require the Mongoid identity map middleware in your application to clear out the map with each request:

use Rack::Mongoid::Middleware::IdentityMap

Non Rack Based Applications

For users not using rack based apps, you will need to wrap your requests or application defined unit of work in a Mongoid.unit_of_work block:

Mongoid.unit_of_work do
  Person.create(title: "Grand Poobah")
end

	

The unit of work can also be used to temporarily disable the identity map and force documents to load from the database.

# Disable the identity map for the current thread only.
Mongoid.unit_of_work(disable: :current) do
  Band.find(1)
end

# Disable the identity map for all threads.
Mongoid.unit_of_work(disable: :all) do
  Band.find(1)
end

Rake Tasks and Background Jobs

It is important to note that you should never be using the identity map when executing background jobs, rake tasks, etc. unless you really know that not many documents will be loaded and memory consumption will be low. Otherwise it's a server takedown waiting to happen. In these cases it's best to wrap your task or job in a unit of work.

desc "A long running rake task"
task "db:migrate_data" => :environment do
  Mongoid.unit_of_work(disable: :all) do
    # Do my work here.
  end
end

class BackgroundJob
  @queue = :background

  def self.perform(id)
    Mongoid.unit_of_work(disable: :all) do
      # Do work here.
    end
  end
end

Testing

If you have the identity map enabled in your application, you should set up a global hook to clear out the map before each test so the test suite does not create memory bloat. For example in RSpec in spec_helper.rb.

RSpec.configure do |config|
  config.before(:each) { Mongoid::IdentityMap.clear }
end

