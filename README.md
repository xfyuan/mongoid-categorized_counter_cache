# Mongoid::CategorizedCounterCache

Like CounterCache of ActiveRecord or Mongoid, but with category.

[![Build Status](https://travis-ci.com/flanker/mongoid-categorized_counter_cache.svg?branch=master)](https://travis-ci.com/flanker/mongoid-categorized_counter_cache)

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'mongoid-categorized_counter_cache'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install mongoid-categorized_counter_cache

## Usage

### Counter categories by a field value

```
# User class
categorized_counter_cache :RELATION, by: :CATEGORY_FILED
```

For example, you have a `company` and `user` relationship. With normal `CounterCache` it is easy to cache the number of users on a company.

But if we want to cache the counter of `active_users` and `inactive_users`, we could use `Mongoid::CategorizedCounterCache` to setup the counter cache with the category field (`status` in this case)

The code will be like:

```
class Company
  include Mongoid::Document
  include Mongoid::Attributes::Dynamic

  field :users_count, type: Integer, default: 0
  field :users_active_count, type: Integer, default: 0
  field :users_inactive_count, type: Integer, default: 0

  embeds_many :users
end

class User
  include Mongoid::Document
  include Mongoid::CategorizedCounterCache

  field :name
  field :status    # active, inactive

  belongs_to :company, counter_cache: true
  categorized_counter_cache :company, by: :gender
end
```

#### When create a user

```
user = User.create company: company, status: :active

company.users_active_count # => old_value + 1
```

#### When update a user

```
user = User.create company: company, status: :active
user.update_attributes status: :inactive

company.users_active_count # => old_value - 1
company.users_inactive_count # => old_value + 1
```

#### When update the relation

```
user = User.create company: company, status: :active
user.update_attributes company: :another_company

company.users_active_count # => old_value - 1
another_company.users_active_count # => old_value + 1
```

#### When delete a user 

```
user = User.create company: company, status: :active
user.destroy

company.users_active_count # => old_value - 1
```

### Counter categories by a relation

You can specify the category by a field from a relation.

```
# User class
categorized_counter_cache :company, by: 'city.region'
```

For example:

```
module CategoryAsRelation

  class Company
    include Mongoid::Document

    field :users_count, type: Integer, default: 0
    field :users_west_count, type: Integer, default: 0
    field :users_east_count, type: Integer, default: 0

    has_many :users
  end

  class City
    include Mongoid::Document

    field :name
    field :region    # west, east

  end

  class User
    include Mongoid::Document
    include Mongoid::CategorizedCounterCache

    field :status    # active, inactive

    belongs_to :city

    belongs_to :company, counter_cache: true
    categorized_counter_cache :company, by: 'city.region'
  end

end
```

In this case, `company` has counter cache of different categories of users. And the cateogry is from city's region to which user belongs.

### Other options

You can user category as a prefix in the counter cache field name:

```
# User class
categorized_counter_cache :company, by: :status, prefix: true
```

And the counter cache field will be:

```
field :active_users_count, type: Integer, default: 0
field :inactive_users_count, type: Integer, default: 0
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/flanker/mongoid-categorized_counter_cache.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
