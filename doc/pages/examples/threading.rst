Threading
=========

The :class:`~scheduler.core.Scheduler` is thread safe and supports parallel execution
of pending :class:`~scheduler.job.Job`\ s.
:class:`~scheduler.job.Job`\ s with a relevant execution time or blocking IO operations
can delay each other.
The following examples show the difference between concurrent and parallel
:class:`~scheduler.core.Scheduler`\ s:

Concurrent execution
--------------------

By default the :class:`~scheduler.core.Scheduler` will execute its
:class:`~scheduler.job.Job`\ s sequentially. The total duration when executing multiple
:class:`~scheduler.job.Job`\ s will therefore be greater than the sum of the individual
run times.

.. code-block:: pycon

    >>> import time
    >>> import datetime as dt
    >>> from scheduler import Scheduler

    >>> def sleep(secs: float):
    ...     time.sleep(secs)

    >>> sch = Scheduler()
    >>> job_1 = sch.once(dt.timedelta(), sleep, params={"secs": 0.1})
    >>> job_2 = sch.once(dt.timedelta(), sleep, params={"secs": 0.1})

    >>> start_time = time.perf_counter()
    >>> n_exec = sch.exec_jobs()
    >>> total_seconds = time.perf_counter() - start_time
    >>> n_exec
    2

    >>> 0.2 < total_seconds and total_seconds < 0.21
    True

Parallel execution
------------------

The number of worker threads for the :class:`~scheduler.core.Scheduler` can be defined
with the `n_threads` argument. For ``n_threads = 0`` the :class:`~scheduler.core.Scheduler`
will spawn a seperate worker thread for every pending `Job`.

.. code-block:: pycon

    >>> import time
    >>> import datetime as dt
    >>> from scheduler import Scheduler

    >>> def sleep(secs: float):
    ...     time.sleep(secs)

    >>> sch = Scheduler(n_threads=0)
    >>> job_1 = sch.once(dt.timedelta(), sleep, params={"secs": 0.1})
    >>> job_2 = sch.once(dt.timedelta(), sleep, params={"secs": 0.1})

    >>> start_time = time.perf_counter()
    >>> n_exec = sch.exec_jobs()
    >>> total_seconds = time.perf_counter() - start_time
    >>> n_exec
    2

    >>> 0.1 < total_seconds and total_seconds < 0.11
    True

.. warning:: When running `Job`\ s in parallel, be sure that possible side effects
    of the scheduled functions are implemented in a thread safe manner.