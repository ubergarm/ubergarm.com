There's more than one way to configure a server: some are complex, some simple, some static, and some dynamic. I've definitely become a fan of using nginx to reverse proxy Docker containers to a specified subdomain e.g. *container.yourdomain.com*.

I spent a lot of time in the past tweaking nginx rewrite rules to give a dynamic feel while keeping the simplicity of a static config file.

I recently discovered jwilder's versitile [docker-gen](https://hub.docker.com/r/jwilder/docker-gen/) which listens on the Docker sock and updates config files based on the [Go text/template language](https://golang.org/pkg/text/template/).

While [jwilder/nginx-proxy](https://index.docker.io/u/jwilder/nginx-proxy/) is an out of the box solution, I wanted more control over the template and volume mounts to also serve up some static files.

This solution works great with the official nginx image and allows me to serve up static files off the host as well as reverse proxy Containers.

Here is an example docker-compose.yml:

```
# Config file will be templated and updated by docker-gen below 
nginx:
  image: nginx
  volumes:
    - /home/www/nginx/conf.d:/etc/nginx/conf.d
    - /home/www/nginx/html:/usr/share/nginx/html
  ports:
    - "80:80"
  environment:
    STATIC_HOST: yourdomain.com,www.yourdomain.com

# Dynamically auto update nginx config template 
gen:
  image: jwilder/docker-gen
  volumes_from:
    - nginx
  volumes:
    - /var/run/docker.sock:/tmp/docker.sock:ro
    - /home/www/nginx/conf.d:/etc/docker-gen/templates
  command: -notify-sighup nginx -watch -only-exposed /etc/docker-gen/templates/nginx.tmpl /etc/nginx/conf.d/default.conf

# lets blog!
ghost:
  image: ghost
  expose:
    - "2368"
  volumes:
    - /home/www/ghost:/var/lib/ghost
  environment:
    NODE_ENV: production
    VIRTUAL_HOST: blog.yourdomain.com
```

This assumes that your config files and such will be bind mounted into the host under the `/home/www` users home directory.

I updated the nginx.tmpl to add another server block with the root set to serve static files off of the top level domain name and www.yourdomain.com.  The STATIC_HOST env vars on the nginx container are readable at template generation.

After some testing and cleaning up, I should push a github repo or gist.  Comment if you have other ideas or to encourage me! ;)
