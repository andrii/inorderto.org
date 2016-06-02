---
layout: default
title: Save Time Writing the Devise Acceptance Tests
---

# {{ page.title }}

By [Andrii Ponomarov](http://andriiponomarov.com) on {{ page.date | date: '%B %d, %Y' }}

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

## Generating specs

## Deleting specs

## Skipping specs

## Testing specs
