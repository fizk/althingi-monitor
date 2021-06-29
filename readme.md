## Network config
This repo is configured to be able to run inside the same network as [althingi-master](althingi-master). While the master defines its networks as:
```yml
networks:
  main-althingi-networks:
    driver: bridge
    name: main-althingi-networks
```

This repo defines its networking as:
```yml
networks:
  main-althingi-networks:
    external: true
    name: main-althingi-networks
```
The main application is defining a network called `main-althingi-networks` and this repo is then connecting to it as an external network.

This implies that the network needs to be created before this repo is started. After that, you can go ahead and run the containers

## Running the service
To run the monitoring service, simply run the docker-compose file:

```sh
docker compose up elasticsearch kibana metricbeat filebeat logstash -d
```

Now the monitor system is up and running, but Kibana does not have the required dashboards and search indexes. Do do that, the _metricbeat setup_ process has to be run. For it to run within the correct network, the docker-compose file is used.

```sh
docker compose run metricbeat bash -c "metricbeat setup -E setup.kibana.host=kibana:5601 -E output.elasticsearch.hosts=[\"elasticsearch:9200\"]"
```

Update network names and ports accordingly.

## Monitor dashboard
This docker-compose file exposes one port, **8081** which is where Kibana keeps it dashboards.


## Bootstrapping search
Elasticsearch needs indexes defined before accepting input. A cURL command can be run to populate Elastic search with the required index-mapping. This repo contains a docker-container which its only purpose it to run once, send a cURL instruction and exit.

When this docker-image is built, it wil copy the payload required to run the cURL (json files) and then set up a shell-script to make the actual HTTP call (via cURL). It then puts the shell-script (called `init.sh`) in the `CMS` command of the Dockerfile.

Because the shell-script is going to connect to a remote server/service (in this case the Elasticsearch cluster), it needs to know where to locate server. Pass in a environment variable to set Elasticsearch host and port, for example:

```sh
docker run -it -e MONITOR_ES=elasticsearch:9200 <docker-image-name>
```

| VAR        | TYPE           | DEFAULT             | DESCRIPTION                                                    |
| ---------- | -------------- | ------------------- | -------------------------------------------------------------- |
| MONITOR_ES | <host>:<port>  | elasticsearch:9200  | The host and port of Elasticsearch that hosts the monitor logs |
| SEARCH_ES  | <host>:<port>  | elasticsearch:9200  | The host and port of Elasticsearch that hosts the search logs  |