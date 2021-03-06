---
layout: post
title:  Getting Partial Results Using The Projection API
categories: XAP97NET
parent: querying-the-space.html
weight: 100
---

{% summary %}This page describes how you can obtain partial results when querying the space to improve application performance and reduce memory footprint.{% endsummary %}

# Overview

{% section %}

{% column width=40 %}

In some cases when querying the space for objects, only specific properties of objects are required and not the entire object (delta read). The same scenario is also relevant when subscribing for notifications on space data changes, where only specific properties are of interest to the subscriber. For that purpose the Projection API can be used where one can specify which properties are of interest and the space will only populate these properties with the actual data when the result is returned back to the user. This approach reduces network overhead , garbage memory generation and serialization CPU overhead.

{% endcolumn %}

{% column %}

![space-projections.jpg](/attachment_files/xap97net/space-projections.jpg)

{% endcolumn %}

{% endsection %}

# Specifying a Projection with your Query

Projections are supported using a [SqlQuery](./sqlquery.html) or [ID Queries](./id-queries.html). Below is a simple example that demonstrates reading a `Person` object where only the 'FirstName' and 'LastName' properties are returned with the query result array. All other `Person` properties will not be returned:

{% highlight java %}
public class Person
{
  ...
  [SpaceId(AutoGenerate = false)]
  public Long Id {get; set;}
  public String FirstName {get; set;}
  public String LastName {get; set;}
  public String Address {get; set;}
  ...
}

ISpaceProxy space = //... obtain a space reference.
Long id = //... obtain the space object ID.
Person result = space.Read<Person>(new IdQuery<Person>(id) {Projections = new []{"FirstName", "LastName"});
{% endhighlight %}

With the above example a specific Person is being read but only its 'FirstName' and 'LastName' will contains values and all the other properties will contain a `null` value.

You may use the same approach with the `SqlQuery` or `IdsQuery`:

{% highlight java %}
SqlQuery<Person> query = new SqlQuery<Person>("") {Projections = new []{"FirstName", "LastName"});
Person result[] = space.ReadMultiple(query);

IdsQuery<Person> idsQuery = new IdsQuery<Person>(new Long[]{id1,id2}) {Projections = new []{"FirstName", "LastName"};
Person result[] = space.ReadByIds(idsQuery).ResultsArray;
{% endhighlight %}

The [SpaceDocument](./document-(schema-free)-entries.html) support projections as well:

{% highlight java %}
SqlQuery<SpaceDocument> docQuery = new SqlQuery<SpaceDocument>(typeof(Person).Name ,"") {Projections = new []{"FirstName", "LastName"};
SpaceDocument docresult[] = space.ReadMultiple(docQuery);
{% endhighlight %}

# Supported Operations

The projection is defined for any operation that returns data from the space. Therefore ID Based or Query based operations support projections. You can use projections with `Read`,`Take`,`ReadById`,`TakeById`,`ReadMultiple` and `TakeMultiple` operations. When performing a `Take` operation with projections, the entire object will be removed from the space, but the result returned to the user will contain only the projected properties.
You can use projections with the [Notify Container](./notify-container-component.html) when subscribing to notifications, or with the [Polling Container](./polling-container-component.html) when consuming space objects. You can also create a [Local View](./local-view.html) with templates or a `View` using projections. The local view will maintain the relevant objects, but with the projected data - only with the projected properties.
Projected properties can specify both dynamic or fixed properties and the usage is the same. As a result, when providing a projected property name which is not part of the fixed properties set, it will be treated as a dynamic property. If there is no dynamic property present with that name on an object which is a result of the query - that projection property will be ignored (and no exception will be raised). Please note that a result may contain multiple objects, each with different set of properties (fixed and dynamic), each object will be treated individually when applying the projections on it.

{% exclamation %} Space Iterator does not support projections

# Considerations

1. Projections are supported only for first level properties (root level). Nested properties can't be specified as part of the projection properties list.
2. You can't use a projection on [Local Cache](./local-cache.html) as the local cache needs to contain the fully constructed objects, and reconstructing it locally with projections will only impact performance.
3. You can't use a projection to query a Local View for the same reason as Local Cache, however, you can create the local view with projection template and the Local View will be contain the objects in their projected form.
