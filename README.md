# Elasticsearch-kibana-Filebeat installation in Docker
## Installation steps
### 1. Docker installation:
     sudo apt-get update -y && sudo apt-get install docker.io -y
### 2. Docker Compose installation:
      curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
      chmod +x /usr/local/bin/docker-compose
      docker-compose version
### 3. Docker compose files.
####       -> elasticsearch with Kibana docker composefile.
              version: "3"
              services:
                  elasticsearch:
                      image: "docker.elastic.co/elasticsearch/elasticsearch:7.2.0"
                      environment
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
                  
####       ->  filebeat with nginx docker composefile.
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
  ####       ->  create the filebeat configuration file in /etc/filebeat.yml:
                 
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
 ####       ->  docker-compose -f docker-compose1.yml -f docker-compose2.yml up
                  
