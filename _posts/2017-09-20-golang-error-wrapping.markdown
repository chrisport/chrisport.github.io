---
layout: post
title:  "Bubble up errors in Golang"
date:   2017-09-20
categories: "Golang"
author: Christoph Portmann
status: "released"
---
<Draft>
One problem I often encounter, are errors without any context.
If an arbitrary method returns an error "invalid character '}' after object key", you will have to dive down the stack until you find
the exact place the error happened. Therefore whenever you receive an error and you decide to not handle it, but instead
return it to the caller of your method, you should provide context about the reason the error occurred.

Let's take the following method as example:
{% highlight go %}
func main() {
	err := process([]byte("some_invalid_input"))
	if err != nil {
		panic(err)
	}
}

func process(input []byte) error {
	r, err = serviceA.Process(input)
	if err != nil {
		return err
	}

	err = serviceB.Process(r)
	if err != nil {
		return err
	}

	return nil
}
{% endhighlight %}

If serviceA or serviceB returns an error, we don't know which one failed and we don't know wether it is our fault
or maybe the service is unreachable or misconfigured?
The error could look like this:
{% highlight go %}
panic: invalid character '}' looking for beginning of value

goroutine 1 [running]:
main.main()
	/Users/christophportmann/workspace/go/src/github.com/chrisport/go-playground/errorhandling/main.go:10 +0x92
exit status 2
{% endhighlight %}

By wrapping the error, we can provide the missing information about what part failed:
{% highlight go %}
func main() {
	err := process([]byte("some_invalid_input"))
	if err != nil {
		panic(err)
	}
}

func process(input []byte) error {
	r, err = serviceA.Process(input)
	if err != nil {
		return errors.Wrap(err,"call to service A failed")
	}

	err = serviceB.Process(r)
	if err != nil {
		return errors.Wrap(err,"call to service B failed")
	}

	return nil
}
{% endhighlight %}

The result would be unambigous:
{% highlight go %}
panic: calling service B with provided input failed: invalid character '}' looking for beginning of value

goroutine 1 [running]:
main.main()
	/Users/christophportmann/workspace/go/src/github.com/chrisport/go-playground/errorhandling/main.go:11 +0x92
exit status 2
{% endhighlight %}


In this simple example the advantage is not crystal clear. But imagine serviceA is doing a complex task itself and doesn't wrap it's error as well.   
You could end up getting generic errors (e.g. a timeout on an http call) and need to debug down the whole stack to find out which
call has failed. But if errors are always wrapped with necessary context, an error message could look as nice as this:   
{% highlight go %}
panic: calling service B with provided input failed: call to backend failed: http: wrote more than the declared Content-Length
{% endhighlight %}
   
Therefore <b>if you decide to bubble up an error, always provide context</b>.
