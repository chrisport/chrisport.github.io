---
layout: post
title:  "Context Aware Java Executor and Spring's @Async"
date:   2016-10-13 11:00
categories: "Java"
github: https://github.com/chrisport/thread-context-demo
author: Christoph Portmann
---
Recently I came across the problem of ThreadLocal context in multithreaded environment.
If the Thread that handled a request uses an [Executor](https://docs.oracle.com/javase/tutorial/essential/concurrency/exinter.html) 
to asynchronously execute tasks, the ThreadLocal context is lost and information such as requestId will be be missing in the logs.    
We can solve this using the Decorator pattern:   

1. Wrap Runnable to preserve caller Thread's context
2. Implement ExecutorDecorator that wraps every execution  
3. Additional: Configure Spring's @Async to use Decorator 

Please note:   
- [*MDC* (Mapped Diagnostic Context)](http://www.slf4j.org/api/org/slf4j/MDC.html) used in the example as the ThreadLocal context      
- *caller Thread*, which wants to schedule some task   
- *executor Thread*, which can be any Thread (e.g. from a pool) that executes the task   

### 1. Make the runnable context aware

To pass the MDC of the caller Thread to the Executor thred, we get a copy of the ContextMap.  
Then we set this context map before execution of the task and reset it after the execution.      

{% highlight java %}
 
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

{% endhighlight %}


### 2. Implement custom Executor that decorates every task


{% highlight java %}
  
public class ContextAwareExecutorDecorator implements Executor, TaskExecutor {

    private final Executor executor;

    public ContextAwareExecutor(Executor executor) {
        this.executor = executor;
    }

    @Override
    public void execute(Runnable command) {
        // decorate the task
        Runnable ctxAwareCommand = wrapContextAware(command);
        // execute the decorated task
        executor.execute(ctxAwareCommand);
    }

  
    private Runnable wrapContextAware(Runnable command) {
        ... see above
        return ctxAwareTask;
    }
}

{% endhighlight %}


Please note that I also implement Spring's TaskExecutor for usage below.

### 3. Addition: Configure Spring's @Async to use Decorator
 
We can configure the default executor easily with the following Configuration.
See also [Spring's documentation: EnableAsync](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/annotation/EnableAsync.html).

{% highlight java %}

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

{% endhighlight %}

