# Working with local cluster

This doc covers how to build the plugin and load it on a local Opensearch cluster.

## Building the plugin

```sh
./gradlew build -Dopensearch.version=2.18.0
```

This should create `opensearch-arrow.zip` file under `build/distributions`.

## Running Opensearch cluster

We will run opensearch with docker.

Pull the image (one time):
```sh
docker pull opensearchproject/opensearch:latest
```

Run a container:
```sh
docker run -d --name opensearch-node1 \
    -p 9200:9200 -p 9600:9600 \
    -e "DISABLE_INSTALL_DEMO_CONFIG=true" \
    -e "DISABLE_SECURITY_PLUGIN=true" \
    -e "discovery.type=single-node" opensearchproject/opensearch:latest
```

## Installing the plugin

Copy plugin to docker container:
```sh
docker cp build/distributions/opensearch-arrow.zip opensearch-node1:/usr/share/opensearch/
```

Install plugin:
```sh
docker exec -it opensearch-node1 \
    /usr/share/opensearch/bin/opensearch-plugin install file:/usr/share/opensearch/opensearch-arrow.zip
```

Restart the cluster:
```sh
docker restart opensearch-node1
```

Verify installation:
```sh
docker exec -it opensearch-node1 /usr/share/opensearch/bin/opensearch-plugin list
```

You will also see plugin loaded logs:
```
2024-11-12 23:21:06 [2024-11-13T04:21:06,546][INFO ][o.o.p.PluginsService     ] [3ddd98a7db9e] loaded plugin [opensearch-arrow]
```

## Testing

Hit the cluster:
```sh
curl -X GET "http://localhost:9200/"
```

Logs:

```sh
docker logs opensearch-node1
```

Uninstall plugin:
```sh
docker exec -it opensearch-node1 \
    /usr/share/opensearch/bin/opensearch-plugin remove opensearch-arrow
```