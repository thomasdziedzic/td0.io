---
title: "Lobste.rs: Upgrading Rails to 6.0"
---

## Starting up.

Rails 6.0 was [recently](https://github.com/rails/rails/releases/tag/v6.0.0) released on August 16, 2019.
Since [I upgraded lobste.rs last year to 5.2](/posts/2018-08-21-lobsters-upgrading-rails.html), I decided I'll take it up again this year now that I have done it a couple of times.

## Helping hands with the required ruby upgrade.

One of the requirements for upgrading to rails 6.0 is that you need to be running at least ruby 2.5.
Luckily [hmadison](https://github.com/hmadison) on github worked on getting lobste.rs upgraded from ubuntu 16.04 to 18.04 which includes ruby 2.5.
His [pull request](https://github.com/lobsters/lobsters-ansible/pull/44) helped out my cause a lot and saved me a lot of time that I would have to spend learning ansible.

## Handling incompatible gems with rails 6.0.

After bumping rails to 6.0 in the Gemfile, I ran a `bundle install` and found out that there were 2 gems that were incompatible with rails 6.0.
The actionpack-page_caching gem doesn't have a released version compatible with rails 6.0 so I used a version available in git.
I also opened a [ticket](https://github.com/rails/actionpack-page_caching/issues/58) to track releasing a version compatible with rails 6.0.
The second incompatible gem was rspec-rails. It has a beta available that is compatible under the version 4.0.0.beta2.
Using these unreleased versions isn't ideal but after running the tests, there were no issues found with the gems.

## Merging in changes that come with rails 6.0.

Running `bundle exec rails app:update` changed a bunch of files and created a file under `config/initializers/new_framework_defaults_6_0.rb` which includes the new defaults for rails 6.0.
The file changes needed to be merged in after the update task.
You can find the final pull request to upgrade rails 6.0 at [https://github.com/lobsters/lobsters/pull/723](https://github.com/lobsters/lobsters/pull/723).
After merging the rails 6.0 change, we had to [enable all the options](https://github.com/lobsters/lobsters/issues/724) in production
Finally we were able to get rid of the file and call `config.load_defaults 6.0` in config/application.rb.

## Conclusion

Lobste.rs is now running rails 6.0 without a hitch!
