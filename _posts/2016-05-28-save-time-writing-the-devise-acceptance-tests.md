---
layout: default
title: Save Time Writing the Devise Acceptance Tests
---

# {{ page.title }}

By Andrii Ponomarov on {{ page.date | date: '%B %d, %Y' }}

<div id="story">In order to save time spent writing the Devise
authentication acceptance specs from scratch
as a developer,
I want them to be generated automatically
when I run the devise generator.
</div>

## Which Devise features to test?

<table>
  <tr>
    <th>Method</th>
    <th>Path</th>
    <th>Responsibility</th>
    <th>Feature</th>
  </tr>

  <tr class="selected">
    <td>GET</td>
    <td>/users/sign_in</td>
    <td>Renders the login form</td>
    <td rowspan="2">User signs in</td>
  </tr>
  <tr class="selected separated">
    <td>POST</td>
    <td>/users/sign_in</td>
    <td>Logs in a user and redirects</td>
  </tr>

  <tr class="selected separated">
    <td>DELETE</td>
    <td>/users/sign_out</td>
    <td>Logs out a user and redirects</td>
    <td>User signs out</td>
  </tr>

  <tr class="selected">
    <td>POST</td>
    <td>/users/password</td>
    <td>Sends the password reset email and redirects</td>
    <td rowspan="5">User resets password</td>
  </tr>
  <tr class="selected">
    <td>GET</td>
    <td>/users/password/new</td>
    <td>Renders the password reset form</td>
  </tr>
  <tr class="selected">
    <td>GET</td>
    <td>/users/password/edit</td>
    <td>Renders the change password form</td>
  </tr>
  <tr class="selected">
    <td>PATCH</td>
    <td>/users/password</td>
    <td rowspan="2">Updates a password, logs in a user and redirects</td>
  </tr>
  <tr class="selected separated">
    <td>PUT</td>
    <td>/users/password</td>
  </tr>

  <tr class="skipped separated">
    <td>GET</td>
    <td>/users/cancel</td>
    <td>Clears the Devise session and redirects</td>
    <td>User cancels signup via OAuth</td>
  </tr>

  <tr class="selected">
    <td>POST</td>
    <td>/users</td>
    <td>Creates a user, logs in and redirects</td>
    <td rowspan="2">User signs up</td>
  </tr>
  <tr class="selected separated">
    <td>GET</td>
    <td>/users/sign_up</td>
    <td>Renders the registration form</td>
  </tr>

  <tr class="skipped">
    <td>GET</td>
    <td>/users/edit</td>
    <td>Renders the edit user form</td>
    <td rowspan="3">User updates account</td>
  </tr>
  <tr class="skipped">
    <td>PATCH</td>
    <td>/users</td>
    <td rowspan="2">Updates a user and redirects</td>
  </tr>
  <tr class="skipped separated">
    <td>PUT</td>
    <td>/users</td>
  </tr>

  <tr class="skipped">
    <td>DELETE</td>
    <td>/users</td>
    <td>Deletes a user and redirects</td>
    <td>User cancels account</td>
  </tr>
</table>

## How to test the features?

### User signs up

<pre><code class="ruby">require 'rails_helper'

feature 'User signs up' do
  scenario 'with valid data' do
    visit new_user_registration_path

    fill_in 'Email', with: 'username@example.com'
    fill_in 'Password', with: 'password'
    fill_in 'Password confirmation', with: 'password'
    click_button 'Sign up'

    expect(page).to have_text 'Welcome! You have signed up successfully.'
    expect(page).to have_link 'Sign Out'
    expect(page).to have_current_path root_path
  end

  scenario 'with invalid data' do
    visit new_user_registration_path

    click_button 'Sign up'

    expect(page).to have_text "Email can't be blank"
    expect(page).to have_text "Password can't be blank"
    expect(page).to have_no_link 'Sign Out'
  end
end
</code></pre>

### User signs in

<pre><code class="ruby">require 'rails_helper'

feature 'User signs in' do
  scenario 'with valid credentials' do
    user = create :user

    sign_in user

    expect(page).to have_text 'Signed in successfully.'
    expect(page).to have_link 'Sign Out'
    expect(page).to have_current_path root_path
  end

  scenario 'with invalid credentials' do
    user = build :user

    sign_in user

    expect(page).to have_text 'Invalid Email or password.'
    expect(page).to have_no_link 'Sign Out'
  end
end
</code></pre>

### User signs out

<pre><code class="ruby">require 'rails_helper'

feature 'User signs out' do
  scenario 'user signed in' do
    user = create :user

    sign_in user

    click_link 'Sign Out'

    expect(page).to have_text 'Signed out successfully.'
    expect(page).to have_no_link 'Sign Out'
    expect(page).to have_current_path root_path
  end
end
</code></pre>

### User resets password

<pre><code class="ruby">require 'rails_helper'

feature 'User resets a password' do
  scenario 'user enters a valid email' do
    user = create :user

    visit new_user_password_path

    fill_in 'Email', with: user.email
    click_button 'Send me reset password instructions'

    expect(page).to have_text 'You will receive an email with instructions'
    expect(page).to have_current_path new_user_session_path
  end

  scenario 'user enters an invalid email' do
    visit new_user_password_path

    fill_in 'Email', with: 'username@example.com'
    click_button 'Send me reset password instructions'

    expect(page).to have_text 'Email not found'
  end

  scenario 'user changes password' do
    token = create(:user).send_reset_password_instructions

    visit edit_user_password_path(reset_password_token: token)

    fill_in 'New password', with: 'p4ssw0rd'
    fill_in 'Confirm new password', with: 'p4ssw0rd'
    click_button 'Change my password'

    expect(page).to have_text 'Your password has been changed successfully.'
    expect(page).to have_current_path root_path
  end

  scenario 'password reset token is invalid' do
    visit edit_user_password_path(reset_password_token: 'token')

    fill_in 'New password', with: 'p4ssw0rd'
    fill_in 'Confirm new password', with: 'p4ssw0rd'
    click_button 'Change my password'

    expect(page).to have_text 'Reset password token is invalid'
  end
end
</code></pre>

## Writing the Rails generator

Let's declare Aruba as a development dependency in the `devise-specs.gemspec` file:

<pre><code class="gemspec">Gem::Specification.new do |s|
  s.name    = 'devise-specs'
  s.version = '0.0.1'
  s.summary = 'Generates the Devise acceptance tests.'
  s.authors = ['Andrii Ponomarov']

  s.add_development_dependency 'aruba', '~> 0.14.1'
end
</code></pre>

We could now install Aruba [via RubyGems](http://guides.rubygems.org/patterns/#runtime-vs-development) by building the gem and then installing it with all its dependencies using `gem install --dev devise-specs`. But the [convention](http://yehudakatz.com/2010/12/16/clarifying-the-roles-of-the-gemspec-and-gemfile/) is to load the gemspec in the `Gemfile` and use Bundler to solve the dependency hell problem with gem [isolation](http://12factor.net/dependencies):

<pre><code class="ruby">source 'https://rubygems.org'

gemspec
</code></pre>

Let's install Bundler with `gem install bundler`, dependendencies with `bundle install`, initialize the `features/` directory with `cucumber --init` and make sure Cucumber can execute the feature files:

<pre class="terminal">
root@814a2d18e7b7:/devise-specs# bundle exec cucumber
0 scenarios
0 steps
0m0.000s
</pre>

## Generating specs with factories

Omitted for brevity. [feature](https://github.com/andrii/devise-specs/blob/master/features/generate_specs/with_factories.feature)

<pre><code class="gherkin">Feature: With factories

  Background:
    Given I set up devise-specs

  Scenario: running a devise generator with Factory Girl installed
    Given I install factory_girl_rails
    When I run `rails generate devise User`
    Then the output should contain:
      """
            invoke  specs
              gsub    spec/rails_helper.rb
            insert    spec/factories/users.rb
            create    spec/support/factory_girl.rb
            create    spec/support/helpers.rb
            create    spec/features/user_signs_up_spec.rb
            create    spec/features/user_signs_in_spec.rb
            create    spec/features/user_signs_out_spec.rb
            create    spec/features/user_resets_password_spec.rb
      """
    And the file "spec/features/user_signs_up_spec.rb" should contain:
      """ruby
      ...
      """
    And the file "spec/features/user_signs_in_spec.rb" should contain:
      """ruby
      ...
      """
    And the file "spec/features/user_signs_out_spec.rb" should contain:
      """ruby
      ...
      """
    And the file "spec/features/user_resets_password_spec.rb" should contain:
      """ruby
      ...
      """
</code></pre>

You can implement step definitions for undefined steps with these snippets:

* `Given(/^I set up devise\-specs$/) do`
* `Given(/^I install factory_girl_rails$/) do`
* ``When(/^I run `rails generate devise User`$/) do``
* `Then(/^the output should contain:$/) do |string|`
* `Then(/^the file "([^"]*)" should contain:$/) do |arg1, string|`

To load Aruba step definitions let's add `require 'aruba/cucumber'` to `features/support/env.rb`.

<pre><code class="ruby">Given(/^I set up devise\-specs$/) do
  run_simple 'gem install rails --no-document'
  run_simple 'rails new . --skip-spring'

  append_to_file 'Gemfile', <<~RUBY
    gem 'devise'
    gem 'devise-specs', path: '../..'
    gem 'rspec-rails'
    gem 'capybara'
  RUBY

  # define root route
  copy '%/routes.rb', 'config/routes.rb'

  run_simple 'bundle install'
  run_simple 'rails generate rspec:install'
  run_simple 'rails generate devise:install'
end
</code></pre>

<pre class="terminal">
root@1a8ef1ddc064:/devise-specs# bundle exec cucumber
Feature: With factories

  Background:                   # features/generate_specs/with_factories.feature:3
      Given I set up devise-specs # features/step_definitions/devise-specs_steps.rb:1
            expected "gem install rails --no-document" to be successfully executed
</pre>

Bundler `features/suppport/env.rb`

<pre><code class="ruby">Before do
  delete_environment_variable 'RUBYOPT'
end
</code></pre>

`features/support/env.rb`

<pre class="terminal">
root@1a8ef1ddc064:/devise-specs# bundle exec cucumber
Feature: With factories

  Background:                   # features/generate_specs/with_factories.feature:3
      Given I set up devise-specs # features/step_definitions/devise-specs_steps.rb:1
            expected "gem install rails --no-document" to have finished in time
</pre>

<pre><code class="ruby">Aruba.configure do |config|
  config.exit_timeout = 100
end
</code></pre>

again

<pre class="terminal">
root@1a8ef1ddc064:/devise-specs# bundle exec cucumber
Feature: With factories

  Background:                   # features/generate_specs/with_factories.feature:3
      Given I set up devise-specs # features/step_definitions/devise-specs_steps.rb:1
            No existing fixtures directory found in "/devise-specs/features/fixtures", "/devise-specs/spec/fixtures", "/devise-specs/test/fixtures", "/devise-specs/fixtures".
</pre>

`fixtures/routes.rb`

<pre><code class="ruby">Rails.application.routes.draw do
  root 'home#index'
end
</code></pre>

<pre class="terminal">
root@1a8ef1ddc064:/devise-specs# bundle exec cucumber
Feature: With factories

  Background:                   # features/generate_specs/with_factories.feature:3
    Given I set up devise-specs # features/step_definitions/devise-specs_steps.rb:1
      expected "rails generate rspec:install" to be successfully executed
</pre>

<pre><code class="diff">--- a/features/support/env.rb
+++ b/features/support/env.rb
@@ -2,6 +2,7 @@ require 'aruba/cucumber'

 Before do
   delete_environment_variable 'RUBYOPT'
+  delete_environment_variable 'BUNDLE_GEMFILE'
 end
</code></pre>

Now when the above step passes and this is pending let's add a new step definition

<pre><code class="ruby">Given(/^I install factory_girl_rails$/) do
  append_to_file 'Gemfile', "gem 'factory_girl_rails'\n"

  run_simple 'bundle install'
end
</code></pre>

Then the output should contain: fails because output doesn't match

`lib/devise-specs.rb`

<pre><code class="ruby">require 'devise/specs/railtie'
</code></pre>

<pre><code class="ruby">module Devise
  module Specs
    class Railtie < Rails::Railtie
      generators do
        require 'generators/devise/devise_generator'

        Devise::Generators::DeviseGenerator.hook_for :specs, type: :boolean, default: true
      end
    end
  end
end
</code></pre>

<pre><code class="ruby">module Devise
  module Generators
    class SpecsGenerator < Rails::Generators::Base
      ATTRIBUTES = %(
        email 'username@example.com'
        password 'password')

      source_root File.expand_path("../templates", __FILE__)

      def require_supporting_files
        uncomment_lines 'spec/rails_helper.rb', /spec.support.*rb.*require/
      end

      def insert_fixture_replacement_attributes
        path  = 'spec/factories/users.rb'
        attrs = ATTRIBUTES.gsub(/^ {4}/, '')
        after = 'factory :user do'

        insert_into_file path, attrs, after: after
      end

      def create_factory_girl_config_file
        template 'factory_girl.rb', 'spec/support/factory_girl.rb'
      end

      def create_helpers_file
        template 'helpers.rb', 'spec/support/helpers.rb'
      end

      def create_specs
        template 'resource_signs_up_spec.rb',
                 'spec/features/user_signs_up_spec.rb'

        template 'resource_signs_in_spec.rb',
                 'spec/features/user_signs_in_spec.rb'

        template 'resource_signs_out_spec.rb',
                 'spec/features/user_signs_out_spec.rb'

        template 'resource_resets_password_spec.rb',
                 'spec/features/user_resets_password_spec.rb'
      end
    end
  end
end
</code></pre>

<pre><code class="ruby">RSpec.configure do |config|
  config.include FactoryGirl::Syntax::Methods
end
</code></pre>

<pre><code class="ruby">module Helpers
  def sign_in(resource)
    visit new_user_session_path

    fill_in 'Email', with: resource.email
    fill_in 'Password', with: resource.password
    click_button 'Log in'
  end
end

RSpec.configure do |config|
  config.include Helpers, type: :feature
end
</code></pre>

4 spec files - same content as above, paths

feature passes

## Generating specs with fabricators

## Deleting specs

## Skipping specs

## Testing specs
