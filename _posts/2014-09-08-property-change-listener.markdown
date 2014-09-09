---
layout: post
title:  "Model Binding for Android"
date:   2014-09-08 20:50
categories: "Android"
author: Christoph Portmann
---

##Introduction
While developing the Android Client for a platform that provides near-realtime experience for interactions between users,
we had to implement a binding between the Model and the UI. During the development we created and evolved our own system,
but we ended up with something very similar to [Java Bean's PropertyChangeListener](http://developer.android.com/reference/java/beans/PropertyChangeListener.html).
To find a better approach, I created a small setup consisting of a model "Profile" and a UI reflecting a profile's properties
(such as name, age, followers). Inside this setup I implemented the different solutions and trialed them in a predefined
set of tests.  
Unfortunately I didn't find time to finish a version with an Eventbus such as [Square's Ottobus](http://square.github.io/otto/)
, I may add this in the future.

##Approaches

###Overview

+ Original: Project's implementation as-is
+ PropertyChangeListener: the java.beans solution + a modified version that notifies on UI-Thread
+ Property Class: Properties are replaced by a Wrapper-class with Observable-feature
+ Proxy solution: Java’s dynamic proxies is used for notification in a AOP-like manner

### Project's solution
Our own solution is very similar to PropertyChangeListener, a FieldObserver can directly bind to a FieldObservable Model's Field.
FieldObserver and Field are generic and provide type-safety.  
Further it provides Thread-safety and updates the Observers directly on the UIThread which allows us to make updates from
any thread.  
The disadvantages are the huge boilerplate when creating a new model, as well as
 the high complexity of the implementation. Therefore this implementation needs some serious refactoring, which will
 probably never happen, since the app is not under active development anymore.

[Code example on Gist](https://gist.github.com/chrisport/748ba6d5370f769e58b6)

Let's take a look how we implement the Profile:
{% highlight java %}
public interface Profile extends AbstractFieldsObservable {
    public static final Field<String> FIRST_NAME = new Field("firstName");
    private String firstName;
        
    public void setFirstName(String firstName) {
        this.firstName = firstName;
        notifyFieldObservers(FIRST_NAME, firstName);
    }
    
    public String getFirstName(){
        return firstName;
    }
}
{% endhighlight %}
As you can see, we were using an abstract class to inherit the common functions. Refactoring this to a component would make Profile implementation look like the next one, 
PropertyChangeListener.

### PropertyChangeListener

This implementation is based on Java Bean's PropertyChangeListener. There are basically two versions under test,
the original one and a slightly modified one.
In the modified version the Listeners are notified on the UI-Thread 
which allows us to call the model's update-method from a background thread.

{% highlight java %}
    public void firePropertyChange(final PropertyChangeEvent event) {
        ...
                uiHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        finalP.propertyChange(event);
                    }
                });
      }
{%endhighlight%}

**Profile implementation** looks like this:

{% highlight java %}
public class ProfilePropChangeListenerOwn implements Profile {
    private PropertyChangeSupport changeSupport = new PropertyChangeSupport(this);
    private String name = null;
 
    public String getName() {
        return name;
    }
 
    public void setName(final String firstName) {
        this.name = firstName;
        changeSupport.firePropertyChange("firstName", name, firstName);
    }
 
    public void addPropertyChangeListener(
            PropertyChangeListenerOwn l) {
        changeSupport.addPropertyChangeListener(l);
    }
 
    public void removePropertyChangeListener(
            PropertyChangeListenerOwn l) {
        changeSupport.removePropertyChangeListener(l);
    }
}
{%endhighlight%}

### Property Class

The property class is a generic class, that holds its own listener. The Observer pattern is therefore moved from the model
to its properties. 

[Code example on Gist](https://gist.github.com/chrisport/386cc4c82d7caceb02f0)

**Profile implementation** now gets really comfortable:

{% highlight java%}
public class ProfilePropsObserver implements Profile {
    public final Property<String> name = new Property<String>();
}
{% endhighlight %}
Done. This is a very solid solution and let's you implement new models lighting fast. The disadvantage of this solution
may be the high number of small object that are produced, which shouldn't be a problem in most context. Further it is 
hard to acquire information about who is listening to a model, since the listeners are spread over the
model's property

### Dynamic Proxy solution

The most advanced and interesting solution includes the usage of Java's dynamic proxy. A similar approach is also used by
 the widespread Spring framework.  
 The dynamic proxy stores properties according to the called "set"-method in a Map. Therefore "setFirstname" will store
the provided as value of the key "firstname". Further it implements "get" and "is", as well as addObserver. RemoveObserver
is not implemented, since it is not used by the tests. The full code of the Proxy can be found  

[Code example on gist](https://gist.github.com/chrisport/c2780eff8fa234087751)

This is the **interface of Profile**:
{% highlight java %}
public interface Profile {
    public final static String name = "name";
    public String getName();
    public void setName(String firstName);
}
{%endhighlight%}

Creation of a new Profile will in this case look as follow:

{% highlight java %}
profile = (ProfileP) PojoProxy.newInstance(new Class[]{ProfileP.class}, this);
{%endhighlight%}

**Note:** To use Dynamic Proxy on Android, the ClassLoader must be obtained via a context: context.getClassLoader()

For more information see also ["Java Reflection - Dynamic Proxies" by Jakob Jenkov](http://tutorials.jenkov.com/java-reflection/dynamic-proxies.html)

## Comparison

All test are run on a LG Nexus 4. There are two profile models that are modified during the test.  
The time measurement
ends when all changes have been applied (using Timelatch). 
The times below represent the average time per set and is calculated by:
```(Total time) / (number of setter-calls)```

UI-Thread: 24’000 setter-calls<br>
Singlethread: 24’000 setter-calls in background thread<br>
Multithread: 3 Threads à 8'000 setter-calls<br>

| Approach                      | UI-Thread  | Multithread | GC runs |
|----------------------------   |: --------------|: -------------|: -------------|: -------------|
| Project solution              | 28 μs      | 40 μs       | 20 |
| PropertyChangeListener        | 38  μs     |             | 23 |
| PropertyChangeListener (mod)  | 31 μs      | 44 μs       | 23 |
| Property Class                | 33 μs      | 40 μs       | 20 |
| Dynamic Proxy                 | 85 μs      | 93 μs       | 45 |

## Conclusion

Please note the **performance penalty** of multithreaded runs, which are caused by the low priority of background threads.
The custom implementation seems to have a slight performance advantage. But the differences are small. 
Only dynamic proxy takes significantly more time and causes more GC-runs than the other solutions.
I suspect the overhead of the reflective method lookup causes this performance cost.

From implementation perspective the Property Class is the easiest to implement and use. But the fact that observers
are stored for every property can be a big disadvantage.  
The Dynamic Proxy provides also an easy-to-use solution, but comes with a overhead on runtime. In our case this is the
solution I would go today, because it gives us further possibility to interfere interaction with the model. 


