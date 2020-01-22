# Data Collection Setup

## Overview

![](figures/components.svg)

This repository contains everything necessary to setup the data collection 
pipeline used to collect pegasus composite events and visualize them
on an interactive dashboard. This includes 
[Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/index.html), 
[Logstash](https://www.elastic.co/guide/en/logstash/6.8/index.html), 
[Kibana](https://www.elastic.co/guide/en/kibana/6.8/index.html)
(the ELK stack), 
[RabbitMQ](https://www.rabbitmq.com/documentation.html), and 
[Grafana](https://grafana.com/docs/). 

When `pegasus-monitord` emits workflow events, the following will take place:

**on a submit host**
- a workflow job completes and `pegasus-monitord` sends a message using the AMQP 
  protocol to a RabbitMQ message queue, which is running on the host that you have
  deployed this pipeline to

**on the host you've deployed this infrastructure to**
- RabbitMQ will enqueue the received message
- Logstash will consume the message from the message queue
- Logstash will then insert that message into Elasticsearch under the index 
    `[pegasus-composite-events-]YYYY.MM` (e.g. `pegasus-composite-events-2019.11`)
- Kibana will make the logs entered in Elasticsearch visible through a web 
    interface where they can be querried/filtered
- Grafana will display workflow data on an interactive dashboard made visible 
    through a web interface

To set set up this data collection pipeline, a few prerequisites must be met.

## Prerequisites 
The host on which this will be set up must have the following:
- a static ip address
- following ports must be unused:
    - for RabbitMQ
        - 5672
        - 15672
    - for Elasticsearch
        - 9200
    - for Logstash
        - 9600
    - for Kibana
        - 5601
    - for Grafana
        - 3000
- [Docker v17.02+](https://docs.docker.com/install/)
- [Docker Compose v3.5](https://docs.docker.com/compose/install/)

## Setup
1. The following data directories must be given read/write permissions.
    ```
    dibbs-data-collection-setup/
    ├── README.md
    ├── docker-compose.yml
    ├── elasticsearch
    │   └── data                <-- directory that will be read/written from/to by elasticsearch
    │       └── README.md
    ├── grafana
    │   ├── dashboards
    │   │   ├── all.yml
    │   │   └── dibbs_dashboard.json 
    │   ├── data                <-- directory that will be read/written from/to by grafana
    │   │   └── README.md
    │   └── datasources
    │       └── all.yaml
    ├── kibana
    │   └── data                <-- directory that will be read/written from/to by kibana
    │       └── README.md
    ├── logstash
    │   ├── logstash.conf
    │   └── workflow-events.json
    └── rabbitmq
        ├── data                <-- directory that will be read/written from/to by rabbitmq
        │   └── README.md
        ├── definitions.json
        └── rabbitmq.config
    ```
2. While inside the `dibbs-data-collection-setup` directory, run the following command:
    ```
    docker-compose up
    ```
    or to run in detached mode
    ```
    docker-compose up -d
    ```
    - note that `docker-compose down` can be used to stop the containers

## Getting Data
Say that this setup is running on a host `dibbs.isi.edu`, then the following must
be included in the **pegasus configuration file** used to run your pegasus
workflows:

```
pegasus.monitord.encoding = json
pegasus.catalog.workflow.amqp.url = amqp://friend:donatedata@dibbs.isi.edu:5672/prod/workflows
```

## Viewing Data
The data coming into this pipeline can be viewed in a browser through either
Kibana or Grafana.

### Kibana
Say that this setup is running on a host `dibbs.isi.edu`, then in your browser
navigate to `dibbs.isi.edu:5601`.

If this is the first time opening up Kibana, we need to create two index patterns
before being able to view the logs stored in Elasticsearch.
1. Click `Management` on the left hand menu bar
2. Click `Index Patterns` under the `Kibana` header on the left hand side of the page
3. Under `Create index pattern` -> `Step 1 of 2: Define index pattern` -> `Index pattern`
    enter `pegasus-composite-events-*`, then click `Next step`
4. Under `Create index pattern` -> `Step 2 of 2: Configure settings` -> `Time Filter field name`
    select `@timestamp`, then click `Create index pattern`
5. Now click `Create index pattern`, and repeat steps **3** and **4**, except 
    enter `workflow-events-*` instead of `pegasus-composite-events-*`

Click on the `Discover` tab on the lefthand menu to view the data. 

### Grafana
Say that this setup is running on a host `dibbs.isi.edu`, then in your browser
navigate to `dibbs.isi.edu:3000`.

Use the following credentials to log in:
```
Username: admin
Password: dibbs
```

Under `Recently viewed dashboards` select `Main` to view the dashboard.

## Monitoring Incoming Messages with RabbitMQ
RabbitMQ has a web interface that shows when messages are received,
how many messages are queued up, etc. 

Say that this setup is running on host `dibbs.isi.edu`, then in your browser
navigate to `dibbs.isi.edu:15672`.

Use the following credentials to log in:
```
Username: dibbs
Password: dibbs
```
