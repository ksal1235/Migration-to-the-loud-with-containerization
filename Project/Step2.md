# Prepare database schema
Now you need to prepare a database schema so that the Tooling application can connect to it.

## 1. Clone the Tooling-app repository from here

```
git clone https://github.com/StegTechHub/tooling-02.git
```

## 2. On the terminal, export the location of the SQL file

You can find the tooling_db_schema.sql in the html folder of cloned repo.

```
 export tooling_db_schema=tooling-02/html/tooling_db_schema.sql
```

## 3. Use the SQL script to create the database and prepare the schema. With the docker exec command, we can execute a command in a running container.


```
docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema
```

![image](https://github.com/user-attachments/assets/07fc74d8-b4f5-4bd9-af3c-b6c24070c861)

## 4. Update the db_conn.php file with connection details to the database.

```
sudo vi tooling-02/html/db_conn.php

```

* Create a .env file in tooling/html/.env with connection details to the database.
```
sudo vim .env

MYSQL_IP=mysqlserverhost
MYSQL_USER=username
MYSQL_PASS=client-secrete-password
MYSQL_DBNAME=toolingdb
```

![image](https://github.com/user-attachments/assets/a8815e79-7d93-4c38-a31b-9f2d9eabc170)

Flags used:

- MYSQL_IP: mysql ip address "leave as mysqlserverhost"
- MYSQL_USER: mysql username for user exported as environment variable
- MYSQL_PASS: mysql password for the user exported as environment varaible
- MYSQL_DBNAME: mysql databse name "toolingdb"

## 5. Run the Tooling App.

Containerization of an application starts with creation of a file with a special name - Dockerfile (without any extensions). This can be considered as a 'recipe' or 'instruction' that tells Docker how to pack your application into a container.

So, let us containerize our Tooling application; here is the plan:

- Make sure you have checked out your Tooling repo to your machine with Docker engine.
- First, we need to build the Docker image the tooling app will use. The Tooling repo you cloned above has a Dockerfile for this purpose. Explore it and make sure you understand the code inside it.
- Run docker build command
- Launch the container with docker run
- Try to access your application via port exposed from a container

Let us begin:

Ensure you are inside the folder that has the Dockerfile and build your container:

![image](https://github.com/user-attachments/assets/97003ec9-04a1-4714-8b48-8dcc75581417)

![image](https://github.com/user-attachments/assets/84b8f962-7b09-4e33-b530-f0d6dba4a2ec)


In the above command, we specify a parameter -t, so that the image can be tagged tooling:0.0.1 - Also,  have to notice the . at the end. This is important as that tells Docker to locate the Dockerfile in the current directory we are running the command. Otherwise, we would need to specify the absolute path to the Dockerfile.

## 6. Run the container:



 
