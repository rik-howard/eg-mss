


# Rough
Here be dragons ...


## Overview

MSS comprises two microservices, a message queue and a public load balancer:

* the music microservice (UMS),
* the money microservice (OMS),
* the MSS message queue (MMQ) and
* the MSS load balancer (MLB).

The microservices, respectively, comprise some applications and databases:

* the music command application (UCA),
* the music query application (UQA) and
* the music database (UDB); and
* the money persistence application (OPA),
* the money query application (OQA) and
* the money database (ODB).

UQA publishes and OPA subscribes to an MMQ channel:

* the music query channel (UQQ)

### Contracts
These are written ito the solution domain [DDD] language.

Platform

* CloudPP -- platform project: GCP
* CloudLB -- load balancer: external HTTP(S) load balancer (Global, Regional, Classic?)
* CloudAE -- application engine: AppEngine
* CloudDB -- database: CloudMySQL
* CloudMQ -- message queue: Pub/Sub

Application

* Java for CloudAE
* Java for CloudDB
* Java for CloudMQ


## The Production Platform
The following components are provided by a cloud platform.

1.  A CloudLB supports some public HTTP endpoints, each endpoint supporting a subset of HTTP methods.  Each request to an endpoint-method is routed to a protected CloudAE endpoint.  Two public endpoints are shown, conceptualising two microservices.

            https://eg.net/mss/music POST PUT DELETE GET
            https://eg.net/mss/money POST GET

1.  The CloudAE supports some protected HTTP endpoints, each endpoint being supported by a command or query application.  Each command application publishes to a message channel, returning an acknoledgement; each query application reads from a database and publishes to a message channel, returning the database result.  Four protected endpoints are shown, corresponding to four applications.

            https://eg.net/mss/music/command POST PUT DELETE
            https://eg.net/mss/money/command POST
            https://eg.net/mss/music/query GET
            https://eg.net/mss/money/query GET

1.  A CloudMQ supports some message channels.  Each channel is published-to by a command or query application; each channel is subscribed-to by a a persistence application (CloudLF).

            net.eg.mss.music.command
            net.eg.mss.music.query

1.  Some CloudLFs are driven by the CloudMQ.  Each such CLF is a persistence application; it subscribes to a message channel and writes to a database.  (Do the CLFs need EPs?)

            https://eg.net/mss/music/persist POST PUT DELETE
            https://eg.net/mss/money/persist POST PUT DELETE

1.  Some CloudSQLs provide the databases to which the persistence applications write and from which the query applications read.

            net.eg.mss.music.database
            net.eg.mss.money.database


## The Databases
The primary key of each table is the `id` field. The secondary key of each table is the combination of fields indicated by the asterisks.

Music Database (UDB):

        Artist (Long id, String name*)

Money Database (ODB):

        ArtistCount (Long id, String name*, Long count)

Note that, in general:

* *artist.id ≠ artistCount.id*, when *artist.name = artistCount.name*
* *artist.name ≠ artistCount.name*, when *artist.id = artistCount.id*.


## The Applications
UCA

        request: {type: "ArtistRequest", ...}
        requote: {type: "Requote", request: ${request}, requestId: 0, method: UPSERTION, stamp: 2021-12-12+00:00:00}
        resolve: {type: "Resolve", requote: ${requote}, respondId: 0, exitId: 0}
        respond: {type: "Respond", resolve: ${resolve}, status: OkAY}

UQA

        request: {type: "ArtistRequest", ...}
        requote: {type: "Requote", request: ${request}, requestId: 2, method: SELECTION, stamp: 2021-12-12+02:00:00}
        resolve: {type: "Resolve", requote: ${requote}, respondId: 2, exitId: 0, datum: {type: "Artist", ...}}
        respond: {type: "Respond", resolve: ${resolve}, status: OKAY}

OQA

        request: {type: "ArtistCountRequest", ...}
        requote: {type: "Requote", request: ${request}, requestId: 3, method: SELECTION, stamp: 2021-12-12+03:00:00}
        resolve: {type: "Resolve", requote: ${requote}, respondId: 3, exitId: 0, datum: {type: "ArtistCount", ...}}
        respond: {type: "Respond", resolve: ${resolve}, status: YAKO}
