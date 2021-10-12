Exploring HAProxy 

### Creating HAProxy Deployment with Kustomize 
- Official HAProxy image is used with Kustomize Generating configmap from `haproxy.cfg` file then the file gets mounted at `/usr/local/etc/haproxy`
- Currently config file only contains configs for `stats` page
- To deploy use `kustomize build . | kubectl apply -f -` 

---

- The following Section is following AcloudGuru instructions from their course [Hands-On with HAProxy Load Balancer](https://acloudguru.com/course/hands-on-with-haproxy-load-balancer)

### Generating Pages for each server 
```bash
# Create main_site directory
mkdir -v main_site

# Generate the desired 6 pages for the nginx instances
for site in `seq 1 2`; do for server in `seq 1 3`; do figlet Site$site - Server$server > main_site/site$site-server$server.html; done; done;

```

### Running nginx servers with generated pages 
```bash
for site in `seq 1 2`; do for server in `seq 1 3`; do podman run -dt --name nginx-site$site-server$server --publish 80$site$server:80 nginx  && podman cp main_site/site$site-server$server.html nginx-site$site-server$server:/usr/share/nginx/html/test.txt; done; done;
```
- Should run nginx with the following ports published 
  1. port `8011` `8012` `8013` for site1 
  2. port `8021` `8022` `8023` for site2

- 
<details>
<summary> Testing Site1 with curl </summary>
<p>

```bash
curl localhost:8011/test.txt
 ____  _ _       _           ____                           _
/ ___|(_) |_ ___/ |         / ___|  ___ _ ____   _____ _ __/ |
\___ \| | __/ _ \ |  _____  \___ \ / _ \ '__\ \ / / _ \ '__| |
 ___) | | ||  __/ | |_____|  ___) |  __/ |   \ V /  __/ |  | |
|____/|_|\__\___|_|         |____/ \___|_|    \_/ \___|_|  |_|
```

```bash
curl localhost:8012/test.txt
 ____  _ _       _           ____                          ____
/ ___|(_) |_ ___/ |         / ___|  ___ _ ____   _____ _ _|___ \
\___ \| | __/ _ \ |  _____  \___ \ / _ \ '__\ \ / / _ \ '__|__) |
 ___) | | ||  __/ | |_____|  ___) |  __/ |   \ V /  __/ |  / __/
|____/|_|\__\___|_|         |____/ \___|_|    \_/ \___|_| |_____|
```

```bash
 curl localhost:8013/test.txt
 ____  _ _       _           ____                          _____
/ ___|(_) |_ ___/ |         / ___|  ___ _ ____   _____ _ _|___ /
\___ \| | __/ _ \ |  _____  \___ \ / _ \ '__\ \ / / _ \ '__||_ \
 ___) | | ||  __/ | |_____|  ___) |  __/ |   \ V /  __/ |  ___) |
|____/|_|\__\___|_|         |____/ \___|_|    \_/ \___|_| |____/

```

</p>
</details>


---

### HaProxy config file
```h 
# Frontend site1
frontend site1
  bind *:80
  default_backend site1

# Backend site1 
backend site1 
  balance roundrobin
  server server1 127.0.0.1:8011
  server server2 127.0.0.1:8012
  server server3 127.0.0.1:8013

# Frontend site2 
frontend site2 
  bind *:81 
  default_backend site2 

# Backend site2 
backend site2 
  balance roundrobin 
  server server1 127.0.0.1:8021
  server server2 127.0.0.1:8022
  server server3 127.0.0.1:8023

```
