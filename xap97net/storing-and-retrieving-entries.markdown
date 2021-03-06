---
layout: post
title:  Storing and Retrieving Entries
categories: XAP97NET
parent: the-space-api.html
weight: 100
---

{% summary %}Storing entry in a space, retrieving entry from a space{% endsummary %}

# Overview

Once you've connected to a space and obtained a `ISpaceProxy`, you can use it to store entries in the space and retrieve them, retrieve entries stored by other clients, and so on. An entry can be any .NET object (it does not have to inherit/implement and base class or interface). This page demonstrates basic space operations using a simple class called `Person`:

{% highlight java %}
class Person
{
  private String _userId;
  private String _name;

  public String UserId
  {
    get { return _userId; }
    set { _userId = value; }
  }

  public String Name
  {
    get { return _name; }
    set { _name = value; }
  }

  public Person()
  {
  }
}
{% endhighlight %}

# Storing an Entry - Write

Entries can be stored in the space using the `Write` method. A write operation stores a copy of the given object in the space (which means that changing the object after the write operation will not affect the entry in the space).

{% indent %}
![space_write.jpg](/attachment_files/xap97net/space_write.jpg)
{% endindent %}

In order to write an entry to the space, create an object instance, populate the relevant properties/fields, then use the `Write` operation to store the object in the space:

{% highlight java %}
Person kermit = new Person();
kermit.UserId = "011-1111111";
kermit.Name = "Kermit the frog";

// Write the object to the space
proxy.Write(kermit);
{% endhighlight %}

{% refer %}See full details in [Writing and Updating Entries](./writing-and-updating-entries.html){% endrefer %}

{% comment %}

{% refer %}See how to [Connecting to a Space]{% endrefer %}

{% endcomment %}

# Retrieving an Entry - Read

Entries can be retrieved from the space using the `Read` method. A read operation queries the space for an entry matching the provided [Query Template Types], and returns a copy of that entry (or null, if no match is found).
The returned object is a copy of the entry stored in the space, which means that changing the returned object does not affect the entry stored in the space.

{% indent %}
![space_read.jpg](/attachment_files/xap97net/space_read.jpg)
{% endindent %}

In order to retrieve an entry from the space, create an instance of the class you wish to retrieve, populate the properties/fields you wish to match by (null properties/fields will be ignored), then use the `Read` operation to retrieve the entry from the space:

{% highlight java %}
// Create a template of a Person with a specific userId
Person readTemplate = new Person();
readTemplate.UserId = "011-1111111";

// Reads a person from the space into readResult
Person readResult = proxy.Read(readTemplate);
{% endhighlight %}

{% refer %}See full details in [Reading Entries](./reading-entries.html){% endrefer %}

# Retrieving and Removing an Entry - Take

Using the `Read` operation does not affect the space - the read entry remains in storage. The `Take` method can be used to retrieve and remove the entry from the space. This operation is very similiar to the `Read`.

{% indent %}
![space_take.jpg](/attachment_files/xap97net/space_take.jpg)
{% endindent %}

In order to take an entry from the space, create a template as you did in the read operation, then use the `Take` operation to retrieve and remove the entry from the space:

{% highlight java %}
// Create a template of a Person with a specific userId
Person takeTemplate = new Person();
takeTemplate.UserId = "011-1111111";

// Takes a person from the space into takeResult
Person takeResult = proxy.Take(takeTemplate);
{% endhighlight %}

# Cluster routing

All operations are done through a proxy to a space, that proxy can also be a proxy to a cluster of spaces where each space holds a certain partition of the data of all the cluster, this is known as the partitioned cluster topology. Each new object that is stored in the cluster is routed to a specific partition (space), this partition is determined by the hashcode of the SpaceRouting field or property. Routing an object to a specific partition allows fast retrieval afterwards. Instead of searching for the object in each one of the spaces in the cluster, the proxy can query a specific space with in the cluster using the SpaceRouting field or property.

The SpaceRouting is determined by the [`[SpaceRouting]`](./object-metadata.html#SpaceRouting) attribute or using a [gs.xml](./gs.xml-metadata.html).

{% refer %}SpaceRouting consideration in [Write and Update operation](./writing-and-updating-entries.html){% endrefer %}

{% refer %}SpaceRouting consideration in [Read operation](./reading-entries.html){% endrefer %}
