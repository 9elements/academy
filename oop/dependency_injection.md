# Dependency Injection

Imgine the following code:

```java
public class PoorManTweeting {
  public void tweet(String tweet) {
    TwitterApi api = new TwitterApi();
    api.postTweet("JakeWharton", tweet)
  }
}

public class TwitterApi {
  public void postTweet(String user, String tweet) {
    OkHttpClient client = new OkHttpClient();
    Request request = new Request(...);
    client.newCall(request).execute();
  }
}

```

This code is poor in many ways. First the `PoorManTweeting` directly relies on the `TwitterApi` and there is now way to change that. Second `TwitterApi`creates a http client each request. We can beautify that code by moving the client to a static class member:

```java
public class TwitterApi {
  private final OkHttpClient client = new OkHttpClient();

  public void postTweet(String user, String tweet) {
    Request request = new Request(...);
    client.newCall(request).execute();
  }
}
```

But a better way would be to inject the http client as a dependency:

```java
public class TwitterApi {
  private final OkHttpClient client;

  public TwitterApi(OkHttpClient client) {
    this.client = client;
  }

  public void postTweet(String user, String tweet) {
    Request request = new Request(...);
    client.newCall(request).execute();
  }
}
```

Let's look at the whole code with all dependencies:

```java
public class PoorManTweeting {
  public PoorManTweeting(TwitterApi api, String user) {
    this.api = api;
    this.user = user;
  }

  public void tweet(String tweet) {
    api.postTweet(user, tweet)
  }
}

public class TwitterApi {
  private final OkHttpClient client;

  public TwitterApi(OkHttpClient client) {
    this.client = client;
  }

  public void postTweet(String user, String tweet) {
    Request request = new Request(...);
    client.newCall(request).execute();
  }
}
```

If you pair the concrete TwitterApi with an interface (using interface separation) you could swap out the TwitterApi with a MockApi in a testing environment.

# DI on Android

The defacto standard for DI on Android is [Dagger 2](http://google.github.io/dagger/).

# Other resources

This post heavily steals from [Jake Whartons talk at devoxx](https://www.parleys.com/tutorial/5471cdd1e4b065ebcfa1d557/).
