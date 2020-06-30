# Task Scheduling

- [Introduction](#introduction)
- [Defining Schedules](#defining-schedules)

<a name="introduction"></a>
## Introduction

Scheduling lets you run functions (or any other callable) periodically at pre-determined intervals.

<a name="defining-schedules"></a>
## Defining Schedules

    import hunt.concurrency.Executors;
    import hunt.concurrency.Scheduler;
    import hunt.concurrency.ScheduledThreadPoolExecutor;
    import core.time;
    import hunt.util.Common;

    // Get executor
    ScheduledThreadPoolExecutor executor = cast(ScheduledThreadPoolExecutor)Executors.newScheduledThreadPool(8);// Use 8 threads to process

    // Create a task
    executor.scheduleAtFixedRate(
      new class Runnable {
        void run() {
          // Do something

        }
      }
    , 1.seconds, 600.seconds); // Delayed execution by 1 second, repeated execution at 600 second intervals

    /**
    * Initiates an orderly shutdown in which previously submitted
    * tasks are executed, but no new tasks will be accepted.
    * Invocation has no additional effect if already shut down.
    **/
    executor.shutdown();
    
## Parameters

    ScheduledFuture!void scheduleAtFixedRate(Runnable command,Duration initialDelay,Duration period) 

    command : The job functions
    initialDelay : The delay execution time
    period : intervals
    
##  Duration defining

    alias weeks   = dur!"weeks";   /// Ditto
    alias days    = dur!"days";    /// Ditto
    alias hours   = dur!"hours";   /// Ditto
    alias minutes = dur!"minutes"; /// Ditto
    alias seconds = dur!"seconds"; /// Ditto
    alias msecs   = dur!"msecs";   /// Ditto
    alias usecs   = dur!"usecs";   /// Ditto
    alias hnsecs  = dur!"hnsecs";  /// Ditto
    alias nsecs   = dur!"nsecs";   /// Ditto
    
    
