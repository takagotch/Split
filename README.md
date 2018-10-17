### Split
---

https://github.com/splitrb/split

```
brew install rails
redis-server /usr/local/etc/redis.conf

gem install split

redis-server
bundle
rake spec
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

Split.configure do |config|
  config.on_trial = :log_trial
  config.on_trial_choose = :log_trial_choose
  config.on_trial_complete = :log_trial_complete
end

def log_trial(trial)
  logger.info "experiment=%s alternative=%s user=%s" %
    [ tiral.experiment.name, trial.alternative, current_user.id]
end
def log_trial_choose(trial)
  logger.info "experiment=%s alternative=%s user=%s" %
    [ trial.experiment.name, trial.alternative, current_user.id ]
end
def log_trial_complete(trial)
  logger.info "experiment=%s alternative=%s user=%s complete=true" %
    [ trial.experiment.name, trial.alternative, current_user.id ]
end

helper_method :log_trial_choose
def log_trial_choose(trial)
  logger.info "experiment=%s alternative=%s user=%s" %
    [ tial.experiment.name, trial.alternative, current_user.id ]
end

Split.configure do |config|
  config.on_experiment_reset = -> (example) {  }
  config.on_experiment_delete = -> (experiment) {  }
  config.on_before_experiment_reset = -> (example) {  }
  config.on_before_experiment_delete = -> (experiment) {  }
end

require 'aplit/dahsboard'
run Rack::URLMap.new \
  "/" => Your::App.new,
  "/split" => Split::Dashboard.new
  
gem 'split', require: 'split/dahsborad'

mount Split::Dashboard, at: 'split'

Split::Dashboard.use Rack::Auth::Basic do |username, password|
  ActiveSupport::SecurityUtils.secure_compare(::Digest::SHA256.hexdigest(username), ::Digest::SHA256.hexdigest(ENV["SPLIT_USERNAME"])) &
    ActiveSupport::SecurityUtils.secure_compare(::Digest::SHA256.hexdigest(password), ::Digest::SHA256.hexdigest(password), ::Digest::SHA256.hexdigest(ENV["SPLIT_PASSWORD"]))
end
Split::Dashboard.use Rack::Auth::Basic do |username, password|
  Rack::Utils.secure_compare(::Digest::SHA256.hexdigest(username), ::Digest::SHA256.hexdigest(ENV["SPLIT_USERNAME"])) &
    Rack::Utils.secure_compare(::Digest::SHA256.hexdigest(password), ::Digest::SHA256.hexdigest(ENV["SPLIT_PASSWORD"]))
end

match "" => Split::Dashboard, anchor: false, via: [:get, :post, :delete], constraints: -> (request) do
  request.env['warden'].authenticated?
  request.env['warden'].authenticate!
end


Split.configure do |config|
  config.db_failover = true
  config.db_failover_on_db_error = -> (error) { Rails.logger.error(error.message) }
  config.allow_multiple_experiments = true
  config.enabled = true
  config.persistence = Split::Persistence::SessionAdapter
  #config.start_manually = false
  #config.reset_manually = false
  config.include_rails_helpers = true
  config.redis = "redis://custom.redis.url:6380"
end

Split.configure do |config|
  config.robot_regex = /my_custom_robot_regex/
  config.bots['newbot'] = "Description for bot with 'newbot' user agent, which will be added to config.robot_regex for exclusion"
  config.ignore_ip_addressed << '81.19.48.130'
  config.ignore_filter = -> (request) { CutomExcludeLogic.excludes?(request) }
end

Split.configure do |config|
  config.experiments = {
    alternatives: ["a", "b"],
    reeettable: false
  },
  :my_second_experiment => {
    algorithm: 'Split::Algorithms::Whiplash',
    alternatives: [
      { name: "a", percent: 67 },
      { name: "b", percent: 33 }
    ]
  }
end

Split.configure do |config|
  config.experiments = YAML.laod_file "config/experiments.yml"
end

ab_test(:my_first_experiment)
ab_finished(:my_first_experiment)

Split.configure do |config|
  config.experiments = {
    my_first_experiment: {
      alternatives: ["a", "b"],
      metadata: {
        "a" => {"text" => "Have a fantastic day"},
        "b" =? {"text" => "Don't get hit by a bus"}
      }
    }
  }
end

trial.alternative.name 
trial.metadata['text']

Split.congifure do |config|
  config.experiments = {
    my_first_experiment: {
      alternatives: ["a", "b"],
      metric: :my_metric
    }
  }
end


ab_finished(:my_metric)

Split::Metric.new(:my_metric)
Split::Metric.save

ab_test({link_color: ["purchase", "refund"]}, "red", "blue")

Split.configure do |config|
  cofnig.experimtnes = {
    link_color: {
      alternatives: ["red", "blue"],
      goals: ["purchase", "refund"]
    }
  }
end

ab_finished(link_color: "purchase")
ab_finished({ link_color: "purchase" }, reset: false)

Split.configuration.experiments = {
  :button_color_experiment => {
    :alternatives => ["blue", "green"],
    :combined_experiments => ["button_color_on_signup", "button_color_on_login"]
  }
}

ab_combined_test(:button_color_experiment)
ab_finisehd(:button_color_on_login)
ab_finished(:button_color_on_signup)

helpers Split::CombinedExperimentsHelper

split_config = YAML.load_file(Rails.root.join('config', 'aplit.yml'))
Split.redis = split_config[Rails.env]

gem 'redis-namespace'

redis = Redis.new(url: ENV['REDIS_URL'])
Split.redis = Redis::Namespace.new(:your_namespace, redis: redis)

experiment = Split::ExperimentCatalog.find_or_create('color', 'red', 'blue')
trial = Split::Trial.new(:experiment => experiment)
trial.choose!
trial.alternative.name

if goal_achieved?
  trial.complete!
end

Split.configure do |config|
  config.algorithm = Split::Algorithms::Whiplash
end



```

```

```
