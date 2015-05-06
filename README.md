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

this command initialize ``spec/`` folder.

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

When we make model, add some validates and associations.

Now we check validates, has correct associations doing well.

### Validates

Assume that we make a blog app and want to make User model.

We think that we want user's email and password are presence, email is uniqueness and password is have minimum 6 words. Yes, this is model validates.

it's very simple. open ``app/models/user.rb`` and add following codes.

```sh
class User < ActiveRecord::Base
  validates :email, :password, presence: true
  validates :email, uniqueness: true
  validates :password, length: { minimum: 6 }
end
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

***Tips***
> If you want to use guard-rails, execute command ``guard`` in shell, guard-rails is run.  
When you change your codes, guard run test automatically. It's very useful when you write test codes.  
else, execute command ``rspec spec/models/user_spec.rb`` run test also. I recommend first.

### Associations

In blog, 1 user can have many posts and one post can have many comments. Yes, it's 1:N associations!  
Now we test User model has proper association with Post.

Add following codes in ``app/models/user.rb``

```sh
  has_many :posts, dependent: :destroy
```

This code means that User can have many posts and when delete user, also delete posts too.

And add following codes in ``app/models/post.rb``

***Before***
> If you didn't create Post model, run ``rails g model post body:string user_id:integer`` and ``rake db:migrate``

```sh
class Post < ActiveRecord::Base
  belongs_to :user
end
```

Now we make associations before we talk. So it's test time!

open ``spec/models/user_spec.rb``

```sh
context 'associations' do
  it 'can have many posts' do
    as = User.reflect_on_associations(:posts)
    expect(as.macro).to eq(:has_many)
  end
end
```

and open ``spec/models/post_spec.rb``

```sh
context 'associations' do
  it 'belongs to user' do
    as = Post.reflect_on_associations(:user)
    expect(as.macro).to eq(:belongs_to)
  end
end
```

These tests are doing well. But you want see associations doing really well, run ``rails c``, make user instance ``user`` and ``user.posts`` returns ActiveRecord associations. 


## Controller test

Before we test model validates, associations. If model consist of validates and associations, controller sonsist of actions and logic.

So in controller test, we test actions and logic.

### Actions

Before we assume that we make blog app. So we need to login feature. This is action.

Let's make login controller. run ``rails g controller login``

***Before***
> In login controller, we return result using json. So in test we parse json.

Open ``spec/controllers/login_controller.rb`` and add following codes.


```sh
require 'rails_helper'
require 'spec_helper'

RSpec.describe LoginController, type: :controller do
  before(:each) do
    @email = 'test@test.com'
    @password = 'asdfaa'
  end
  
  describe '#login' do
    it 'email is nil' do
      post :login, { password: @password }
      
      body = JSON.parse(response.body)
      
      expect(body['success']).to eq(false)
    end
    
    it 'password is nil' do
      post :login, { email: @email }
      
      body = JSON.parse(response.body)
      
      expect(body['success']).to eq(false)
    end
    
    it 'no user' do
      post :login, { email: @email, password: @password }
      
      body = JSON.parse(response.body)
      
      expect(body['success']).to eq(false)
    end
    
    it 'login success' do
      post :login, { email: @email, password: @password }
      
      body = JSON.parse(response.body)
      
      expect(body['success']).to eq(true)
    end
  end
end
```

We test one action, but test is so long. Because tests are test all possible cases. See this [link][all]


## Author

Wonjae Kim / [@Akkiros][Facebook]


[link]: http://everydayrails.com/2012/03/12/testing-series-rspec-setup.html
[Facebook]: https://www.facebook.com/akkiros
[Linkedin]: https://www.linkedin.com/profile/view?id=79379340
[all]: http://betterspecs.org/#all