# livestub

[![Build Status](https://travis-ci.org/softwaremill/livestub.svg?branch=master)](https://travis-ci.org/softwaremill/livestub)
[![Docker Pulls](https://img.shields.io/docker/pulls/softwaremill/sttp.livestub.svg)](https://hub.docker.com/r/softwaremill/sttp.livestub/)
[![ImageLayers](https://images.microbadger.com/badges/image/softwaremill/sttp.livestub.svg)](https://microbadger.com/#/images/softwaremill/sttp.livestub)
[![Scala Steward badge](https://img.shields.io/badge/Scala_Steward-helping-brightgreen.svg?style=flat&logo=data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAA4AAAAQCAMAAAARSr4IAAAAVFBMVEUAAACHjojlOy5NWlrKzcYRKjGFjIbp293YycuLa3pYY2LSqql4f3pCUFTgSjNodYRmcXUsPD/NTTbjRS+2jomhgnzNc223cGvZS0HaSD0XLjbaSjElhIr+AAAAAXRSTlMAQObYZgAAAHlJREFUCNdNyosOwyAIhWHAQS1Vt7a77/3fcxxdmv0xwmckutAR1nkm4ggbyEcg/wWmlGLDAA3oL50xi6fk5ffZ3E2E3QfZDCcCN2YtbEWZt+Drc6u6rlqv7Uk0LdKqqr5rk2UCRXOk0vmQKGfc94nOJyQjouF9H/wCc9gECEYfONoAAAAASUVORK5CYII=)](https://scala-steward.org)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.softwaremill.sttp.livestub/livestub-app_2.13/badge.svg)](https://search.maven.org/search?q=g:com.softwaremill.sttp.livestub)


With livestub you can easly setup http server that behaves exactly as you would like it to. That makes livestub a perfect solution for faking arbitrary services which might be very useful in development/testing.

## Usage

### launch
 - **coursier**

    `coursier launch com.softwaremill.sttp.livestub:livestub-app_2.13:0.1.9 -- -p 7070`

- **docker**

    `docker run -p 7070:7070 softwaremill/sttp.livestub`

### stub
```
curl -X POST 'localhost:7070/__set' \
-d '{"when":{"method":"GET", "url":"animals/1/status"}, "then": {"statusCode":200, "body":{"status": "happy"} }}'
```

### invoke
```
curl 'localhost:7070/animals/1/status'
{"status":"happy"}
```

### swagger

Swagger is available under `/__admin/docs`

### advanced stubbing

Livestub supports wildcard http methods as well as wildcard path parameters.

wildcard method:
```
curl -X POST 'localhost:7070/__set' \
-d '{"when":{"method":"*", "url":"dogs"}, "then": {"statusCode":200, "body":{"status": "OK"} }}'
```

wildcard path param: 
```
curl -X POST 'localhost:7070/__set' \
-d '{"when":{"method":"GET", "url":"dogs/*/status"}, "then": {"statusCode":200, "body":{"status": "happy"} }}'
```

multiwildcard path param: (this one catches all routes which originate from `/dogs`)
```
curl -X POST 'localhost:7070/__set' \
-d '{"when":{"method":"GET", "url":"dogs/**"}, "then": {"statusCode":200, "body":{"status": "cheerful"} }}'
```

### additional methods

clear stubbed routes
```
curl -X POST 'localhost:7070/__clear'
```

show stubbed routes

```
curl 'localhost:7070/__routes'
```

set many responses which will be cycled through
```
curl -X POST 'localhost:7070/__set_many' \
-d '{"when":{"method":"GET", "url":"animals/1/status"}, "then": [
    {"statusCode":200, "body":{"status": "happy"} },
    {"statusCode":200, "body":{"status": "unhappy"} }
  ]
}'
```

### stubbing from code - sdk

```scala
libraryDependencies += "com.softwaremill.sttp.livestub" % "livestub-sdk" % "0.1.9"
```

stubbing an arbitrary request
```scala
AsyncHttpClientCatsBackend[IO]().flatMap { implicit backend: SttpBackend[IO, Nothing, WebSocketHandler] =>
      val livestub = new LiveStubSdk[IO, Nothing, WebSocketHandler](uri"http://mock:7070")
      livestub
        .when(RequestStub(MethodValue.Wildcard, "/user/*/status"))
        .thenRespond(Response.emptyBody(StatusCode.Ok, List(ResponseHeader("X-App", "123"))))
    }
```

stubbing given [sttp](https://github.com/softwaremill/sttp) request:
```scala
    AsyncHttpClientCatsBackend[IO]().flatMap { implicit backend: SttpBackend[IO, Nothing, WebSocketHandler] =>
      val request = basicRequest
        .body(Map("name" -> "John", "surname" -> "doe"))
        .post(uri"https://httpbin.org/post?signup=123")

      val livestub = new LiveStubSdk[IO, Nothing, WebSocketHandler](uri"http://mock:7070")
      livestub.when(request).thenRespond(Response(Some(Json.fromString("OK")), StatusCode.Ok))
    }
```
stubbing given [tapir](https://github.com/softwaremill/tapir) endpoint:
```scala
    AsyncHttpClientCatsBackend[IO]().flatMap { implicit backend: SttpBackend[IO, Nothing, WebSocketHandler] =>
      val myEndpoint = endpoint.get.in("/status").out(stringBody)

      val livestub = new LiveStubSdk[IO, Nothing, WebSocketHandler](uri"http://mock:7070")
      livestub.when(myEndpoint).thenRespond(Response.emptyBody(StatusCode.Ok, List(ResponseHeader("X-App", "123"))))
    }
```
