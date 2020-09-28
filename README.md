# 🚀 Freeipa Integrated with NextCloud 🚀
> FreeIPA
> Nextcloud
> MariaDB "mysql"
# ـــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــ


### 1- Install Docker-compose ✨ 
```sh
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.27.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
```sh
```

```sh
$ sudo chmod +x /usr/local/bin/docker-compose
```
### 2- Test the installation ✨ 
```sh
$ docker-compose --version
docker-compose version 1.27.3, build 1110ad01
```
# ـــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــ

### 1- Create freeipa Container as below ✨ 
- define your version of your docker-compose file :
```sh
version: "3.7"
services:
```
- define your container name "freeipa" , define the "image" you will use for your freeipa image.

```sh
freeipa:
    image: freeipa/freeipa-server:centos-8
```
- The container is always restarts.
```sh
restart: always
```
- change "ipa.ldap.local" to your Hostname

``` sh
hostname: ipa.ldap.local
environment:
    - IPA_SERVER_HOSTNAME=ipa.ldap.local
tty: true
stdin_open: true
cap_add:
    - NET_ADMIN
```

- All data beyond what lives in the database is stored in the docker volume as you defined it,That means your data is saved even if the container crashes, is stopped or deleted.

```sh
        volumes:
          - /sys/fs/cgroup:/sys/fs/cgroup:ro
          - ./data/freeipa:/data:Z
```

- Disable IPV6
```sh
        sysctls:
          - net.ipv6.conf.lo.disable_ipv6=0
          - net.ipv6.conf.all.disable_ipv6=0
        security_opt:
          - "seccomp:unconfined"
```

- Configure the FreeIPA master using the inputs provided. For unattended initial installation, if you want to allow dns server, delete `#` from the beginning.

```sh
        command:
          - -U
          - --domain=ldap.local         # add your domain name only
          - --realm=ldap.local
          - --http-pin=UltraS3cure
          - --dirsrv-pin=UltraS3cure
          - --ds-password=UltraS3cure       
          - --admin-password=UltraS3cure    # your default password
          - --no-host-dns
          #- --no-dnssec-validation
          #- --setup-dns
          #- --auto-forwarders
          #- --allow-zone-overlap
          - --unattended
```

- Expose the freeipa ports :

```sh
        ports:
          #- "53:53/udp"
          #- "53:53"
          - "80:80"
          - "443:443"
          - "389:389"
          - "636:636"
          - "88:88"
          - "464:464"
          - "88:88/udp"
          - "464:464/udp"
          - "123:123/udp"
          - "7389:7389"
          - "9443:9443"
          - "9444:9444"
          - "9445:9445"
```

- The Important step for make the Integration between freeipa with Nextcloud , it makes the link between 2 containers.
- 

```sh 
depends_on:
    - nextcloud         # name of the container, you need to make the link.      
```
- define the network card name.

```sh
networks:
    - nextcloud_network
```
# ـــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــ

### 2- Create Nextcloud Container as below ✨ 

-  define the nextcloud service on docker-compose file undername `nextcloud`.
-  select the image you will use , I use the latest version.
-  define the name of the container.
```sh
    nextcloud:
        image: nextcloud:latest
        container_name: nextcloud-app
```
- Expose the port '8080' into local machine, and port '80' from docker container.

```sh
        ports: 
            - 8080:80
```
- All data beyond what lives in the database is stored in the docker volume as you defined it,That means your data is saved even if the container crashes, is stopped or deleted.
- `.` it's mean the same directory which the docker-compose file there.
```sh
        volumes:
            - ./data/nextcloud:/var/www/html
            - ./data/app/config:/var/www/html/config
            - ./data/app/custom_apps:/var/www/html/custom_apps
            - ./data/app/data:/var/www/html/data
            - ./data/app/themes:/var/www/html/themes
            - /etc/localtime:/etc/localtime:ro
                        
```
- define the environment methods , and write your domain name or your IP.
- `cloud.ldap.local` <-- domain name
```sh
        environment:
            - VIRTUAL_HOST=cloud.ldap.local
            - LETSENCRYPT_HOST=cloud.ldap.local
            - LETSENCRYPT_EMAIL=hakim@gmail.com
```

```sh
        restart: always
        networks:
            - nextcloud_network
```
- Nextcloud needs database will depend on it , `db` it mean the database name of service.
```sh
        depends_on:
            - db
```
# ـــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــ

### 3- Create Maria-DB "mysql" Database Container as below ✨ 
- define the service name `db` , Image name `mariadb` , Container name `nextcloud-mariadb`, network card name `nextcloud_network`.

```sh
    db:
        image: mariadb
        container_name: nextcloud-mariadb
        restart : always
        networks:
            - nextcloud_network
```
- make mount volume for `database`.
```sh
        volumes:
            - ./data/db:/var/lib/mysql
            - /etc/localtime:/etc/localtime:ro
```

- define the database name,password,user name.
```sh
        environment:
            - MYSQL_ROOT_PASSWORD=toor
            - MYSQL_PASSWORD=mysql
            - MYSQL_DATABASE=nextcloud
            - MYSQL_USER=nextcloud
```
- define on the root level the volumes names for create automatickly
```sh
volumes:
    nextcloud:
    db:
    datafreeipa:
```
- define network card for creation :
```sh
networks:
    nextcloud_network:
```
# ـــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــ
Edit your hosts file if you don't have DNS server, and add this lines , your local IP or your localHost with your hostname for Freeipa , Nextcloud.

![hosts](https://i.ibb.co/dDzm8HL/hosts.png)

### - Run docker-compose 
```sh
docker-compose up -d
```
if you want to show the logs , run this command
```sh
docker-compose logs -f
```
- wait for freeipa finishing the Installation and configuration.
# ـــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــ

- Configure Nextcloud application for start and Integration
- open `browser` and go to `http://cloud.ldap.local:8080`
- user name is `admin` password `admin`
- Press `Finish setup`

![Image3](https://i.ibb.co/4VrTLfW/nc3.png)

![Image4](https://i.ibb.co/jVkShdk/nc4.png)

![Image5](https://i.ibb.co/9wPn6kk/nc5.png)

![Image6](https://i.ibb.co/Mpc1L7m/nc6.png)

![Image7](https://i.ibb.co/6H3ynXF/nc7.png)

![Image8](https://i.ibb.co/qM4WNx9/nc8.png)

![Image9](https://i.ibb.co/tJpTPYw/nc9.png)

![Image10](https://i.ibb.co/t8p9KZJ/nc10.png)

![Image11](https://i.ibb.co/L8sGJrt/nc11.png)

![Image12](https://i.ibb.co/cQgDT2y/nc12.png)

![Image13](https://i.ibb.co/C7nb9MJ/nc13.png)

![Image14](https://i.ibb.co/VVBRHCV/nc14.png)


# ـــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــ
- Login to LDAP Server and testing for adding user and go back to nextcloud to join by the same user.
- open `browser` and go to `http://ipa.ldap.local`
- user name `admin` password `UltraS3cure`

![Image15](https://i.ibb.co/HhNcjxG/ldap15.png)

![Image16](https://i.ibb.co/nzY11RN/ldap16.png)

![Image17](https://i.ibb.co/n3KvD56/ldap17.png)

# ـــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــــ

- go back to nextcloud for test the `hakim` user.

![Image18](https://i.ibb.co/tLHtDBL/nc18.png)

![Image19](https://i.ibb.co/5vGR4pt/nc19.png)

![Image20](https://i.ibb.co/7WrMFtT/nc20.png)

![Image21](https://i.ibb.co/MSq0jjS/nc21.png)






















docker network rm $(docker network ls -q) && docker volume rm $(docker volume ls -q)
