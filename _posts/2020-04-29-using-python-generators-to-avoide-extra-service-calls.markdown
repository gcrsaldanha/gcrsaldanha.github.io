---
layout: post
title:  "Using Python Generators to avoid extra service calls"
categories: python programming tips
---
This post is also available on [dev.to](https://dev.to/gcrsaldanha/using-python-generators-to-avoid-extra-service-calls-pl), and is my first public post on the Internet that is not a tweet. Feedbacks are very welcome! :)

---
I've been using Python Generators for a while now, having to deal with large `Django Querysets`, reading large Excel files, and being able to save memory by using generators became part of my day to day work. However, these days I faced a different problem where I wanted to avoid making extra calls to services and solved it with generators. This is what this post is about.

Suppose you want to check if a given `user_email` is registered in **any** of the following social media: Facebook, Github, or Twitter.

```python
def has_facebook_account(user_email):
    print('calling Facebook service')
    return False

def has_github_account(user_email):
    print('calling Github service')
    return True

def has_twitter_account(user_email):
    print('calling Twitter service')
    return True

def has_social_media_account(user_email):
    print('Checking social media apps...')
    # Python's `any` receives an Iterable
    # and returns True when the first truthy clause is found.
    response = any([
        has_facebook_account(user_email),  # This is False
        has_github_account(user_email),  # This is True
        has_twitter_account(user_email),  # This is True
    ])
    print('Done!')
    return response
```
What is wrong with the approach above? Well, if you run it locally you will see the following output:
```shell
>>> has_social_media_account('fake@email.com')
Checking social media apps...
calling Facebook service  # This is False, keep going...
calling Github service  # This is True! I can stop now...
calling Twitter service  # Oh no!
Done!
```
The problem is that a `list` is evaluated as soon as it is created (opposite to *lazy evaluation*). Thus, if it's a list of method calls, all the calls will be performed.

At the moment, I thought the problem was that I was actually *calling* the methods when initializing the list. So maybe keeping only the methods references and using a *list comprehension* would do the trick:
```python
def has_social_account(user_email):
    calls = [has_facebook_account, has_github_account, has_twitter_account]  # Refs
    return any([call(user_email) for call in calls])
```
Well, it did not work as I wanted either. As I said before, `a list is fully evaluated upon its creation`. In the example above, it will first make sure to call all methods so the list is *fully built*, having all of its elements evaluated - which in turn means having all methods being called. The list comprehension is not aware of its context either, i.e., it does not know that it is being used as an argument to `any(...)`.

How did I solve it? By using generator.
```python
def has_social_account(user_email):
    calls = [has_facebook_account, has_github_account, has_twitter_account]
    return any((call(user_email) for call in calls))  # Note the ( ) instead of [ ]

>>> has_social_account('fake@email.com')
calling Facebook service  # This is False, keep going...
calling Github service  # Aw yeah!
```
`(call(user_email) for call in calls)` evaluates to a `generator` (not a fully built list), which is then used by `any`. `any(...)` will *iterate* over the generator elements evaluating **one at a time**. This way, no extra calls are being made for once `any` evaluates the second element (`has_github_account(user_email) -> True`), the `any` function is evaluated itself, returning `True` and not calling the third service method (`has_twitter_account`).

---

[Here is the Gist if you're interested](https://gist.github.com/gcrsaldanha/dfdf007cc015a4a36ccddef88def617a).

Please let me know if something is not clear and/or if you'd like to have a more in-depth article on Python Generators.

You can always reach out to me on [Telegram](https://t.me/gcrsaldanha)! Or on any  social media I go by @gcrsaldanha.