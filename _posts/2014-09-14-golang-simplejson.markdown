---
layout: post
title:  "Simple JSON in Golang"
date:   2014-09-14 22:25
categories: "Golang"
github: https://github.com/chrisport/simplejson
author: Christoph Portmann
---
Simplejson is an imitation of Java's json-simple for go language. It allows to access json-data without the overhead of
predefining a model or casting elements manually. Therefore it is useful, whenever you want to get data from different schemas
in a quick and clean way. It additionally provides a handy Error-handling, as well as segmented keys to directly access
nested Objects and Arrays. Current Version is 0.3 and its API will change slightly to allow a better error handling in the next version.

### Context

While developing a configuration reader, I came across the problem: how to generically access data in a nested json-files in golang.
Golang's native json-package provides methods to access data in json-format by unmarshaling into a predefined struct. 
Therefore to handle various config-files, every project had to define it's own config-model that is used
to unmarshall data and this was a pain.  
If the type is not clear you are always free to parse it to a map[string] interface{}, but it means you need to 
 cast the elements you access on this map (except they are all of same type, e.g. map[string] string)
But when you have nested data, it gets a pain quickly, because in every step you need to cast the elements.  
Let's look at an example.
Json config file:
{% highlight json %}
{
    "routes": [
        {
            "path": "profile/id/:id",
            "url": "https://api.eventmanager.com/events",
            "method": "GET"
        },
        {
            "path": "events/id/: id",
            "url": "http: //api.eventmanager.com/events",
            "method": "GET"
        }
    ],
    "adminMode": true
}
{% endhighlight %}

### Built-in possibilities to access
To access the "path"-attribute of the first route, there is the first solution including casting:

{% highlight go %}

// defined the base map
var rootMap map[string] interface{}
// unmarshal data into map
json.Unmarshal(data, &rootMap)
// access and cast until desired data is retrieved
routes := rootMap["routes"].([]interface{})
firstRoute := routes[0].(map[string]interface{})
path := firstRoute["path"].(string)

{% endhighlight %}

That's crazy, let's try another solution by defining a local struct:

{% highlight go %}
var localStruct struct{
  Routes []struct{
    Path string `json:"path"`
  } `json:"routes"`
}

json.Unmarshal(data, &localStruct)
pathOfFirstObject := localStruct.Routes[0].Path
{% endhighlight %}

That's a bit less crazy, but still quite crazy. And this doesn't make sense at all, if we need to write this for 
multiple items, then we can just define the model on its own.

### Imitation of simple-json
From Java I am familiar with json-simple, which allows you to access json-data like this:
{% highlight java %}

//parse to json
JSONObject rootObject = new JSONObject(data);
//get object
String pathOfFirstObject = object.getJSONArray("routes").getJSONObject(0).getString("path");
{% endhighlight %}


The implementation of json-simple in golang allows to do exactly the same:

{% highlight go %}

rootObject, _ := NewJSONObjectFromString(jsonString)
pathOfFirstObject, _ := rootObject.JSONArray("routes").JSONObject(0).String("path")

{% endhighlight %}

### Direct access through segmented keys
Beside chaining calls, you have also the possibility to directly access nested data by providing a segmented key-string.
To access the example data, you would call the key "routes::0::path":

{% highlight go %}

rootObject, _ := NewJSONObjectFromString(jsonString)
pathOfFirstObject, _ := rootObject.String("routes::0::path")
{% endhighlight %}

### Version 0.4 preview: Error handling and chained calls
This feature is planned to be included in Version 0.4.
For sure Error handling is an important topic. In Java we have the possibility to wrap the whole chain
into a Try-Catch block, which prevents the application from crashing, but may hide the source of the error.
For this implementation you can do something similar::

{% highlight go %}

pathOfFirstObject, err := NewJSONObjectFromString(jsonString).
                            JSONArray("routes").
                            JSONObject(0).
                            String("path")
if err != nil {
  fmt.Println("Gotcha! Message: " + err.Error())
}
{% endhighlight %}

The special thing about this is the customized error that is returned when any error happened. The error provides all 
methods that JSONObject and JSONArray provide, therefore the chained method-calls can be executed safely and without 
worrying about panic. All methods of this error return the error itself, so it can be analyzed at the end.
Additionally the error includes the key where the casting failed and can be used to track back the source of the error.
