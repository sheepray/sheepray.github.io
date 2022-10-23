---
layout: post
title:  "Add common WireMock setup in scalatest"
date:   2022-10-23 16:54:02 +0200
categories: wiremock scalatest scala
---
# Problem
I had a task to create a HTTP client to call another service. When creating tests for the HTTP client, I have used WireMock to mock the APIs that the client consumes. But the problem is that I need to setup WireMock in each tests.

# Solution
To avoid code duplication, I have implemented the following `trait`

{% highlight scala %}
trait WireMockAware extends BeforeAndAfterAll {
 this: Suite =>
 val host = "localhost"
 val wireMockServer = new WireMockServer(wireMockConfig().port(9090))

 override protected def beforeAll(): Unit = {
   super.beforeAll()
   wireMockServer.start()
   WireMock.configureFor(host, wireMockServer.port)
 }

 override protected def afterAll(): Unit = {
   super.afterAll()
   wireMockServer.stop()
 }

 def getWireMockEndpoint(): String = {
   s"http://$host:${wireMockServer.port}"
 }
}
{% endhighlight %}

and I can use it in my tests which requires WireMock, for instance:
{% highlight scala %}
class MyHTTPClientSpec extends AnyFlatSpec with WireMockAware {
  "getUser" should "returns an user with the given id" in {
    ...
  }
}
{% endhighlight %}

If I need to add additional setup in `beforeAll` or extra clean up in `afterAll`, I can just add
{% highlight scala %}
class MyHTTPClientSpec extends AnyFlatSpec with WireMockAware {
  override protected def beforeAll(): Unit = {
    super.beforeAll()
    // additional setup
  }

  override protected def afterAll(): Unit = {
    super.afterAll()
    // additional clean up
  }

  "getUser" should "returns an user with the given id" in {
    ...
  }
}
{% endhighlight %}
