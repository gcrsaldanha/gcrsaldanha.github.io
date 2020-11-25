---
layout: post
title:  "How to write concurrent code using Python's Future"
categories: python asynchronous concurrency howto
---
# Python futures (concurrency)

tl;dr;

> Using concurrency to speed up things is quite simple in Python using the `concurrent.futures` module. However, it's no silver bullet and one must know **when** to use it.

---

## All started with generators...

In a [previous post](https://blog.saldanha.dev/python/programming/tips/2020/04/29/using-python-generators-to-avoide-extra-service-calls.html) I mentioned how you could use Python generators as a way to avoid performing extra service calls, saving up some time and resources. Here is a summary:

```python
# Suppose you want to check if a given user_email is registered in 
# any of the following social media: Facebook, Github, or Twitter.

def has_facebook_account(user_email):
    ...

def has_github_account(user_email):
    ...

def has_twitter_account(user_email):
    ...

def has_social_account(user_email):
    calls = [
        has_facebook_account,
        has_github_account,
        has_twitter_account,
    ]
    return any((call(user_email) for call in calls))
    # (expr for i in iterable) syntax denotes a generator comprehension
```

Pretty simple, huh? `any` iterates and evaluates each `call(user_email)` yield from the generator until one of them returns `True`. This is known as *early return* — you basically save up some time and resources by not performing unnecessary calls.

Some folks gave nice feedback mentioning that I should make it **concurrent**, i.e., I could make **all the calls** concurrently and *early return* as soon as any *call* returned True. "That's a good idea", I thought. I'm glad there are smarter people than myself out there.

If that's not clear *why* I would want to do it: suppose `has_facebook_account` takes too long to run (as usually happens with any I/O and network operations due to high latency) and `has_github_account` is pretty fast (maybe it's cached, for example). I would **always** need to wait for `has_facebook_account` return **before** calling `has_github_account` since the generator's items would be evaluated **orderly**. That does not sound fun.

## Make it concurrent!

I am using [Python's `concurrent.futures` module](https://docs.python.org/3/library/concurrent.futures.html) (available since version 3.2). This module consists basically of two entities: the `Executor` and the `Future` object. You should read this documentation, it is really short and straight to the point.

The `Executor` *abstract class* is responsible for *scheduling* a task (or *callable*) to be executed *asynchronously* (or *concurrently*). Scheduling a task returns a `Future` object, which is a reference to the task and represents its state — *pending, finished, or canceled*.

If you have ever worked with *JavaScript Promises* before, `Future` is very similar to a `Promise`: you know its execution will *eventually* be done, but you can not know when. The nice thing is: it is **non-blocking code**, meaning the Python interpreter does not have to wait until the scheduled task's execution finishes before running the next line of code.

Thus, in our scenario, we could *schedule* three tasks, one for querying each platform (Facebook, GitHub, and Twitter) for a user email address. This way, once any of these tasks *eventually* returns a value, I can *early return* if the value is `True`, since all we want to know is if the user has an account in any of these platforms.

## Talk is cheap. Show me the code.

The example code below is easy to follow but I strongly suggest you read the commented lines.

```python
import time  # I will use it to simulate latency with time.sleep
from concurrent.futures import ThreadPoolExecutor, as_completed

def has_facebook_account(user_email):
    time.sleep(5)  # 5 seconds! That is bad.
    print("Finished facebook after 5 seconds!")
    return True

def has_github_account(user_email):
    time.sleep(1)  # 1 second. Phew!
    print("Finished github after 1 second!")
    return True

def has_twitter_account(user_email):
    time.sleep(3)  # Well...
    print("Finished twitter after 3 seconds!")
    return False

# Main method that answers if a user has an account in any of the platforms
def has_social_account(user_email):
    # ThreadPoolExecutor is a subclass of Executor that uses threads.
    # max_workers is the max number of threads that will be used.
    # Since we are scheduling only 3 tasks, it does not make sense to have
    # more than 3 threads, otherwise we would be wasting resources.
    executor = ThreadPoolExecutor(max_workers=3)

    # Schedule (submit) 3 tasks (one for each social account check)
    # .submit method returns a Future object
    facebook_future = executor.submit(has_facebook_account, user_email)
    twitter_future = executor.submit(has_twitter_account, user_email)
    github_future = executor.submit(has_github_account, user_email)
    future_list = [facebook_future, github_future, twitter_future]

    # as_completed receives an iterable of Future objects
    # and yields each future once it has been completed.
    for future in as_completed(future_list):
        # .result() returns the future object return value
        future_return_value = future.result()
        print(future_return_value)
        if future_return_value is True:
            # I can early return once any result is True
            return True

user_email = "user@email.com"
if __name__ == '__main__':
    has_social_account(user_email)
```

```bash
Finished github after 1 second!
User has social account.
# The created threads will still run until completion
Finished twitter after 3 seconds!
Finished facebook after 5 seconds!
```

Notice that even though `facebook_future` takes longer than the other two scheduled tasks to finish, however, it does not **block** the execution — it keeps working on its own thread. And although `github_future` is the last scheduled task, it is the first to finish.

## Quick summary

- `Future` is an object that represents a scheduled task that will **eventually** finish.
- `Executor` is the scheduler of tasks (once a task is scheduled, it returns a `Future` object).
    - It can be a `ThreadPoolExecutor` or a `ProcessPoolExecutor` (using threads vs processes).
- One can use `executor.submit(callable)` to schedule a task to be run asynchronously.
- `as_completed` receives an iterable of `Future` objects and returns a generator where each yielded element is a **finished** task.

## When would I *not* want to use concurrency then?

As software engineers, our job is not only knowing **how** to use a tool, but also **when** to use it. Network operations (and I/O bound operations in general) are usually a good place to use concurrent code due to their latency. But there is always a trade-off...

In the example above we **traded off performance for resource usage.** How so? When using **generators**, only the **worst-case scenario** would end up consuming 3 services — one call for each `has_<plataform>_account`. That's because we could early return `True` if any service returned `True`.

In our new example using concurrency, we are **always** consuming the 3 service — since the calls are made *asynchronously*.

"Ah, but that still could save us lots of time!", you say. It depends on the **services** you're consuming. In the example above I artificially made the `has_facebook_account` ***really slow —*** 5 times slower than the fastest alternative. But, if all the services had a similar response time and if **saving resources** was important (suppose that calling each service would trigger a **really heavy query** in the database, for instance), using a *synchronous* code could be a better approach.

For the sake of data: [Facebook has over **2.7 billion monthly active users**](https://www.statista.com/statistics/264810/number-of-monthly-active-facebook-users-worldwide/#:~:text=How%20many%20users%20does%20Facebook,the%20biggest%20social%20network%20worldwide.), while [Twitter has around **330 million**](https://www.statista.com/statistics/282087/number-of-monthly-active-twitter-users/), and [GitHub has merely **40 million users**](https://en.wikipedia.org/wiki/GitHub#:~:text=As%20of%20January%202020%2C%20GitHub,source%20code%20in%20the%20world.). So, it is highly likely that calling the `has_facebook_account` first would be enough in a huge majority of scenarios since it would return `True` with a much higher frequency than the other services, thus, saving lots of unnecessary calls.

## Conclusion

Know how to write concurrent code, which is pretty easy with Python Futures. But more important: know **when** to do so. There are cases where the performance increase does not pay off the resource usage.

I strongly advise you to read the [docs on `concurrent.futures`](https://docs.python.org/3/library/concurrent.futures.html) and the [Chapter 17 on Luciano Ramalho's excellent Fluent Python book](https://amzn.to/33fS84W).

[Ah, and here is the gist if you want it!](https://gist.github.com/gcrsaldanha/2d4709f604212e4ebbd9b5e2dc9b4a67)