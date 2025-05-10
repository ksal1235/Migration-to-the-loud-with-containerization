## Deployment with Docker Compose:

All we have done until now required quite a lot of effort to create an image and launch an application inside it. We should not have to always run Docker commands on the terminal to get our applications up and running. There are solutions that make it easy to write declarative code in YAML, and get all the applications and dependencies up and running with minimal effort by launching a single command.

In this section, we will refactor the Tooling app POC so that we can leverage the power of Docker Compose.

#### 1. First, install Docker Compose on your workstation from

With Docker Desktop, Docker Compose is now integrated as a Docker CLI plugin, and you use it by typing docker compose instead of docker-compose (for stand alone docker compose tool).

#### 2. Begin to write the Docker Compose definitions with YAML syntax. The YAML file is used for defining services, networks, and volumes:

```
version: "3.9"
services:
  tooling_frontend:
    build: .
    ports:
      - "5000:80"
    volumes:
      - tooling_frontend:/var/www/html
```

- version: Is used to specify the version of Docker Compose API that the Docker Compose engine will connect to. This field is optional from docker compose version v1.27.0.
- service: A service definition contains a configuration that is applied to each container started for that service. In the snippet above, the only service listed there is tooling_frontend. So, every other field under the tooling_frontend service will execute some commands that relate only to that service. Therefore, all the below-listed fields relate to the tooling_frontend service.
- build
- port
- volumes
- links

  Let us fill up the entire file and test our application:

```
version: "3.9"
services:
  tooling_frontend:
    build: .
    ports:
      - "5001:80"
    volumes:
      - tooling_frontend:/var/www/html
    links:
      - db
  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: <The database name required by Tooling app >
      MYSQL_USER: <The user required by Tooling app >
      MYSQL_PASSWORD: <The password required by Tooling app >
      MYSQL_RANDOM_ROOT_PASSWORD: "1"
    volumes:
      - db:/var/lib/mysql
volumes:
  tooling_frontend:
  db:
```

Run the command to start the containers

``
docker-compose -f tooling.yaml  up -d
``

### Conclusion
![image](https://github.com/user-attachments/assets/24d06029-ac30-4b26-8dc8-15500b393889)

We have migrated our application running on virtual machines into the Cloud with containerization.


