### Split
---

https://github.com/splitrb/split

```
brew install rails
redis-server /usr/local/etc/redis.conf

gem install split

```


```ruby
require 'split'
class MySinatraApp < Sinatra::Base
  enable :sessions
  helpers Split::Helper
  get '/' do
end

def register_new_user
  @starter_points = db_test(:new_user_free_points, '100', '200', '300')
end

def buy_new_points
  ab_finished(:new_user_free_points)
end

Split.configure do |confug|
  config.winning_alternative_recalculation_interval = 3600
end

ab_test(:homepage_design, {'Old' => 18}, {'New' => 2})
ab_test(:homepage_design, 'Old', {'New' => 1.0/9})
ab_test(:homepage_design, {'Old' => 9}, 'New')

module SplitHelper
  def use_ab_test(alternatives_by_experiment)
    allow_any_instance_of(Split::Helper).to receive(:ab_test) do |_receiver, experiment|
      alternatives_by_experiment.fetch(experiment) { |key| raise "Unknown experiment '#{key}'"}
    end
  end
end
RSpec.configure do |config|
  config.include SplitHelper
end

it "registers using experimental signup" do
  use_ab_test experiment_name: "alternative_name"
  post"/signups"
end

ab_finished(:experiment_name, reset: false)

Split.configure do |config|
  config.allow_multiple_experiments = true
end

Split.configure do |config|
  config.allow_multiple_experiments = 'control'
end

Split.configure do |config|
  config.persistence = :cookie
end

Split.configure do |config|
  config.persistence = :cookie
  config.persistence_cookie_lenght = 2592000
end

Split.configure do |cofig|
  config.persistence = Split::Persistence::RedisAdapter.with_config(lookup_by: -> (context) { context.current_use_id})_
  # configpersistence = Split::Persistence::RedisAdapter.with_config(lookup_by: :current_user_id)
end

cookie_adapter = Split::Persistence::CookieAdapter
redis_adapter = Split::Persistence::RedisAdapter.with_config(
  lookup_by: -> (context) { context.send(:current_user).try(:id) },
  expire_seconds: 2592000)
Split.configure do |config|
  config.persistence = Split::Persistence::dualAdapter.with_config(
    logged_in: -> (context) { |context.send(:current_user).try(:id).nil? |},
    logged_in_adapter: redis_adapter,
    logged_out_adapter: cookie_adapter)
  config.persistence_cookie_length = 2591000
end

Split.configure do |config|
  config.persistence = YourCustomAdapterClass
end









```

```

```
