The nginx-proxy-with-css sets up a container running nginx and [docker-gen][1].  docker-gen generates reverse proxy configs for nginx and reloads nginx when containers are started and stopped. For more details have a look at [Automated Nginx Reverse Proxy for Docker][2].

### Usage

Start the `nginx-proxy-with-css`:

    $ docker run -d -p 80:80 -v /var/run/docker.sock:/tmp/docker.sock:ro bde202/nginx-proxy-with-css

Start the containers you want to be proxied with the environment variable `VIRTUAL_HOST=subdomain.youdomain.com`

    $ docker run -e VIRTUAL_HOST=foo.bar.com  ...

#### Configure the proxy
This Docker image is an extension of [jwilder/nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy/). Hence, all the configuration options of the `jwilder/nginx-proxy` image also apply on this image.

The containers being proxied must [expose](https://docs.docker.com/reference/run/#expose-incoming-ports) the port to be proxied, either by using the `EXPOSE` directive in the `Dockerfile` or by using the `--expose` flag on the `docker run` or `docker create` command.

Provided your DNS is setup to forward `foo.bar.com` to a host running the `nginx-proxy-with-css` container, the request will be proxied to the container with the `VIRTUAL_HOST=foo.bar.com` environment variable set.

#### Configure the CSS insertion
Configure the environment variable `CSS_SOURCE` on the service for which you want to activate the automatic insertion of a (or many) CSS in the proxied HTML pages. The value of the `CSS_SOURCE` must be one of:
- flink
- hadoop
- hdfs
- sextant
- solr
- spark-master
- strabon

This list will be extended in the future.

### Adding a new CSS insertion

1. Add your CSS files in the `/extra-css` directory. 
2. Insert some sub_filter rules in the `nginx.tmpl` file:

```
# This is where are located all the CSS insertions
{{ if (eq $css "inverted") }}
sub_filter '</head>' '<link rel="stylesheet" type="text/css" href="/extra-css/inverted.css"/></head>';
{{ end }}

# My own CSS for a specific service
{{ if (eq $css "myservice") }}
sub_filter '</head>' '<link rel="stylesheet" type="text/css" href="/extra-css/myservice/custom_style_override.css"/></head>';
{{ end }}
```

Then run your service with the environment variable CSS_SOURCE set to
"myservice".
