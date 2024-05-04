---
layout: post
title: "Use embedding dimension with the new OpenAI models in Java"
---


OpenAI provides its own Python library to interact with their APIs but when it comes to Java, there is an OpenSource project [TheoKanning/openai-java](https://github.com/TheoKanning/openai-java/pull/470) which provides an SDK over these APIs.

But the problem I faced was that a couple of months back, OpenAI launched their [third generation of embedding models](https://openai.com/index/new-embedding-models-and-api-updates). These models are more efficient even in lesser dimensions. So it made sense to switch to them and save vector storage cost. But the aforementioned Java library did not provide an option to interact with these new models. You could but there was no option to pass the embedding dimension in the request so you could not use the new models to their fullest. 

I tried opening a merge request but as the project was not under active development and I faced a lot of trouble with some stale tests, I decided to just override the methods. Here is how I did it.

They are using an interface called `OpenAiApi` which provides you with all the various endpoints that are being called. Here `retrofit` client is being used to map the endpoints to their services. 

So, first I created a new interface called `OpenAiApiExtended`, extending from `OpenAiApi` interface.


```java
import com.theokanning.openai.client.OpenAiApi;
import com.theokanning.openai.embedding.EmbeddingResult;
import io.reactivex.Single;
import retrofit2.http.Body;
import retrofit2.http.POST;

/** Interface for using embedding model with dimension size. */
public interface OpenAiApiExtended extends OpenAiApi {

  @POST("/v1/embeddings")
  Single<EmbeddingResult> createEmbeddingsV2(@Body EmbeddingV2Request var1);
}
```

Here, I have given the embedding API endpoint I want to call `/v1/embeddings/` and to support the new dimension parameter, I am passing `EmbeddingV2Request` object, which looks like this

```java
import java.util.List;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.NonNull;

@Builder
@NoArgsConstructor
@AllArgsConstructor
@Data
public class EmbeddingV2Request {

  String model;

  @NonNull List<String> input;

  String user;

  Integer dimensions;
}
```

The main API logic in the library is written as part of `OpenAiService`. So in order to make a similar functionality here, I extending this class in my new `OpenAiServiceV2` class and provided it with the basic things in the constructor that were needed - `ObjectMapper` and `OkHttpClient` objects which are then used to create `Retrofit` client. Then simply I created a new function `createEmbeddingsV2` which called the `OpenAiApiExtended` interface's `createEmbeddingsV2` method.

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.theokanning.openai.embedding.EmbeddingResult;
import com.theokanning.openai.service.OpenAiService;
import java.time.Duration;
import okhttp3.OkHttpClient;
import retrofit2.Retrofit;

/** Wrapper class of OpenAiService that enables request with an updated request body. */
public class OpenAiServiceV2 extends OpenAiService {

  private static final Duration DEFAULT_TIMEOUT = Duration.ofSeconds(10);

  private final OpenAiApiExtended apiV2;

  /**
   * Creates a new OpenAiService that wraps OpenAiApi.
   *
   * @param token OpenAi token
   */
  public OpenAiServiceV2(final String token) {
    super(token);

    ObjectMapper mapper = defaultObjectMapper();
    OkHttpClient client = defaultClient(token, DEFAULT_TIMEOUT);
    Retrofit retrofit = defaultRetrofit(client, mapper);

    this.apiV2 = retrofit.create(OpenAiApiExt.class);
  }

  public EmbeddingResult createEmbeddingsV2(EmbeddingV2Request request) {
    return execute(apiV2.createEmbeddingsV2(request));
  }
}
```

Worked like a charm!