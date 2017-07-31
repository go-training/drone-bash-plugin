# drone-bash-plugin

Example Bash Plugin for Drone CI/CD service.

## How to write bash plugin

This provides a brief tutorial for creating a Drone webhook plugin, using simple shell scripting, to make an http requests during the build pipeline. The below example demonstrates how we might configure a webhook plugin in the Yaml file:

```yml
pipeline:
  webhook:
    image: foo/webhook
    url: http://foo.com
    method: post
    body: |
      hello world
```

Create a simple shell script that invokes curl using the Yaml configuration parameters, which are passed to the script as environment variables in uppercase and prefixed with PLUGIN_.

```sh
#!/bin/sh

curl \
  -X ${PLUGIN_METHOD} \
  -d ${PLUGIN_BODY} \
  ${PLUGIN_URL}
```

Create a Dockerfile that adds your shell script to the image, and configures the image to execute your shell script as the main entrypoint.

```dockerfile
FROM alpine
ADD post.sh /bin/
RUN chmod +x /bin/post.sh
RUN apk -Uuv add curl ca-certificates
ENTRYPOINT /bin/post.sh
```

Build and publish your plugin to the Docker registry. Once published your plugin can be shared with the broader Drone community.

```sh
$ docker build -t foo/webhook .
# login your dockerhub account.
$ docker login
$ docker push foo/webhook
```

Execute your plugin locally from the command line to verify it is working:

```sh
docker run --rm \
  -e PLUGIN_METHOD=post \
  -e PLUGIN_URL=http://foo.com \
  -e PLUGIN_BODY="hello world" \
  foo/webhook
```

Result:

```
<head><title>400 Bad Request</title></head>
<body bgcolor="white">
<center><h1>400 Bad Request</h1></center>
<hr><center>nginx/1.10.3 (Ubuntu)</center>
</body>
</html>
```
