This tutorial teaches FIWARE users about CRUD Operations.

The tutorial builds on the data created in the previous [stock management example](http://fiware.github.io/tutorials.Entity-Relationships/) and introduces the concept of [CRUD operations](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete), allowing users to manipulate the data held within the context.

The tutorial uses [cUrl](https://ec.haxx.se/) commands throughout, but is also available as [Postman documentation](http://fiware.github.io/tutorials.CRUD-Operations/).

[![Run in Postman](https://run.pstmn.io/button.svg)](https://www.getpostman.com/collections/0671934f64958d3200b3)

# Contents

- [Data Entities](#data-entities)
  * [Entities within a stock management system](#entities-within-a-stock-management-system)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
  * [Docker](#docker)
  * [Cygwin](#cygwin)
- [Start Up](#start-up)

# Data Entities

Within the FIWARE platform, an entity represents the state of a physical or conceptural object which exists in the real world.

## Entities within a stock management system

Within our simple stock management system, currently have four types of entity. The relationship between our entities is defined as shown:

![](https://fiware.github.io/tutorials.Entity-Relationships/img/entities.png)

* A **Store** is a real world bricks and mortar building. Stores would have properties such as:
  + A name of the store e.g. "Checkpoint Markt"
  + An address "Friedrichstraße 44, 10969 Kreuzberg, Berlin"
  + A phyiscal location  e.g. *52.5075 N, 13.3903 E*
* A **Shelf** is a real world device to hold objects which we wish to sell. Each shelf would have properties such as:
  + A name of the shelf e.g. "Wall Unit"
  + A phyiscal location  e.g. *52.5075 N, 13.3903 E*
  + A maximum capacity
  + An association to the store in which the shelf is present
* A **Product** is defined as something that we sell - it is conceptural object. Products would have properties such as:
  + A name of the product e.g. "Vodka"
  + A price e.g. 13.99 Euros
  + A size e.g. Small
* An **Inventory Item** is another conceptural entity, used to assocate products, stores, shelves and physical objects. It would have properties such as:
  + An assocation to the product being sold
  + An association to the store in which the product is being sold
  + An association to the shelf where the product is being displayed
  + A stock count of the quantity of the product available in the warehouse
  + A stock count of the quantity of the product available on the shelf


As you can see, each of the entities defined above contain some properties which are liable to change. A product could change its price, stock could be sold and the shelf count of stock could be reduced and so on.


# Architecture

This application will only make use of one FIWARE component - the [Orion Context Broker](https://catalogue.fiware.org/enablers/publishsubscribe-context-broker-orion-context-broker). Usage of the Orion Context Broker is sufficient for an application to qualify as *“Powered by FIWARE”*.

Currently, the Orion Context Broker relies on open source [MongoDB](https://www.mongodb.com/) technology to keep persistence of the context data it holds. Therefore, the architecture will consist of two elements:

* The Orion Context Broker server which will receive requests using [NGSI](http://fiware.github.io/specifications/ngsiv2/latest/)
* The underlying MongoDB database associated to the Orion Context Broker server

Since all interactions between the two elements are initiated by HTTP requests, the entities can be containerized and run from exposed ports. 

![](https://fiware.github.io/tutorials.Entity-Relationships/img/architecture.png)

# Prerequisites

## Docker

To keep things simple both components will be run using [Docker](https://www.docker.com). **Docker** is a container technology which allows to different components isolated into their respective environments. 

* To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
* To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)
* To install Docker on Linux follow the instructions [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker applications. A [YAML file](https://raw.githubusercontent.com/Fiware/tutorials.Entity-Relationships/master/docker-compose.yml) is used configure the required
services for the application. This means all container sevices can be brought up in a single commmand. Docker Compose is installed by default as part of Docker for Windows and  Docker for Mac, however Linux users will need to follow the instructions found [here](https://docs.docker.com/compose/install/)

## Cygwin 

We will start up our services using a simple bash script. Windows users should download [cygwin](www.cygwin.com) to provide a command line functionality similar to a Linux distribution on Windows. 


# Start Up

All services can be initialised from the command line by running the bash script provided within the repository:

```bash
./services start
```

This command will also import seed data from the previous [Store Finder tutorial](https://github.com/Fiware/tutorials.Entity-Relationships) on startup.



# What is CRUD?

**Create**, **Read**, **Update** and **Delete** are the four basic functions of persistent storage.  These operations are usually referred to using the acronym **CRUD**. Within a database each of these operations map directly to a series of commands, however the relationship with a RESTful API is slightly more complex.

The Orion Context Broker server uses [NGSI](http://fiware.github.io/specifications/ngsiv2/latest/) to manipulate the context data. As a RESTful API, it follows the standard conventions when mapping HTTP verbs to requests to manipulate the data held within the context. 

## Entity CRUD Operations

| HTTP Verb   | `/v2/entities`  | `/v2/entities/<entity-id>`  |
|-----------  |:--------------: |:--------------------------: |
| **POST**    | CREATE a new entity and add to the context.  | CREATE or UPDATE an attribute of a specified entity. |
| **GET**     | READ entity data from the context.<br><br>This will return data from multiple entities. The data can be filtered.  | READ entity data from a specified entity.<br><br>This will return data from a single entity only.<br><br>The data can be filtered.  | 
| **PUT**     | :x:   | :x:   |
| **PATCH**   | :x:   | :x:   |
| **DELETE**  | :x:  | DELETE an entity from the context   | 

## Attribute CRUD Operations

| HTTP Verb   | `/v2/entities/<entity-id>/attrs`  | `/v2/entities/<entity-id>/attrs/<attribute>`  | `/v2/entities/<entity-id>/attrs/<attribute>/value`  |
|-----------  |:--------------------------------: |:--------------------------------------------: |:------------------------------------------------------: |
| **POST**  |  :x:   | :x:   | :x:   |
| **GET**   |  :x:   | :x:   | READ the value of an attribute from a specified entity.<br>This will return a single field.   |
| **PUT**   |  :x:   | :x:   | UPDATE the value of single attribute from a specified entity.   |
| **PATCH** |  UPDATE one or more existing attributes from an existing entity.  | :x:   | :x:   |
| **DELETE**|  :x: | DELETE an existing attribute from an existing entity.  | :x:  |


