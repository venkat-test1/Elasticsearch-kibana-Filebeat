#### root@ip-172-31-38-109:~# history
    1  curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
    2  vi docker-compose1.yaml
            version: "3"
            services:
                elasticsearch:
                    image: "docker.elastic.co/elasticsearch/elasticsearch:7.2.0"
                    environment:
                        - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
                        - "discovery.type=single-node"
                    ports:
                        - "9200:9200"
                    volumes:
                        - elasticsearch_data:/usr/share/elasticsearch/data

                kibana:
                    image: "docker.elastic.co/kibana/kibana:7.2.0"
                    ports:
                        - "5601:5601"
            volumes:
                elasticsearch_data:
    3  vi docker-compose2.yaml
              version: "3"
              services:
                  filebeat:
                      image: "docker.elastic.co/beats/filebeat:7.2.0"
                      user: root
                      volumes:
                          - /etc/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
                          - /var/lib/docker:/var/lib/docker:ro
                          - /var/run/docker.sock:/var/run/docker.sock
                  nginx:
                   image: "nginx"
                      ports:
                          - "80:80"
              volumes:
                  elasticsearch_data:
    4  cd /etc
    5  vi filebeat.yml
              filebeat.inputs:
              - type: container
                paths:
                  - '/var/lib/docker/containers/*/*.log'

              processors:
              - add_docker_metadata:
                  host: "unix:///var/run/docker.sock"

              - decode_json_fields:
                  fields: ["message"]
                  target: "json"
                  overwrite_keys: true

              output.elasticsearch:
                hosts: ["elasticsearch:9200"]
                indices:
                  - index: "filebeat-%{[agent.version]}-%{+yyyy.MM.dd}"

              logging.json: true
              logging.metrics.enabled: false

    6  cd ~
    7  docker-compose -f docker-compose1.yaml -f docker-compose2.yaml up
    
    Log file should be like below:
                             "o.e.c.r.a.AllocationService", "cluster.name": "docker-cluster", "node.name": "dce78d7253ae", "cluster.uuid": "t4mB96dLSyKoK47512EF-w", "node.id": "a7GrWiRySTuRdyMF_ZMaaQ",  "message": "Cluster health status changed from [YELLOW] to [GREEN] (reason: [shards started [[.kibana_task_manager][0]] ...])."  }
                  kibana_1         | {"type":"log","@timestamp":"2021-01-11T03:11:02Z","tags":["info","migrations"],"pid":1,"message":"Creating index .kibana_1."}
                  elasticsearch_1  | {"type": "server", "timestamp": "2021-01-11T03:11:02,484+0000", "level": "INFO", "component": "o.e.c.m.MetaDataCreateIndexService", "cluster.name": "docker-cluster", "node.name": "dce78d7253ae", "cluster.uuid": "t4mB96dLSyKoK47512EF-w", "node.id": "a7GrWiRySTuRdyMF_ZMaaQ",  "message": "[.kibana_1] creating index, cause [api], templates [], shards [1]/[1], mappings [_doc]"  }
                  elasticsearch_1  | {"type": "server", "timestamp": "2021-01-11T03:11:02,490+0000", "level": "INFO", "component": "o.e.c.r.a.AllocationService", "cluster.name": "docker-cluster", "node.name": "dce78d7253ae", "cluster.uuid": "t4mB96dLSyKoK47512EF-w", "node.id": "a7GrWiRySTuRdyMF_ZMaaQ",  "message": "updating number_of_replicas to [0] for indices [.kibana_1]"  }
                  elasticsearch_1  | {"type": "server", "timestamp": "2021-01-11T03:11:02,591+0000", "level": "INFO", "component": "o.e.c.r.a.AllocationService", "cluster.name": "docker-cluster", "node.name": "dce78d7253ae", "cluster.uuid": "t4mB96dLSyKoK47512EF-w", "node.id": "a7GrWiRySTuRdyMF_ZMaaQ",  "message": "Cluster health status changed from [YELLOW] to [GREEN] (reason: [shards started [[.kibana_1][0]] ...])."  }
                  kibana_1         | {"type":"log","@timestamp":"2021-01-11T03:11:02Z","tags":["info","migrations"],"pid":1,"message":"Pointing alias .kibana to .kibana_1."}
                  kibana_1         | {"type":"log","@timestamp":"2021-01-11T03:11:02Z","tags":["info","migrations"],"pid":1,"message":"Finished in 275ms."}
                  kibana_1         | {"type":"log","@timestamp":"2021-01-11T03:11:02Z","tags":["listening","info"],"pid":1,"message":"Server running at http://0:5601"}
                  elasticsearch_1  | {"type": "server", "timestamp": "2021-01-11T03:11:02,765+0000", "level": "INFO", "component": "o.e.c.m.MetaDataMappingService", "cluster.name": "docker-cluster", "node.name": "dce78d7253ae", "cluster.uuid": "t4mB96dLSyKoK47512EF-w", "node.id": "a7GrWiRySTuRdyMF_ZMaaQ",  "message": "[.kibana_1/pG-jZJHZT5G63Oa9z5bFuA] update_mapping [_doc]"  }
                  kibana_1         | {"type":"log","@timestamp":"2021-01-11T03:11:03Z","tags":["status","plugin:spaces@7.2.0","info"],"pid":1,"state":"green","message":"Status changed from yellow to green - Ready","prevState":"yellow","prevMsg":"Waiting for Elasticsearch"}


    
Now we can access Kibana using below url

![image](https://user-images.githubusercontent.com/77243596/104145770-4fbc6280-53ee-11eb-876b-81455280e56a.png)

The first thing you have to do is to configure the ElasticSearch indices that can be displayed in Kibana.

![image](https://user-images.githubusercontent.com/77243596/104145851-a1fd8380-53ee-11eb-9059-f16ef1b2a3e0.png)

You can use the pattern filebeat-* to include all the logs coming from FileBeat.

You also need to define the field used as the log timestamp. You should use @timestamp as shown below:

![image](https://user-images.githubusercontent.com/77243596/104145920-d5d8a900-53ee-11eb-84d3-64015bbd90d0.png)

we can see all container logs in below page

![image](https://user-images.githubusercontent.com/77243596/104145962-0a4c6500-53ef-11eb-9866-d2c91542eef6.png)

ngnix container logs

![image](https://user-images.githubusercontent.com/77243596/104146013-336cf580-53ef-11eb-911f-06fc99f36b1e.png)

ngnix container logs when we sent GET request through Postman.

![image](https://user-images.githubusercontent.com/77243596/104146110-8b0b6100-53ef-11eb-8f98-9cb81542f44d.png)

ngnix container logs when we sent POST request through Postman.

![image](https://user-images.githubusercontent.com/77243596/104146076-6911de80-53ef-11eb-93a2-f66c15431550.png)




