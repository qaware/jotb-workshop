# Protobuf Serialization with Quarkus and JAX-RS

## Protocol Buffers in a Nutshell

Protocol Buffers are a language-neutral, platform-neutral extensible mechanism for serializing structured data.
- https://developers.google.com/protocol-buffers
- Like XML or JSON - just smaller, faster and easier!
- Google Protobuf uses an efficient binary format to serialize data structures.
- An Interface Definition Language (IDL) is used to define data structures and message payloads. Many primitive types, enums, maps, arrays, nested types.
- Protocol Buffers supports code generation for Java, Python, Objective-C, C++, Kotlin, Dart, Go, Ruby und C#
- Protobuf supports evolution as well as extension of schemas. Backwards and forwards compatibility are supported.

## Quarkus REST Quickstart

If you want to start yourself, go to the Quarkus webpage at https://code.quarkus.io and select the following packages:
- RESTeasy Classic
- RESTeasy Classic JSON-B
- SmallRye Health 

Otherwise, open the `quarkus-beer-rest/` project in your IDE. The `quarkus-beer-rest/README.md` contains instructions
on how to build and run the service locally.

```bash
# local development
./gradlew quarkusDev

# build and deploy microservice
skaffold dev --no-prune=false --cache-artifacts=false

# alternatively, use Tilt
tilt up
```

## Protocol Buffers Definition and Build Integration

First, we need to extend the Gradle build with the Protobuf Gradle plugin, its configuration and dependency.
Open the `build.gradle` file and extend it with the following content:
```groovy
plugins {
    // add this to the plugins section
    id "com.google.protobuf" version "0.8.19"
}

dependencies {
    // add this to the dependencies
    implementation 'com.google.protobuf:protobuf-java:3.21.5'
}

protobuf {
    // Configure the protoc executable
    protoc {
        // Download from repositories
        artifact = 'com.google.protobuf:protoc:3.21.5'
    }
}

sourceSets.main.java.srcDirs += ['build/generated/source/proto/main/java']
```

Next, create a file called `beer.proto` in the directory `src/main/proto` with the following content:
```proto
syntax = "proto3";

package beer;

option java_package = "hands.on.grpc";
option java_outer_classname = "BeerProtos";
option optimize_for = SPEED;

message Beer {
  string asin = 1;
  string name = 2;
  string brand = 3;
  string country = 4;
  float alcohol = 5;
  enum BeerType {
    IndianPaleAle = 0;
    SessionIpa = 1;
    Lager = 2;
  }
  BeerType type = 6;
}

message GetBeersRequest {
}

message GetBeersResponse {
  repeated Beer beers = 1;
}

message GetBeerRequest {
  string asin = 1;
}

message GetBeerResponse {
  Beer beer = 1;
}

message CreateBeerRequest {
  Beer beer = 1;
}

message UpdateBeerRequest {
  Beer beer = 1;
}

message DeleteBeerRequest {
  string asin = 1;
}
```

Execute `./gradlew build` after this step and check if the proto files have been generated.

# Using Protocol Buffers with JAX-RS

Protocol Buffers can be used in JAX-RS using a `MessageBodyReader` and `MessageBodyWriter` implementation.
In order to use the generated Protobuf message payloads we need to implement a JAX-RS endpoint and methods.

- Copy the `protobuf/` directory and the contained Java files into the `src/main/java/hands/on/grcp/` directory.
- Copy the `ProtoResource.java` file into the `src/main/java/hands/on/grcp/` directory.
- Copy the `ProtoResourceTest.java` file into the `src/test/java/hands/on/grcp/` directory.

Execute `./gradlew build` after this step and check if it succeeds.

# How to test that it works

You can execute the `ProtoResourceTest.java` that we just copied to the `src/test/java/hands/on/grcp/` directory.
You may also try out `protocurl`: https://github.com/qaware/protocurl, which is cURL for Protobuf.

You can run the following command in the `quarkus-beer-rest` directory:
```
docker run -v "$PWD/src/main/proto:/proto" --network host qaware/protocurl \
-f beer.proto -i beer.GetBeersRequest -o beer.GetBeersResponse \
-u http://localhost:18080/api/proto/getBeers -d ""
```