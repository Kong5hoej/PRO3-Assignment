# PRO3 Course Assignment – Slaughterhouse

A distributed system that tracks animals through a slaughterhouse: from registration, through cutting into parts and trays, to packed products. Each animal and product can be traced in both directions (which animals went into a product, which products came from an animal).

Built with Java 21, gRPC and Spring Boot.

## Architecture

```
Station1Client ──► Station1Server :9091 ──┐
Station2Client ──► Station2Server :9092 ──┼──► DataBaseServer  gRPC :9090
Station3Client ──► Station3Server :9093 ──┘                   REST :8080
```

| Component | Role |
|-----------|------|
| **Station 1** | Registers arriving animals (weight, origin). |
| **Station 2** | Cuts animals into parts, creates trays and puts parts on trays. |
| **Station 3** | Packs parts into products, either a package of one part type or a "half an animal". |
| **DataBaseServer** | Stores everything in memory. Serves gRPC for the stations and a REST API for lookups. |

Each station has a console **client** (operator input) and a **server** that forwards requests to the database server over gRPC.

## Requirements

- JDK 21
- Maven 3.9+

## Build

```sh
mvn compile
```

This also generates the gRPC classes from [src/main/proto/animal.proto](src/main/proto/animal.proto) into `target/`.

## Run

Start the components in this order, each in its own terminal (or run the `main` method from IntelliJ):

1. `slaughterhouse.databaseserver.DataBaseServer`
2. `slaughterhouse.station1.server.Station1Server`
3. `slaughterhouse.station2.sever.Station2Server`
4. `slaughterhouse.station3.server.Station3Server`
5. The clients you need: `slaughterhouse.station1.client.Station1Client`, `slaughterhouse.station2.client.Station2Client`, `slaughterhouse.station3.client.Station3Client`

From the command line:

```sh
mvn spring-boot:run -Dspring-boot.run.main-class=slaughterhouse.databaseserver.DataBaseServer
mvn exec:java -Dexec.mainClass=slaughterhouse.station1.server.Station1Server
mvn exec:java -Dexec.mainClass=slaughterhouse.station1.client.Station1Client
```

The database starts with four sample animals. All data is lost when it stops.

## REST API

Served by `DataBaseServer` on `http://localhost:8080`.

| Method | Path | Description |
|--------|------|-------------|
| GET | `/animals` | All animals. Optional filters: `origin` (case-insensitive) and `date` (`YYYY-MM-DD`). |
| GET | `/animals/{id}` | One animal by id. |

```sh
curl "http://localhost:8080/animals?origin=Danmark"
```

## gRPC services

Defined in [src/main/proto/animal.proto](src/main/proto/animal.proto).

| Service | RPCs |
|---------|------|
| `Station1Service` | `RegisterAnimal` |
| `Station2Service` | `RegisterAnimalPart`, `CreateTray`, `AddAnimalPartToTray` |
| `Station3Service` | `PackParts`, `HalfAnAnimal` |
| `SlaughterhouseService` | `GetAnimalsInProduct`, `GetProductsFromAnimal` |

## Project structure

```
src/main/
├── proto/animal.proto           gRPC service and message definitions
└── java/slaughterhouse/
    ├── databaseserver/          gRPC + REST server, in-memory DAO
    ├── domain/                  Animal, AnimalPart, Tray, Product, Package, HalfAnAnimal
    ├── station1/                client + server
    ├── station2/                client + server
    └── station3/                client + server
```
