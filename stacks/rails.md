# Rails

Developing web applications is standing on the shoulders of giants. We use many open source libraries to get specific tasks done. This is only a short list of the - if you need something larger feel free to look at the [Awesome Rails Gem](https://github.com/hothero/awesome-rails-gem) list.

## Views

- [HAML](http://haml.info/) for writing valid HTML pages.
- [SASS / SCSS](http://sass-lang.com/) as a preprocessor for CSS.
- [Webpacker](https://github.com/rails/webpacker) for saner ES6/npm integration.

## Form Models

- [OffTheRecord](https://gitlab.9elements.com/9elements/off_the_record) Validation without persistence.

## File uploading and Image Processing

- [Carriewave](https://github.com/carrierwaveuploader/carrierwave) for file uploading and image processing.
- [ActiveStorage](https://github.com/rails/rails/tree/master/activestorage) is the Railsway solution and might supercede Carrierwave in the near future.

## Background Processing

- [Sidekiq](https://github.com/mperham/sidekiq) for everything long running that can't be done in a controller action.
- [Whenever](https://github.com/javan/whenever) for kicking off regular jobs at intervals through CRON


## Full Text Search

- [Elastic Search](https://github.com/elastic/elasticsearch-rails) for communicating with an elastic search database.
- [Elastic Search Model](https://github.com/elastic/elasticsearch-rails/tree/master/elasticsearch-model) for keeping models in sync between a relational database and elastic search.

## Authentication

- [Sorcery](https://github.com/Sorcery/sorcery) for a lightweight authentication library.
- [Devise](https://github.com/plataformatec/devise) a more heavyweight authentication library.
- [OmniAuth](https://github.com/omniauth/omniauth) a OAuth 2 authentication framework (for FB or Twitter).
- [CanCanCan](https://github.com/CanCanCommunity/cancancan) for permission management.
- [JSON Web Token](https://jwt.io/) for authenticating against an API or signing messages.

## Testing

- [RSpec](http://rspec.info/) Behaviour Driven
Development for Ruby.
- [capybara](https://github.com/teamcapybara/capybara) Acceptance test framework for web applications.
- [simplecov](https://github.com/colszowka/simplecov) for Code Coverage.
- [factory girl](https://github.com/thoughtbot/factory_girl) for creating fixtures.
- [faker](https://github.com/stympy/faker) for faking seeds.
- [vcr](https://github.com/vcr/vcr) for mocking 3rd party API requests.

# Developer Tools

- [Pry Rails](https://github.com/rweng/pry-rails) is an IRB alternative and runtime developer console.
- [Pry Byebug](https://github.com/deivid-rodriguez/pry-byebug)Step-by-step debugging and stack navigation in Pry.
- [Pry Stack Explorer](https://github.com/pry/pry-stack_explorer) Walk the stack in a Pry session.
- [Letter Opener](https://github.com/ryanb/letter_opener) and [Letter Opener Web](https://github.com/fgrehm/letter_opener_web) for testing email responses.
- [NewRelic](https://newrelic.com/) Performance and exception tracking.

# API Server

- [Rack Cors](https://github.com/cyu/rack-cors) for setting cors header.

# API Client

- [Httparty](https://github.com/jnunemaker/httparty) for consuming APIs.

# Deployment

- [Capistrano](http://capistranorb.com/) for deployment.

# Ruby Linting

- [rubocop](https://github.com/bbatsov/rubocop) for keeping code standards consistent accross a larger team.

Please also look at our Rails extended architecture guides.

# Admin

- [Rails Admin](https://github.com/sferik/rails_admin) for low budget clients who need a machine room.

# Frontend Libraries

- [Fontello](https://github.com/fontello/fontello) Icon Fonts
- [Fontawesome Rails](https://github.com/bokmann/font-awesome-rails) More Icon Fonts (you will just want one of these font libs)
- [Foundation Rails](https://rubygems.org/gems/foundation-rails) CSS Framework
- [Twitter Bootstrap](https://github.com/seyhunak/twitter-bootstrap-rails) Twitter Bootstrap for Rails
