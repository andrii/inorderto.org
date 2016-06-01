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

### User signs in

### User signs out

### User resets password

## Writing the Rails generator

## Generating specs

## Deleting specs

## Skipping specs

## Testing specs
