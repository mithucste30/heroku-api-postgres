# Heroku::Api::Postgres

Ruby library to invoke Heroku Postgres APIs.
An extension to the official [Platform API]() gem to introduce the missing APIs for Postgres.

[![Build Status](https://travis-ci.org/coorasse/heroku-api-postgres.svg?branch=master)](https://travis-ci.org/coorasse/heroku-api-postgres)
[![Maintainability](https://api.codeclimate.com/v1/badges/4eead5d8263c37498953/maintainability)](https://codeclimate.com/github/coorasse/heroku-api-postgres/maintainability)

**This gem is not officialy supported**. We use the same APIs that the offical Heroku Toolbelt uses,
therefore is unrealistic that a breaking change in the APIs would break it, since it means it would break
both this gem and the official Heroku CLI.

Not all APIs are implemented at the moment but we are working hard on implementing all of them.

Please check the [list of implemented and not implemented services](docs/services.md).

Contributes and Pull Requests are welcome!

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'heroku-api-postgres'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install heroku-api-postgres

## Usage

Even if this gem does not require `platform-api` to be installed, you probably want to have it installed as well.

```ruby
# from platform api gem
platform_api_client = PlatformAPI.connect_oauth(ENV['HEROKU_OAUTH_TOKEN'])
```

this gem client needs to be instantiated as well in a similar way

```ruby
postgres_api_client = Heroku::Api::Postgres.connect_oauth(ENV['HEROKU_OAUTH_TOKEN'])
```

Look into [Models](docs/models.md) for a detailed description of the JSON objects returned by the APIs.
Those are the bare objects returned by the official Heroku API.

### Databases

```ruby
databases_client = postgres_api_client.databases
```

---

```ruby
database_info = databases_client.info(database_id)
```

returns a [Database](docs/models.md#database).

---

```ruby
database = postgres_api_client.databases.wait(database_id, wait_interval: 5)
```
Waits for the given database to be ready.

Polls every `wait_interval` seconds (default 3).


### Backups

```ruby
backups_client = postgres_api_client.backups
```

---

```ruby
backups = backups_client.info(app_id, backup_id)
```

returns a [Backup](docs/models.md#backup).

---

```ruby
backups = backups_client.list(app_id)
```

returns an array of [Backup](docs/models.md#backup).

The app_id can be either the name of your heroku app or the id.

[Check official API](https://devcenter.heroku.com/articles/platform-api-reference#app)

---

```ruby
schedules = backups_client.schedules(database_id)
```

Returns all the backup schedules associated with the database.

Returns an array of [Schedule](docs/models.md#schedule)

---

```ruby
schedule = backups_client.schedule(database_id)
```

Schedules the backups at 00:00 UTC.

Returns a [Schedule](docs/models.md#schedule)

---


```ruby
backup = backups_client.capture(database_id)
```
Captures a new backup for the given database

Returns a [Backup](docs/models.md#backup)


```ruby
backup_url = backups_client.url(app_id, backup_num)
```
Returns a temporary, public accessible URL to download a backup.
Needs the `num` attribute of a Backup.

Returns a [BackupUrl](docs/models.md#backup_url)

---

```ruby
backup = backups_client.restore(database_id, dump_url)
```
Restores a dump from a public URL.

Returns a [Backup](docs/models.md#backup)

---

```ruby
backup = postgres_api_client.backups.wait(app_id, backup_id, wait_interval: 5)
```
Waits for the given backup/restore to be ready.

Polls every `wait_interval` seconds (default 3).

You can pass a block to be executed at each interval:

```ruby
backup = postgres_api_client.backups.wait(app_id, backup_id) do |info|
    puts "Processed #{info[:processed_bytes]} bytes"
end
```


### How do I get the database_id ?
You can obtain a database id by calling the Heroku Platform API

```ruby
addons = heroku.addon.list
databases = addons.select { |addon| addon['addon_service']['name'] == 'heroku-postgresql' }
databases_ids = databases.map{ |addon| addon['id'] }
```

Check also the [Offical API](https://devcenter.heroku.com/articles/platform-api-reference#add-on)

## Development

After checking out the repo, run `bin/setup` to install dependencies.
Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`.
To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`,
which will create a git tag for the version, push git commits and tags,
and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/coorasse/heroku-api-postgres.
This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to
the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).

## Code of Conduct

Everyone interacting in the Heroku::Api::Postgres project’s codebases, issue trackers, chat rooms and mailing lists is
expected to follow the [code of conduct](https://github.com/coorasse/heroku-api-postgres/blob/master/CODE_OF_CONDUCT.md).
