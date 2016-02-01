# Trice

Provide **reference time** concept to application. Use it instead of ad-hoc `Time.now`.

## Progress

- [ ] Usage items
- [ ] query param stubbing
- [ ] header stubbing
- [ ] stubbing toggle
- [ ] Test helpers (unit)
- [ ] Test helpers (E2E)

### Setting consistent reference time

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'trice'
```

And then execute:

```
$ bundle
```

## Usage

### With Rails controller

This gem aims to give consistency of time handling using refrence time to Rails application.
The layer, which should set reference time, is controller layer, because reference time is one of external input.

Include `Trice::ControllerMethods` to your controller

```ruby
class ApplicationController < AC::Base
  include Trice::ControllerMethods
end
```

Then your controller and view gets an accessor method to access consistent time object.

- `requested_at`: returns timestamp of action invoked performing or stubbed timestamp.

### Include helper module outside of controller

Inlude `Trice::ReferenceTime` add `reference_time` instance method to lookup current reference time.

Use it in Rails model.

```ruby
class MyWork
  include Trice::ReferenceTime

  def do!(at: nil)
    self.done_at = at || reference_time
  end
end
```

### Setting consistent reference time

Set reference time with `Trice.refrence_time = _time_` or `Trice.with_refrence_time(_time_, &block)`. The time is stored in thread local variable. Accessible with `Trice.reference_time`..

```ruby
p Time.now
=> 2016-02-01 11:25:37 +0900

Trice.with_refrence_time = Time.iso8601('2016-02-01T09:00:00Z')
p Trice.reference_time
# => 2016-02-01 09:00:00 UTC

Trice.with_refrence_time(Time.iso8601('2016-02-01T10:00:00Z')) do
  p Trice.reference_time
  # => 2016-02-01 10:00:00 UTC
end

Trice.with_refrence_time = nil
p Trice.reference_time
# => raise Trice::NoRefrenceTime
```

## Time Stubbing

Trice allows you to stub reference time to run travelled time-testing and / or previewing your app in future time.

Set `_requested_at=<timish>` query parameter like below
```
$ curl https://example.com/campaigns/12345?_requested_at=20160215130
```

Or can set HTTP header, useful for tests.
```
TriceRequestedAt: 2016-02-15T13:00:00+09:00
```

Value format, which specified both query parameter and header, should be `Time.parsse` parasable.

#### Enable/Disable stubbing

Toggle requested at stubbing in `config/initializers`. The default is below, enabled unelss `Rails.env.production?`.

```ruby
Trice.support_performing_at_stubbing = Rails.env.production?
```

Setting callable object let you choice enable/disable dinamically by seeing request.

```ruby
our_office_network = IPAddr.new('203.0.113.0/24')

Trice.support_performing_at_stubbing = ->(req) {
  next true unless Rails.env.production?

  our_office_network.include?(req.remote_ip)
}
```

## Test helpers

There is a test helper method for feature spec.

```ruby
RSpec.configure do |config|
  config.include Trice::SpecHelper
end
```

I recommend to give reference time to a modle by method and/or constructor argument because reference time is an external input, should be handled controller layer.
But sometimes it is required  from deep inside of model logics and tests for them.

Model unit spec has `with_refrence_time` and `set_now_to_reference_time` declarition method to set `Trice.reference_time` in an example.

```ruby
describe MyModel do
  context  do
    set_now_to_reference_time
    let(:model) { MyModel.find_by_something(key) }

    specify do
      # can accessible `reference_time` in MyModel#do_something
      expect { model.do_something }.not_to raise(Trice::NoRefrenceTime)
    end
  end
end
```

Feature specs (or othre Capybara based E2E tests) also has helper method using stubbing mechanism. `stub_requested_at <timish>` set `X-Trice-Requested-At` automatically.

```ruby
scenario 'See Hinamatsuri special banner at 3/3 request' do
  stub_requested_at Time.zone.parse('2016-03-03 10:00')

  visit root_path
  within '#custom-header' do
    expect(page).to contain 'ひな祭り'
  end
end
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/[USERNAME]/trice. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.


## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

