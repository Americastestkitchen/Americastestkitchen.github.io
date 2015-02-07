---
layout: post
title:  "Postgres Storage for Rollout Gem"
author: Patrick Hereford
github: Americastestkitchen/rollout_postgres_store
github_username: phereford
date:   2015-02-06 23:15:00
categories: ruby rollout
---

[Rollout](https://github.com/FetLife/rollout) is a popular Ruby gem that helps manage the sometimes tedious nature
of feature flagging. It was originally architected with Redis as a dependency.
While we do love Redis immensely, we felt now was not quite the time to add yet
another piece of infrastructure to our application. Thankfully, a semi-recent
update for Rollout removed the Redis dependency and allows the end user to use
any key-value store. The only requirements are that an instance of the store
needs to respond to the `get`, `set`, and `del` methods.  

## The Decision

Given that we are not ready to add another piece of infrastructure to our stack,
we decided to create a wrapper for [PostgreSQL's Hstore](http://www.postgresql.org/docs/9.0/static/hstore.html) feature.
We created a new ActiveRecord model called FeatureFlag that has a singular
attribute (migration and model below).

{% highlight ruby %}
class CreateFeatureFlags < ActiveRecord::Migration
  def change
    create_table :feature_flags do |t|
      t.hstore :data
    end
  
    execute 'CREATE INDEX feature_flags_data ON feature_flags USING GIN(data)'
  end
end

class FeatureFlag < ActiveRecord::Base
end
{% endhighlight %}

So far, so good. We didn't necessarily need an empty AR model and could have
simply written some SQL statements inside of some PG connections, but most of 
our business logic resides in AR.

## Implementing the Store

Now for the fun part! As stated above, an instance of our store needs to be able
to handle the `get`, `set`, and `del` methods appropriately. Just from perusing the 
rollout codebase, the methods are defined as the following:
- `get` takes a key argument and returns the value for the key
- `set` takes two arguments (key, value) and creates or updates the key-value pair
- `del` takes a key argument and deletes the key-value pair

Below is the code that handles these scenarios.  

{% highlight ruby %}
class RolloutPostgresStore
  def initialize(model, attribute)
    @model = model
    @attribute = attribute
  end

  def get(key)
    if flag = @model.where("#{@attribute} ? '#{key}'").first
      flag.send(@attribute)[key]
    end
  end

  def set(key, value)
    current = get(key)

    if current.nil?
      create_feature_flag(key, value)
    else
      update_flag(key, value)
    end
  end

  def del(key)
    if flag = @model.where("#{@attribute} ? '#{key}'").first
      flag.delete
    end
  end

  private
  def create_feature_flag(key, value)
    flag = @model.new
    flag.send("#{@attribute}=", { key => value })
    flag.save
  end

  def update_flag(key, value)
    flag = @model.where("#{@attribute} ? '#{key}'").first
    flag.send(@attribute)[key] = value
    flag.send("#{@attribute}_will_change!")
    flag.save
  end
end
{% endhighlight %}

It is relatively straightforward. We really utilize ActiveRecord a ton here but
I think it's an okay decision given our data structures above. Like any engineer
that adheres to the values of testing, I also threw some tests in to cover the
various scenarios for the RolloutStore. They are below.  

{% highlight ruby %}
ROLLOUT = Rollout.new(RolloutPostgresStore.new(FeatureFlag, 'data'))

describe RolloutPostgresStore, '.new' do
  it 'assigns the model instance variable' do
    store = RolloutPostgresStore.new(FeatureFlag, 'data')
    expect(store.instance_variable_get(:@model)).to be FeatureFlag
  end

  it 'assigns the attribute instance variable' do
    store = RolloutPostgresStore.new(FeatureFlag, 'data')
    expect(store.instance_variable_get(:@attribute)).to eql 'data'
  end
end

describe RolloutPostgresStore, '#get' do
  it 'returns nil for not having found a FeatureFlag with the given key' do
    store = RolloutPostgresStore.new(FeatureFlag, 'data')
    expect(store.get('test_key')).to be nil
  end

  it 'returns the value for a given key when a FeatureFlag is found' do
    ROLLOUT.activate_percentage(:search, 20)

    store = RolloutPostgresStore.new(FeatureFlag, 'data')
    expect(store.get('feature:search')).to eql '20||'
  end
end

describe RolloutPostgresStore, '#set' do
  it 'receives the create_feature_flag message for not having found a flag' do
    expect_any_instance_of(RolloutPostgresStore).to receive(:create_feature_flag).at_most(:once)

    RolloutPostgresStore.new(FeatureFlag, 'data').set('search', '20||')
  end

  it 'receives the update_flag message for having found a flag' do
    ROLLOUT.activate_percentage(:search, 20)

    expect_any_instance_of(RolloutPostgresStore).to receive(:update_flag).at_most(:once)

    RolloutPostgresStore.new(FeatureFlag, 'data').set('feature:search', '20||123')
  end
end

describe RolloutPostgresStore, '#del' do
  it 'returns the object deleted for finding the key to delete' do
    ROLLOUT.activate_percentage(:search, 30)

    # There is only one feature flag per the activation above
    feature_flag = FeatureFlag.first
    store = RolloutPostgresStore.new(FeatureFlag, 'data')
    expect(store.del('feature:search')).to eq feature_flag
  end

  it 'returns nil for not being able to find the key to delete' do
    expect(RolloutPostgresStore.new(FeatureFlag, 'data').del('feature:test')).to be nil
  end
end
{% endhighlight %}

## Conclusion
We wanted to create something with as small a footprint as possible without 
having to add Redis to our stack for feature flagging. This little piece of 
software seems to fit exactly that.

Of note, we have gemified the code and it is now available on rubygems.
