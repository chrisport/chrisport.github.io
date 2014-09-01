---
layout: post
title:  "The Art of Redundancy"
date:   2014-09-01 19:34:00
categories: Jekyll checkout
---
Today we take a look at **the Art of Redundancy**. 
Redundancy is not the simple result of copy&paste-coding, it must be grown slowly by a skilled developer. Let us illustrate this on a simple example.
{% highlight java %}
public boolean isTrue(boolean aBool) throws IllegalArgumentException {
  if (aBool) {
    return true;
  } else if (!aBool) {
    return false;
  } else {
    throw new IllegalArgumentException("Unexpected boolean value");
  }
}
{% endhighlight %}
As you can see this triple-inception-method covers different varieties of redundancy:

- An obsolete method: doing a job that doesn't have to be done
- A conditional statement that returns the condition itself
- Unreachable code

So take a step back. Look at the code again. And enjoy life.

