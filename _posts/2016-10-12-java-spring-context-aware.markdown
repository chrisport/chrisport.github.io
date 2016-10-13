---
layout: post
title:  "Context Aware Java Executor and Spring's @Async"
date:   2016-10-12 11:00
categories: "Java","Spring"
github: https://github.com/chrisport/thread-context-demo
author: Christoph Portmann
---
Recently I came across the problem of ThreadLocal Context in multithreaded environment, in this case when using Java's 
Executor interface with an underlying ThreadPool.   
The problem arises if the Thread that handled the request uses an Executor to asynchronously execute tasks. These may
be executed on any other Thread from a pool or by a newly created Thread. Any ThreadLocal/MDC of the handling Thread is
lost in this case and therefore information such as requestId is not present in the logs.   
I will solve this issue in 2 steps, using Decorator pattern:
1. Wrap the Runnable in order to pass a copy of the ThreadLocal context into the execution
2. Implement custom Executor that decorates every task using above wrapping

Additionally I will configure my Spring Boot environment to automatically use this Executor when wrapping methods that are
annotated with @Async.

In the following example I will use 
[*MDC*(Mapped Diagnostic Context)](http://www.slf4j.org/api/org/slf4j/MDC.html) from the package org.slf4j as the ThreadLocal Context. 
MDC can be used to store information in a ThreadLocal, particularly for additional logging information. 
For example request information, such as requestId can be put to MDC in order to be present on every log-message.

*Important:* Please note the difference between the *caller Thread*, which wants to schedule some task, 
and the *executor Thread*, which can be any Thread (e.g. from a pool) that executes the task.

### Make the runnable context aware

To pass the ThreadLocal context of the caller-Thread inside the task, we will first take a copy of it. 
Then we set this copy as executor-Thread's context before execution of the task and reset it after the execution. 
The result is simply another Runnable which can be executed by anybody, while MDC is ensured.   
So here's the code:

```
// get a copy of the values of calling Thread's MDC
final Map<String, String> callerContextCopy = MDC.getCopyOfContextMap();

Runnable ctxAwareTask = () -> {
  // get a copy of the values of executing-Thread's MDC
  final Map<String, String> executorContextCopy = MDC.getCopyOfContextMap();

  MDC.clear();
  if (callerContextCopy != null) {
    // set the desired context that was present at point of calling execute
    MDC.setContextMap(callerContextCopy);
  }

  // execute the command
  command.run();

  MDC.clear();
  if (executorContextCopy != null) {
    // reset the context
    MDC.setContextMap(executorContextCopy);
  }
};
```

### Implement custom Executor that decorates every task

There is not much to explain, here is the implementation.

``` 
public class ContextAwareExecutorDecorator implements Executor, TaskExecutor {

    private final Executor executor;

    public ContextAwareExecutor(Executor executor) {
        this.executor = executor;
    }

    @Override
    public void execute(Runnable command) {
        Runnable ctxAwareCommand = wrapContextAware(command);
        executor.execute(ctxAwareCommand);
    }

  
    private Runnable wrapContextAware(Runnable command) {
        ... see above
        return ctxAwareTask;
    }
}
```

Please note that I also implement Spring's TaskExecutor for usage below.

### Addition: Configure Spring's @Async to use Decorator

Following [Spring's documentation: EnableAsync](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/annotation/EnableAsync.html), 
we can configure the default executor easily with the following Configuration:

```
@Configuration
@EnableAsync
public class AppConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(7);
        executor.setMaxPoolSize(42);
        executor.setQueueCapacity(11);
        executor.setThreadNamePrefix("ContextAwareExecutor-");
        executor.initialize();
        return new ContextAwareExecutorDecorator(executor);
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        // TBD
        return new SimpleAsyncUncaughtExceptionHandler();
    }
}
```

