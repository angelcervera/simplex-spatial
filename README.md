<!--
<div align="right">
<a href="http://www.reactivemanifesto.org/"> <img style="border: 0; position: fixed; right: 0; top:0; z-index: 9000" src="https://d379ifj7s9wntv.cloudfront.net/reactivemanifesto/images/ribbons/we-are-reactive-orange-right.png"> </a>
</div>
--->

<p align="center">
  <img src="doc/assets/logo.svg" alt="Simplexspatial logo">
</p>

# SimplexSpatial
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fsimplexspatial%2Fsimplexspatial.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Fsimplexspatial%2Fsimplexspatial?ref=badge_shield)
[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-v2.0%20adopted-ff69b4.svg)](code_of_conduct.md)
![Scala CI](https://github.com/simplexspatial/simplexspatial/workflows/Scala%20CI/badge.svg)

SimplexSpatial is a GeoSpatial server, focuses on **distributed data storage and algorithms execution**. It is a distributed, horizontally scalable and fault-tolerant system based in AKKA, following the four pillars of [The Reactive Manifesto](https://www.reactivemanifesto.org/): Responsive, Resilient, Elastic and Message Driven.

# Project status
The project is still in a really early state, so pretty sure a lot of changes will be done in the near future. 

At the moment, this is the list of features implemented:
- Two different APIs has been implemented as entry points:
  - [x] [JSON](#restful-examples): Easy to use restful API, ideally for testing or web.
  - [x] [gRPC](https://grpc.io/): Ideally to intensive requests, like load data. As good example, refer to [load-osm file project](https://github.com/simplexspatial/simplexspatial-loader-osm).
  - [x] gRPC-Web
  - [x] [gRPC Reflection](https://github.com/grpc/grpc/blob/master/doc/server-reflection.md)
- [x] Add/Get Nodes or Vertices.
- [x] Add/Get Ways or Edges.
- [x] Batch mode / Streaming mode commands execution.
- [x] Search the nearest node search.
- [ ] Search the nearest node search.
- [ ] Calculate Area of influence or Isolines
- [ ] Search the Shortest path between two nodes.

 
# Technologies
The current implementation uses AKKA as Actor Model toolkit and Scala as programming language.
- Akka-http + spray json to implement the RestFul entry point.
- Akka gRPC to implement the gRPC entry point.
- Akka Cluster / Sharding to distribute data and computation.
- Akka persistence to implement a CQRS model.

# Launch (Docker Compose Demo)
Because the early state of the project, there are not deployables packages to download, but there is a Docker Repository
with the latest SNAPSHOT at [Simplexspatial Docker Repository](https://hub.docker.com/repository/docker/simplexspatial/simplexspatial)

You can use the docker-compose.yml in the folder [doc/demo/docker-compose](doc/demo/docker-compose) that will start an environment with:
- Cassandra
- Simplexspatial

If you have Docker and Docker Compose installed in your system, the simplest way to run it is [checking out](Checkout the project) the project
and executing `docker-compose up` from the folder [doc/demo/docker-compose](doc/demo/docker-compose).

There is a more detailed documentation at [the Demo with Docker Compose section](doc/demo/docker-compose/README.md).

# Builing

## Tooling to build the artifact
- Java 8: The server has been tested with Java 8, but it should work with newer versions as well. 
- Sbt: It is the building tools used, so you need to install it in your system. The last version is always fine.

## Checkout the project
Clone the last version from the master and move into the new folder. Master will always contain the last stable version.
```ssh
git clone https://github.com/simplexspatial/simplexspatial.git
cd simplexspatial
```

## Packaging
The project uses `sbt` and `sbt-native-packager` to generate the artifact:
```ssh
sbt clean universal:packageZipTarball
```
The previous step generated a file called `core/target/universal/simplexspatial-core-0.0.1-SNAPSHOT.tgz`. This file
contains all the necessary stuff to start a new node.

## Deploying
Uncompress the previous generated file into a folder. From that folder, you will be able to start new instances of the
server.
In this example, we will use the home `~` folder. This is only as example.

```ssh
tar -xvf core/target/universal/simplexspatial-core-0.0.1-SNAPSHOT.tgz -C ~
```

Next, let's move into the new folder. From this point, we will use the new folder as root for all commands.

```ssh
cd ~/simplexspatial-core-0.0.1-SNAPSHOT
```

The new folder contains the follow structure:
```text
.
├── bin
│   ├── simplexspatial-core
│   └── simplexspatial-core.bat
├── conf
│   ├── application-akka-commons.conf
│   ├── application-cassandra.conf
│   ├── application.conf
│   ├── application.ini
│   ├── application-postgres.conf
│   ├── application-simplexspatial.conf
│   └── logback.xml
├── jetty-alpn-agent
│   └── jetty-alpn-agent-2.0.9.jar
└── lib
    └──  A lot of jar files ;)
```

## Configuring
Simplexspatial is using [lightbend config](https://github.com/lightbend/config) as configuration framework, like Akka uses.
The configuration file is `config/application.conf`.

SimplexSpatial is using the same configuration system that is used in
AKKA: [lightbend config](https://github.com/lightbend/config). It means
than you can set and overwrite configuration properties as it is
explained in the Lightbend Config site.

### Simplexspatial configuration parameters
In the [previously generated package](#packaging), `conf/application.conf` contains the specific configuration for the
server. This is the config file used by default from the script used to
start a node.

As reference, this is the set of parameters that the server uses:
```
simplexportal.spatial {
  entrypoint {
    restful {
      interface = "0.0.0.0"
      port = 8080
    }
    grpc {
      interface = "0.0.0.0"
      port = 7080
    }
    grpc-web {
      interface = "0.0.0.0"
      port = 6080
    }
  }
  indexes {
    grid-index {
      partitions {
        nodes-lookup = 50
        ways-lookup = 50
        latitude = 10000
        longitude = 10000
      }
    }
  }
}
```

Remind that the server is base in [AKKA](https://akka.io/), so you can
set any parameters related to AKKA as well.

### Akka cluster configuration
In relation to the AKKA cluster configuration, it is important to configure the way that the cluster is going to work.

In this stage of the project, and only for testing, the default configuration is using Postgres to store the journal and
snapshots.
So this is the default configuration:
```
akka {

  log-dead-letters = 100
  log-dead-letters-during-shutdown = on
  loglevel = "ERROR"

  extensions = [akka.persistence.Persistence]

  persistence {
    journal {
      plugin = "jdbc-journal"
      auto-start-journals = ["jdbc-journal"]
    }
    snapshot-store {
      plugin = "jdbc-snapshot-store"
      auto-start-snapshot-stores = ["jdbc-snapshot-store"]
    }
  }

  cluster {
    seed-nodes = [
      "akka://SimplexSpatialSystem@127.0.1.1:2550",
      "akka://SimplexSpatialSystem@127.0.1.1:2551"
    ]
    sharding {
      number-of-shards = 100
    }
  }

}

akka-persistence-jdbc {
  shared-databases {
    slick {
      profile = "slick.jdbc.PostgresProfile$"
      db {
        host = "localhost"
        url = "jdbc:postgresql://localhost:5432/akka-persistence?reWriteBatchedInserts=true"
        user = "akka"
        password = "pass"
        driver = "org.postgresql.Driver"
        numThreads = 5
        maxConnections = 5
        minConnections = 1
      }
    }
  }
}

jdbc-journal {
  use-shared-db = "slick"
}

jdbc-snapshot-store {
  use-shared-db = "slick"
}
```

This means that:
- It is using in JDBC persistence journal and snapshots.
- It is using fixed seed nodes. Remind to update the IP (in this case it is the local IP for Ubuntu 19.10) and ports.

### Java configuration
All configuration related wto the JVM (memory, JMX config, etc.) is available in the file `conf/application.ini`.

### Logging configuration
AKKA is using [SLF4J](http://www.slf4j.org/) but SimplexSpatial adds
[logback](http://logback.qos.ch/) to the classpath, so that will be the
library to configure.

Important information about logging configuration:
- [AKKA SLF4J backend](https://doc.akka.io/docs/akka/current/typed/logging.html#slf4j-backend)
- [Internal logging by Akka](https://doc.akka.io/docs/akka/current/typed/logging.html#internal-logging-by-akka)
- [LOGBack configuration](http://logback.qos.ch/manual/configuration.html)

## Installing Dependencies
Akka persistence needs a storage system to store the journal and snapshots.

- In production or for performance test, it is recommended to use a distributed storage, like Cassandra.
- For testing ( default configuration ) Postgresql is enough. In the `/dev` folder, there is a `docker-compose.yml` file to
be able to start all dependencies for testing. In the same folder, you can find the SQL file with the database schema.

To use with Postgres, from the `doc/dev` folder:
- To start
    ```ssh
    docker-compose -f docker-compose-postgres.yml up -d
    ```

- To create the database or clean up:
    ```ssh
    docker cp ./schema-postgres.sql dev_db_1:/root/schema-postgres.sql
    docker exec dev_db_1 psql akka-persistence akka -f /root/schema-postgres.sql
    ```

- To stop:
    ```ssh
    docker-compose -f docker-compose-postgres.yml down -d
    ```

More information about [Docker Compose in the documentation](https://docs.docker.com/compose/).

## Running
The last step, start a node. It is important the order starting nodes. In the configuration for testing, seed nodes are
hardcoded, so you need to start the two seed nodes before the rest.

From the folder where you decompressed the server, in out case `~/simplexspatial-core-0.0.1-SNAPSHOT`:

Node 1:
```ssh
bin/simplexspatial-core \
    -J-Xms5G \
    -J-Xmx14G  \
    -Dconfig.file=conf/application.conf \
    -Dakka.remote.artery.canonical.port=2550  \
    -Dsimplexportal.spatial.entrypoint.grpc-web.port=6080 \
    -Dsimplexportal.spatial.entrypoint.grpc.port=7080 \
    -Dsimplexportal.spatial.entrypoint.restful.port=8080 \
    -Dcinnamon.prometheus.http-server.port=9001

bin/simplexspatial-core \
    -java-home /usr/lib/jvm/java-8-openjdk-amd64 \
    -jvm-debug 9010 \
    -J-Xms1G \
    -J-Xmx4G  \
    -Dconfig.file=conf/application.conf\
    -Dakka.remote.artery.canonical.port=2550  \
    -Dsimplexportal.spatial.entrypoint.grpc-web.port=6080 \
    -Dsimplexportal.spatial.entrypoint.grpc.port=7080 \
    -Dsimplexportal.spatial.entrypoint.restful.port=8080 \
    -Dcinnamon.prometheus.http-server.port=9001
```

Node 2:
```ssh
bin/simplexspatial-core \
    -J-Xms5G \
    -J-Xmx14G  \
    -Dconfig.file=conf/application.conf\
    -Dakka.remote.artery.canonical.port=2551  \
    -Dsimplexportal.spatial.entrypoint.grpc-web.port=6081 \
    -Dsimplexportal.spatial.entrypoint.grpc.port=7081 \
    -Dsimplexportal.spatial.entrypoint.restful.port=8081 \
    -Dcinnamon.prometheus.http-server.port=9002
```
bin/simplexspatial-core \
    -J-Xms5G \
    -J-Xmx14G  \
    -Dconfig.file=conf/application.conf\
    -Dakka.remote.artery.canonical.port=2551  \
    -Dsimplexportal.spatial.entrypoint.grpc-web.port=6081 \
    -Dsimplexportal.spatial.entrypoint.grpc.port=7081 \
    -Dsimplexportal.spatial.entrypoint.restful.port=8081 \
    -Dcinnamon.prometheus.http-server.port=9002
For other nodes, we can set the port as 0, so Akka will use a random free port:
```ssh
bin/simplexspatial-core \
    -J-Xms1G \
    -J-Xmx4G  \
    -Dconfig.file=conf/application.conf\
    -Dakka.remote.artery.canonical.port=0  \
    -Dsimplexportal.spatial.entrypoint.grpc-web.port=0 \
    -Dsimplexportal.spatial.entrypoint.grpc.port=0 \
    -Dsimplexportal.spatial.entrypoint.restful.port=0 \
    -Dcinnamon.prometheus.http-server.port=0
```

## Examples.
### gRPC
gRPC is the way to go (if it is possible) in production.

As part of the [packaging step](#packaging), a gRPC Akka Stream client library has been created to used. If you want to
create your own library, the gRPC sevice definition is available in the report, under `protobuf-api/src/main/resources/simplexspatial.proto`.

As example, you can checkout the code in the [osm pbf loader project](https://github.com/simplexspatial/simplexspatial-loader-osm),
used to load osm files into the Simplexspatial server.

### Restful examples
Using [Httpi](https://httpie.org/)

#### Batch mode
To seed the database, it is possible to use a json file with the list of commands to execute. Note that the performance and
size of the seed file is not adequate for big data sets. So this is recommended only for testing purpose.

To execute a sequence of commands in one request.

`batch_commands.json` content:
```json
{
  "nodes": [
    {
      "id": 100001,
      "lon": 1.100001,
      "lat": -1.000002,
      "attributes": {}
    },
    {
      "id": 100002,
      "lon": 2.100001,
      "lat": -2.000002,
      "attributes": {}
    }
  ],
  "ways": [
    {
      "id": 100010,
      "nodes": [
        100001,
        100002
      ],
      "attributes": {
        "name": "Two points way."
      }
    }
  ]
}
```

Request to execute all commands in batch mode:
```ssh
http PUT http://localhost:8080/batch < batch_commands.json
```

#### Nodes

- Add a node

    ```ssh
    http PUT http://localhost:8080/node/1 lon:=3.0001 lat:=-2.4444 attributes:={}
    ```
    
    or
    
    `node_file.json` content:
    ```json
    {
      "lon": 10.100001, "lat": -22.000002, "attributes": {}
    }
    ```
    Request to insert node:
    ```ssh
    http PUT http://localhost:8080/node/1 < node_file.json
    ```

- Get a Node
    ```ssh
    http GET http://localhost:8080/node/1
    ```

#### Ways

- Add a Way

    ```ssh
    http PUT http://localhost:8080/way/10 nodes:=[1,2] attributes:={}
    ```
    
    or
    
    `way_file.json` content:
    ```json
    {
      "nodes": [1, 2],
      "attributes": { }
    }
    ```
    Request to insert way:
    ```ssh
    http PUT http://localhost:8080/node/10 < way_file.json
    ```

- Get a Way
    ```ssh
    http GET http://localhost:8080/way/10
    ```

#### Nearest node
```ssh
http -v GET http://localhost:8080/algorithm/nearest/node lat==43.73819 lon==7.4269
```

## Other documentation
- [Architecture and Design documentation](doc/architecture.md)
- [Performance documentation](doc/performance.md)
- [Demo](doc/demo/docker-compose)

## Notes
- Enable GRPC logs: -Djava.util.logging.config.file=/path/to/grpc-debug-logging.properties
- More information about Artery in the [AKKA documentation](https://doc.akka.io/docs/akka/current/remoting-artery.html).

## License and attributions.
- Using icons from [Online Web Fonts](http://www.onlinewebfonts.com) licensed by CC BY 3.0.
- Using icons from [Font Awesome](https://fontawesome.com/) licensed by CC BY 4.0.

[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fsimplexspatial%2Fsimplexspatial.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Fsimplexspatial%2Fsimplexspatial?ref=badge_large)
