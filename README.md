# premailer-rails

CSS styled emails without the hassle.

[![Build Status][build-image]][build-link]
[![Gem Version][gem-image]][gem-link]
[![Dependency Status][deps-image]][deps-link]
[![Code Climate][gpa-image]][gpa-link]
[![Coverage Status][cov-image]][cov-link]

## Introduction

This gem is a drop in solution for styling HTML emails with CSS without having
to do the hard work yourself.

Styling emails is not just a matter of linking to a stylesheet. Most clients,
especially web clients, ignore linked stylesheets or `<style>` tags in the HTML.
The workaround is to write all the CSS rules in the `style` attribute of each
tag inside your email. This is a rather tedious and hard to maintain approach.

Premailer to the rescue! The great [premailer] gem applies all CSS rules to each
matching HTML element by adding them to the `style` attribute. This allows you
to keep HTML and CSS in separate files, just as you're used to from web
development, thus keeping your sanity.

This gem is an adapter for premailer to work with [actionmailer] out of the box.
Actionmailer is the email framework used in Rails, which also works outside of
Rails. Although premailer-rails has certain Rails specific features, **it also
works in the absence of Rails** making it compatible with other frameworks such
as sinatra.

## How It Works

premailer-rails works with actionmailer by registering a delivery hook. This
causes all emails that are delivered to be processed by premailer-rails. This
means that by simply including premailer-rails in your `Gemfile` you'll get
styled emails without having to set anything up.

Whenever premailer-rails processes an email, it collects the URLs of all linked
stylesheets (`<link rel="stylesheet" href="css_url">`). Then, for each of these
URLs, it tries to get the content through a couple of strategies. As long as
a strategy does not return anything, the next one is used. The strategies and
their order are as follows:

1.  **Cache:** If there's a file in cache matching that URL, the cache content
    is returned.  The cache right now is rather rudimentary. Whenever a CSS file
    is retrieved, it is stored in memory such that subsequent requests to the
    same file are faster. The caching is disabled inside Rails in the
    development environment.

2.  **File System:** If there's a file inside `public/` with the same path as in
    the URL, it is read from disk. E.g. if the URL is
    `http://cdn.example.com/assets/email.css` the contents of the file located
    at `public/assets/email.css` gets returned if it exists.

3.  **Asset Pipeline:** If Rails is available and the asset pipeline is enabled,
    the file is retrieved through the asset pipeline. E.g. if the URL is
    `http://cdn.example.com/assets/email-fingerprint123.css`, the file
    `email.css` is requested from the asset pipeline. That is, the fingerprint
    and the prefix (in this case `assets` is the prefix) are stripped before
    requesting it from the asset pipeline.

4.  **Network:** As a last resort, the URL is simply requested and the response
    body is used. This is usefull when the assets are not bundled in the
    application and only available on a CDN. On Heroku e.g. you can add assets
    to your `.slugignore` causing your assets to not be available to the app
    (and thus resulting in a smaller app) and deploy the assets to a CDN such
    as S3/CloudFront.

## Installation

Simply add the gem to your `Gemfile`:

```ruby
gem 'premailer-rails'
```

premailer-rails requires either [nokogiri] or [hpricot]. It doesn't list them as
a dependency so you can choose which one to use. Since hpricot is no longer
maintained, I suggest you to go with nokogiri. Add either one to your `Gemfile`:

```ruby
gem 'nokogiri'
# or
gem 'hpricot'
```

If both gems are loaded for some reason, premailer chooses hpricot.

That's it!

## Configuration

Premailer itself accepts a number of options. In order for premailer-rails to
pass these options on to the underlying premailer instance, specify them
as follows (in Rails you could do that in an initializer such as
`config/initializers/premailer_rails.rb`):

```ruby
Premailer::Rails.config.merge!(preserve_styles: true, remove_ids: true)
```

For a list of options, refer to the [premailer documentation]. The default
configs are:

```ruby
{
  input_encoding: 'UTF-8',
  generate_text_part: true
}
```

If you don't want to automatically generate a text part from the html part, set
the config `:generate_text_part` to false.

Note that the options `:with_html_string` and `:css_string` are used internally
by premailer-rails and thus will be overridden.

If you're using this gem outside of Rails, you'll need to call
`Premailer::Rails.register_interceptors` manually in order for it to work. This
is done ideally in some kind of initializer, depending on the framework you're
using.

## Usage

premailer-rails processes all outgoing emails by default. If you wish to skip
premailer for a certain email, simply set the `:skip_premailer` header:

```ruby
class UserMailer < ActionMailer::Base
  def welcome_email(user)
    mail to: user.email,
         subject: 'Welcome to My Awesome Site',
         skip_premailer: true
  end
end
```

Note that the mere presence of this header causes premailer to be skipped, i.e.,
even setting `skip_premailer: false` will cause premailer to be skipped. The
reason for that is that the `skip_premailer` is a simple header and the value is
transformed into a string, causing `'false'` to become truethy.

## Small Print

### Author

Philipe Fatio ([@fphilipe][fphilipe twitter])

[![Support via Gittip][tip-image]][tip-link]

### License

premailer-rails is released under the MIT license. See the [license file].

[build-image]: https://travis-ci.org/fphilipe/premailer-rails.svg
[build-link]:  https://travis-ci.org/fphilipe/premailer-rails
[gem-image]:   https://badge.fury.io/rb/premailer-rails.svg
[gem-link]:    https://rubygems.org/gems/premailer-rails
[deps-image]:  https://gemnasium.com/fphilipe/premailer-rails.svg
[deps-link]:   https://gemnasium.com/fphilipe/premailer-rails
[gpa-image]:   https://codeclimate.com/github/fphilipe/premailer-rails.svg
[gpa-link]:    https://codeclimate.com/github/fphilipe/premailer-rails
[cov-image]:   https://coveralls.io/repos/fphilipe/premailer-rails/badge.svg
[cov-link]:    https://coveralls.io/r/fphilipe/premailer-rails
[tip-image]:   https://rawgithub.com/twolfson/gittip-badge/0.1.0/dist/gittip.svg
[tip-link]:    https://www.gittip.com/fphilipe/

[premailer]:    https://github.com/premailer/premailer
[actionmailer]: https://github.com/rails/rails/tree/master/actionmailer
[nokogiri]:     https://github.com/sparklemotion/nokogiri
[hpricot]:      https://github.com/hpricot/hpricot

[premailer documentation]: http://rubydoc.info/gems/premailer/1.7.3/Premailer:initialize

[fphilipe twitter]: https://twitter.com/fphilipe
[license file]:     LICENSE
