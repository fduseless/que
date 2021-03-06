## Command Line Interface

```
usage: que [options] [file/to/require] ...
    -h, --help                       Show this help text.
    -i, --poll-interval [INTERVAL]   Set maximum interval between polls for available jobs, in seconds (default: 5)
    -l, --log-level [LEVEL]          Set level at which to log to STDOUT (debug, info, warn, error, fatal) (default: info)
    -p, --worker-priorities [LIST]   List of priorities to assign to workers (default: 10,30,50,any,any,any)
    -q, --queue-name [NAME]          Set a queue name to work jobs from. Can be passed multiple times. (default: the default queue only)
    -w, --worker-count [COUNT]       Set number of workers in process (default: 6)
    -v, --version                    Print Que version and exit.
        --connection-url [URL]       Set a custom database url to connect to for locking purposes.
        --log-internals              Log verbosely about Que's internal state. Only recommended for debugging issues
        --maximum-buffer-size [SIZE] Set maximum number of jobs to be locked and held in this process awaiting a worker (default: 8)
        --minimum-buffer-size [SIZE] Set minimum number of jobs to be locked and held in this process awaiting a worker (default: 2)
        --wait-period [PERIOD]       Set maximum interval between checks of the in-memory job queue, in milliseconds (default: 50)
```

Some explanation of the more unusual options:

### worker-priorities and worker-count

These options dictate the size and priority distribution of the worker pool. The default worker-priorities is `10,30,50,any,any,any`. This means that the default worker pool will reserve one worker to only works jobs with priorities under 10, one for priorities under 30, and one for priorities under 50. Three more workers will work any job.

For example, with these defaults, you could have a large backlog of jobs of priority 100. When a more important job (priority 40) comes in, there's guaranteed to be a free worker. If the process then becomes saturated with jobs of priority 40, and then a priority 20 job comes in, there's guaranteed to be a free worker for it, and so on. You can pass a priority more than once to have multiple workers at that level (for example: `--worker-priorities=100,100,any,any`). This gives you a lot of freedom to manage your worker capacity at different priority levels.

Instead of passing worker-priorities, you can pass a `worker-count` - this is a shorthand for creating the given number of workers at the `any` priority level. So, `--worker-count=3` is just like passing equivalent to `worker-priorities=any,any,any`.

If you pass both worker-count and worker-priorities, the count will trim or pad the priorities list with `any` workers. So, `--worker-priorities=20,30,40 --worker-count=6` would be the same as passing `--worker-priorities=20,30,40,any,any,any`.

### poll-interval

This option sets the number of seconds the process will wait between polls of the job queue. Jobs that are ready to be worked immediately will be broadcast via the LISTEN/NOTIFY system, so polling is unnecessary for them - polling is only necessary for jobs that are scheduled in the future or which are being delayed due to errors. The default is 5 seconds.

### minimum-buffer-size and maximum-buffer-size

These options set the size of the internal buffer that Que uses to hold jobs until they're ready for workers. The default minimum is 2 and the maximum is 8, meaning that the process won't buffer more than 8 jobs that aren't yet ready to be worked, and will only resort to polling if the buffer dips below 2. If you don't want jobs to be buffered at all, you can set both of these values to zero.

### connection-url

This option sets the URL to be used to open a connection to the database for locking purposes. By default, Que will simply use a connection from the connection pool for locking - this option is only useful if your application connections can't use advisory locks - for example, if they're passed through an external connection pool like PgBouncer. In that case, you'll need to use this option to specify your actual database URL so that Que can establish a direct connection.

### wait-period

This option specifies (in milliseconds) how often the locking thread wakes up to check whether the workers have finished jobs, whether it's time to poll, etc. You shouldn't generally need to tweak this, but it may come in handy for some workloads. The default is 50 milliseconds.

### log-internals

This option instructs Que to output a lot of information about its internal state to the logger. It should only be used if it becomes necessary to debug issues.
