# `Celery`-poc

# References

* [CFE - `Celery` + `Redis` + `Django`](https://www.codingforentrepreneurs.com/blog/celery-redis-django/)

## What this does

`Celery` unlocks a worker process for Django. This means you can offload tasks from the main request/response cycle within `Django`. Using `Celery` becomes critical when your app starts to scale or you need better performance out of `Django`.

`Django` is a batteries included web framework written in Python. `Django` is how you'll create a web application so users can leverage your software.

`Redis` is the datastore and message broker between `Celery` and `Django`. In other words, `Django` and `Celery` use `Redis` to communicate with each other (instead of a SQL database). `Redis` can also be used as a cache as well. An alternative for `Django` & `Celery` is RabbitMQ (not covered here).

All three work together to make some asynchronous magic.

## How to run it

1. Start `redis` in a local `docker` container:
    ```
    docker run -d --name redis-stack-server -p 6379:6379 redis/redis-stack-server:latest
    ```
1. Start `celery`:
    ```
    cd ~/git/celery-poc/src
    celery -A cfehome worker --beat -l info
    ```
    * Note the `cron` job running every five seconds: 
        ```
        [2023-03-29 18:48:02,970: INFO/Beat] Scheduler: Sending due task multiply-every-5-seconds (multiply_two_numbers)
        ```
1. Start a `django` interactive shell:
    ```
    python manage.py shell
    # In the django interactive shell
    >>> from movies.tasks import add, mul, xsum
    >>> add(1,3)
    >>>
    ```
1. Send the task to `celery`:
    ```
    >>> add.delay(1,3)
    <AsyncResult: b148e157-f4dc-4687-a3bf-975a40deeb0d>
    ```
    * Note the `AsyncResult` output
    * Note the log entry in `celery`:
    ```
    [2023-03-29 18:55:03,242: INFO/MainProcess] Task movies.tasks.add[05c986d8-1bd7-4bf7-8de5-358d9ceb7dbd] received
    [2023-03-29 18:55:03,248: INFO/ForkPoolWorker-9] Task movies.tasks.add[05c986d8-1bd7-4bf7-8de5-358d9ceb7dbd] succeeded in 0.005537832999834791s: 4
    ```
