# Kibana


1. docker pull docker.elastic.co/kibana/kibana:8.4.1
2. docker run --name kb-01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.4.1