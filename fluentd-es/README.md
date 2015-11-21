# fluentd-es

Docker containers log to elsticsearch using fluentd

* installed fluentd(actually td-agent) by apt-get 
* installed fluentd-elasticsearch plugin
* change user `td-agent` to `root`

## How To Run

Set up `td-agent.conf`.

```sh
$ docker run -d \
    --name fluentd-es \
    -v [the path for td-agent.conf]:/etc/td-agent/td-agent.conf:ro \
    ermaker/fluentd-es
```

For logging docker logs,

Set up `td-agent.conf` like this:

```
<source>
  type tail
  format json
  time_key time
  path /var/lib/docker/containers/*/*-json.log
  pos_file /var/lib/docker/containers/containers.log.pos
  time_format %Y-%m-%dT%H:%M:%S
  tag docker.*
</source>

...

<match docker.**>
  type elasticsearch
  include_tag_key true
  hosts es:9200
  logstash_format true
  logstash_prefix docker 

  # buffer
  buffer_type file
  buffer_path /var/lib/docker/fluentd/buffer/container.*.buffer
  buffer_chunk_limit 8m
  buffer_queue_limit 10000
  retry_limit 17
  flush_interval 5
</match>
```

```sh
$ docker run -d \
    --name fluentd-es \
    -v [the path for td-agent.conf]:/etc/td-agent/td-agent.conf:ro \
    -v /var/lib/docker:/var/lib/docker \
    ermaker/fluentd-es
```

## Refrence

* kubernetes logging
  * https://github.com/GoogleCloudPlatform/kubernetes/blob/master/contrib/logging/fluentd-es-image/Dockerfile
