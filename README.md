# Local Traefik Docker Setup

This is a basic traefik setup running with docker including HTTPS. The following instructions are optimized for using **WSL 2** (Windows Subsystem for Linux) under Windows 10/11. If you are not familiar with WSL and need to install it, follow the official [installation guide](https://docs.microsoft.com/windows/wsl/install).

I am using **Ubuntu 20.04.4 LTS** as distro. You can simply [download](https://apps.microsoft.com/store/detail/ubuntu-20044-lts/9MTTCL66CPXJ) it from the Microsoft Store.

## Getting started

### Clone repository (recommended folder structure)

Clone this repository into a directory. If you're using linux (WSL) you can use e.g. `/srv/www/traefik`. I like to setup my projects like this:

```
|- /srv/www/
  |- traefik/
  |- webapp/
  |- webservice/
  |- ...
```

### Check or install docker

Make sure you have docker (docker-compose installed). You can check it by getting the version. Simply run `docker -v` and `docker-compose -v` in your terminal. If a version is printed you're good to go otherwise you need to install docker. Just follow the official [installation guide](https://docs.docker.com/get-docker/).

### SSL certificates

#### traefik.me

By default this configuration uses Let's Encrypt wildcard certificates for traefik.me. https://traefik.me is a magic domain that provides wildcard DNS for any IP address. That means no editing of the hosts file and a perfectly working SSL certificate.

#### Self-signed

If your don't want to use traefik.me you can create your own self-signed certificate else skip the next two steps.

##### Add hosts

To use a custom host name you need to add it to the hosts file of your system. In my case I'm using windows, so I added it at the bottom of `C:\Windows\System32\drivers\etc\hosts`:

```
# Custom hosts
127.0.0.1       traefik.docker.local
```

For every new service you need to add a new host e.g.:

```
# Custom hosts
127.0.0.1       traefik.docker.local
127.0.0.1       webapp.docker.local
127.0.0.1       webservice.docker.local
```

##### Create a self-signed certificate

Next you need to create a self signed SSL certificate. If you also set up a custom host, you will change `*.docker.local` to your host name in the following command. Run this command inside `docker/cert/`:

```
openssl req -x509 -out traefik.crt -keyout traefik.key \
  -newkey rsa:2048 -nodes -sha256 \
  -subj '/CN=*.docker.local' -extensions EXT -config <( \
  printf "[dn]\nCN=*.docker.local\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:*.docker.local\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```

### Change host in docker-compose.yml

If you use a different host than `traefik.traefik.me` for the traefik dashboard, you need to change the traefik host in the labels in the docker-compose.yml (replace `<host>` with the host you want to use).

```
- "traefik.http.routers.traefik-dashboard.rule=Host(`<host>`)"
```

If your using a self-signed certificate with `docker.local` as host you can copy the following label:

```
- "traefik.http.routers.traefik-dashboard.rule=Host(`traefik.docker.local`)"
```

### Add new docker network

Before you can start the docker container you need to add a new docker network (replace `<network-name>` with a name of your choice):

```
docker network create <network-name>
```

I named the network `proxy`. If you want to use a different name, you also need to update it in your docker-compose.yml:

```
networks:
  default:
    external:
      name: "<network-name>"
```

For every new service that should use traefik, you need to add this code snippet above to your docker-compose.yml. Otherwise docker has no access to the traefik network.

### Start docker container

Now you can start the docker container by running the `docker-compose up` command. If you want to run the container in the background use `docker-compose up -d`.

After the container is built and up running, you can visit the traefik dashboard under the host you defined. In my case I just open `https://traefik.traefik.me` in my browser of choice.