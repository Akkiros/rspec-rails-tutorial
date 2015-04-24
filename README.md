# Developing Rails app with RSpec

This tutorial is for user who want developing rails app with RSpec.

refered to this [site][link]

## Using

- Rails 4.0.4
- Ruby 2.0.0
- mysql2 0.3.18

## Gemfile

```sh
group :development, :test do
  gem 'rspec-rails'
  gem 'factory_girl_rails'
end

group :test do
  gem 'faker'
  gem 'capybara'
  gem 'guard-rspec'
  gem 'launchy'
end
```

## Setting test database

edit your ``config/database.yml``

MySQL

```sh
test:
  adapter: mysql2
  encoding: utf-8
  reconnect: false
  database: [[ your-database-name ]]_test
  pool: 5
  username: [[ your-database-username ]]
  password: [[ if you use password ]]
  host: 127.0.0.1 # if you want use remote mysql server, replace to remote ip
  port: 3306
  timeout: 5000
```

## RSpec Configuration

after run ``bundle install`` start rspec setting.

```sh
$ rails g rspec:install
```

this command initialize spec/ folder.

and edit ``.rspec``file in your project folder for easy to see result.

```sh
--format documentation
```

## Generators

edit ``config/application.rb`` and add following code.

```sh
config.generators do |g|
  g.test_framework :rspec,
    fixtures: true,
    view_specs: false,
    helper_specs: false,
    routing_specs: false,
    controller_specs: true,
    request_specs: true
  g.fixture_replacement :factory_girl, dir: 'spec/factories'
end
```

when you make models or controllers(like ``rails g model user``), spec file made automatically.

## Model test

When we make model, add some validates, class methods and associations.

Now we check validates, class methods doing well and has correct associations.

### Validates

Assume that we make a blog app and want to make User model.

We think that we want user's email and password are presence, email is uniqueness and password is have minimum 6 words. Yes, this is model validates.

it's very simple. open ``app/models/user.rb`` and add following codes.

```sh
validates :email, :password, presence: true
validates :email, uniqueness: true
validates :password, length: { minimum: 6 }
```

Before test, make test data using FactoryGirl.

open ``spec/factories/user.rb`` and add following codes.

```sh
FactoryGirl.define do
  factory :wonjae, class: User do |f|
    f.email "test@test.com"
    f.password "99e193ca18"
  end
  
  factory :minsoo, class: User do |f|
    f.email "test2@tester.com"
    f.password "992193ca18"
  end
end
```

When we run ``@user = FactoryGirl.create(:wonjae)`` ``@user`` have email "test@test.com" and password  "99e193ca18". it's very good for testing.

Now we should test this validates. open ``spec/models/user_spec.rb`` and make test.

```sh
require 'rails_helper'
require 'spec_helper'

RSpec.describe User, type: model do
  context '#create' do
    it 'email should presence' do
      expect(FactoryGirl.create(:wonjae, email: '')).not_to be_valid
    end
    
    it 'password should presence' do
      expect(FactoryGirl.create(:wonjae, password: '')).not_to be_valid
    end
    
    it 'email should unique' do
      FactoryGirl.create(:wonjae)
      
      expect(FactoryGirl.build(:wonjae)).not_to be_valid
    end
  end
end
```

*Tips*
> If you want to use guard-rails, execute command ``guard`` in shell, guard-rails is run.  
When you change your codes, guard run test automatically. It's very useful when you write test codes.  
else, execute command ``rspec spec/models/user_spec.rb`` run test also. I recommend first.



[link]: http://everydayrails.com/2012/03/12/testing-series-rspec-setup.html