---
title: "Rubygems.org: Upgrading Rails to 5.2"
---

## Overview

After [upgrading lobste.rs](/posts/2018-08-21-lobsters-upgrading-rails.html), I decided to upgrade [rubygems.org](rubygems.org).
I wanted to contribute because I felt that the lobste.rs upgrade went relatively smoothly and I felt that I can take on another similar project.
I looked through the codebase and the setup was very similar to lobste.rs; it included a good testsuite, and it was on rails 5.1 so it only needed a minor bump.

## Planning

Since I previously performed a successful rails 5.1 to 5.2 upgrade, I wasn't too worried about planning.
The plan I had was to just perform the initial upgrade and fix any test failures.

## Beginning the upgrade

Just like I did with the lobste.rs upgrade, I started by upgrading the rails gem.
The next step was to run `bundle exec rails app:update` to get the latest configurations available from rails 5.2.
While merging back all the changes that were reverted by running the update command, I did a commit per file to give me checkpoints along the way.
In my opinion, each file change should be a commit so that the history could be easily navigated.

## Test failures

After performing the initial upgrade, I ran the tests and was welcomed with a lot of test failures.
I got a little bit discouraged by the upgrade at this point.
But at second glance, the error messages all seemed related to each other.
This brought some relief to my first take on the upgrade since similar errors usually mean that there is 1 underlying issue that you need to fix.
After debugging the code, I found that the tests rely on setting `RAW_POST_DATA` in the test but in the controllers it was passed in as `nil`.
I then stepped through the code using the very useful tool [pry-byebug](https://github.com/deivid-rodriguez/pry-byebug).
This led me to [scrub_env!](https://github.com/rails/rails/blob/v5.2.1/actionpack/lib/action_controller/test_case.rb#L601-L610) which deleted `RAW_POST_DATA`.
After git blaming, I found that it was related to [https://github.com/rails/rails/pull/32673](https://github.com/rails/rails/pull/32673).
So the cause of the test failures was due to rails 5.2.1 breaking backwards compatibility!
Luckily the [fix for the break was easy](https://github.com/rubygems/rubygems.org/pull/1771/commits/c01be79670dd2323ab9abf6f202fa23d4970dcaa).

## Wrapping up the upgrade

After fixing all the tests there was still work to be done.
This included cleaning up the [deprecation noise](https://github.com/rubygems/rubygems.org/pull/1771/commits/75523b6275d6ce7d9c6fc008c3273d54fb01c2cf) to future proof the code.
I also encountered more test failures, but this was due to rubocop, so I went ahead and cleaned those up [1](https://github.com/rubygems/rubygems.org/pull/1771/commits/2741a0ea740ce1b2f77ef797c791ba6479a92b21) [2](https://github.com/rubygems/rubygems.org/pull/1771/commits/19434e0986911026419f9f2570791e99d9cda446).
Finally, just like the rails upgrade for lobste.rs, I found out that the previous default configuration options didn't get migrated.
I opened an [issue](https://github.com/rubygems/rubygems.org/issues/1773) to track this for future work.

## Conclusion

The [pull request](https://github.com/rubygems/rubygems.org/pull/1771) is still open, but I consider this a success.
The initial upgrade went as expected, just like the lobste.rs upgrade.
The part that took the longest was finding out about the rails 5.2.1 backwards compability break.
Luckily after finding the root cause it was relatively easy to fix.
One takeaway that I have is that people seem to forget to migrate to the new default settings when upgrading rails.
This isn't a required step, but in my opinion it should be required before you consider a rails upgrade complete.
Not performing the default configurations migration might catch some people off guard.
When you're using rails 5.2, you expect it to have the same behavior as a new rails 5.2 application.
