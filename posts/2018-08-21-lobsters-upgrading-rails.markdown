---
title: "Lobste.rs: Upgrading Rails to 5.2"
---

# Overview

I'm a frequent reader of [lobste.rs](https://lobste.rs), a link aggregation news site about computing.
It has a unique invite only signup method which I think keeps it very focused.
I recently discovered that the code for it is [available](https://github.com/lobsters/lobsters) on github.
After briefly looking through the code, I decided to contribute by shaving off the biggest yak I could find.
Upgrading the rails version from 5.1 to 5.2.
This post is about my experience performing that upgrade.

# Planning 

The first thing that I wanted to do was skim the codebase for anything that stuck out as a potential flag.
The directory layout seemed pretty standard, and it had tests!
It's always nice to see a decent test suite in a project when onboarding onto a new codebase.
It removes a lot of stress from worrying about breaking the codebase when you make a large change such as upgrading rails.
Another great thing about the codebase was that it was only 1 point release behind the latest version.
I've heard that you want to upgrade a rails codebase by incrementally upgrading through all the minor, then major versions until you reach the latest version.

In addition to familiarizing myself with the general structure and quality of the codebase, I also went through a couple of resources on upgrading rails.
Some great resources that I found were the
[official rails upgrade guide](https://guides.rubyonrails.org/upgrading_ruby_on_rails.html#upgrading-from-rails-5-1-to-rails-5-2), 
[rails 5.2 release notes](https://guides.rubyonrails.org/5_2_release_notes.html#upgrading-to-rails-5-2),
and a [blog post by Ombulabs](https://www.ombulabs.com/blog/rails/upgrades/upgrade-rails-from-5-1-to-5-2.html).

Once I was comfortable enough with all the concepts needed to upgrade a rails application, I went ahead and started working on it.

# First take

The first step whenever I'm working in an unfamiliar codebase is to get the test suite running.
After setting up mysql locally, and a couple of rails incantations to get the schema into the right state, I was able to get all the tests passing.
I later uncovered that lobste.rs used [MariaDB 10.1.35](https://github.com/lobsters/lobsters/issues/529#issuecomment-413188718) while I set up Mysql 5.7.
This turned out to be a fortunate accident because this uncovered an [issue](https://github.com/lobsters/lobsters/issues/529) with some of the sql that was written previously.
I later fixed this issue by correcting the sql.

Once I had all the tests passing, I updated rails in the Gemfile and ran `bundle update rails` to perform the gem upgrade.
The next step that the upgrade guides mentioned was to run `bundle exec rails app:update` to update automatically generated files and bring in new configurations.
Running the rails update command changed *a lot* of automatically generated files.
From removing all the routes in the routes.rb file to style changes across a dozen or more files.
I recommend using a version control system to check in each change you make after every command to keep things organized.
It will make sure that you don't lose any reasons why the massive rails upgrade went the way it did.
The original upgrade work can be found at [PR 498](https://github.com/lobsters/lobsters/pull/498).

# Bootsnap Hiccup

One issue we encountered during the upgrade was when we tried to enable bootsnap.
Bootsnap is a new default feature in rails 5.2 and it advertises that it helps decrease boot times by caching expensive computations.
After enabling this, there was an issue booting the application due to some kind of file system permissioning issue related to bootsnap [issue 77](https://github.com/Shopify/bootsnap/issues/77).
The feature was reverted because lobste.rs is a small app and already has a relatively quick boot time.

# The Long Run

By far, the most annoying task was reading about the new configuration options in `config/initializers/new_framework_defaults_5_2.rb`.
This file was automatically generated and it contains several options that need to be enabled one by one.
It's recommended that after enabling each option, you restart the server and in some cases you might want to give it some time before continuing the upgrade to make sure everything is ok.
This would have normally been not as bad as it was, but I didn't have access to the server and merges needed to be approved each time.

The final one of these options looked particularly important, so this was the last option that we turned on before I considered the upgrade complete.

``` ruby
# Use AES-256-GCM authenticated encryption for encrypted cookies.
# Also, embed cookie expiry in signed or encrypted cookies for increased security.
#
# This option is not backwards compatible with earlier Rails versions.
# It's best enabled when your entire app is migrated and stable on 5.2.
#
# Existing cookies will be converted on read then written with the new scheme.
Rails.application.config.action_dispatch.use_authenticated_cookie_encryption = true
```

# Cleaning Up

The remaining work was to cleanup the intermediate new framework defaults config files that rails automatically generated and put `config.load_defaults 5.2` in `config/application.rb`.

# Conclusion

I am finally happy with the rails 5.2 upgrade for lobste.rs.
The majority of the work was reading about how to perform the upgrade and then following through.
There were some documentation bugs and lobste.rs bugs, but I tried to fix the majority of them and I documented the remaining known issues.
This upgrade should allow further enhancements like adding a [content security policy](https://github.com/lobsters/lobsters/issues/486).

# Lobste.rs is now running rails 5.2!
