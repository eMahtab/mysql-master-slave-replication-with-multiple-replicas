# MySQL Master Slave Replication with multiple replicas

This is just an extension of https://github.com/eMahtab/mysql-master-slave-replication , where we had setup Master Slave replication from a single mysql master to a single mysql slave/replica.

In this example we will setup one mysql master and five mysql replicas. So it will be exactly similar to https://github.com/eMahtab/mysql-master-slave-replication, we will run exact same commands but now for five replicas. We will disconnect some of the replicas manually (by pausing the docker containers on which mysql replica is running), this will cause replicas to lag behind master, and we will see how replica catches up with master in very short time (the time it takes for a replica to catch up with master depends on how many operations have been performed on master since replica got disconnected, if the number of operations are large and replica was disconnected for long time, then it will take more time for replica to catch up with master).


## Step 1 : Create the Docker compose file and execute docker compose up, to start a single MySQL master and five MySQL replicas

Below docker-compose-more-replica.yml declares six services, named as `mysql_master, mysql_slave_1, mysql_slave_2, mysql_slave_3, mysql_slave_4, mysql_slave_5` (in docker compose file actual containers are named as `mysql-master, mysql-slave-1, mysql-slave-2, mysql-slave-3, mysql-slave-4, mysql-slave-5`). We are using **mysql:8.0** as the docker image, and declare root user password as toor and create a test database. **Make sure docker engine is running on your host machine before running the docker compose up command.**

**Execute the command `docker compose -f docker-compose-more-replica.yml up`**

```yml
---
version: "2"
services:
  mysql_master:
    image: mysql:8.0
    container_name: mysql-master
    volumes:
      - mysql-master-volume:/tmp
    command:
      [
        "mysqld",
        "--datadir=/tmp/master/data",
        "--log-bin=bin.log",
        "--server-id=1"
      ]
    environment:
      &mysql-default-environment
      MYSQL_ROOT_PASSWORD: toor
      MYSQL_DATABASE: test
      MYSQL_USER: test_user
      MYSQL_PASSWORD: insecure
    ports:
      - "3308:3306"

  mysql_slave_1:
    image: mysql:8.0
    container_name: mysql-slave-1
    volumes:
      - mysql-replica1-volume:/tmp
    command:
      [
        "mysqld",
        "--datadir=/tmp/slave/data",
        "--log-bin=bin.log",
        "--server-id=2"
      ]
    environment: *mysql-default-environment
    ports:
      - "3309:3306"

  mysql_slave_2:
    image: mysql:8.0
    container_name: mysql-slave-2
    volumes:
      - mysql-replica2-volume:/tmp
    command:
      [
        "mysqld",
        "--datadir=/tmp/slave/data",
        "--log-bin=bin.log",
        "--server-id=3"
      ]
    environment: *mysql-default-environment
    ports:
      - "3310:3306"

  mysql_slave_3:
    image: mysql:8.0
    container_name: mysql-slave-3
    volumes:
      - mysql-replica3-volume:/tmp
    command:
      [
        "mysqld",
        "--datadir=/tmp/slave/data",
        "--log-bin=bin.log",
        "--server-id=4"
      ]
    environment: *mysql-default-environment
    ports:
      - "3311:3306"

  mysql_slave_4:
    image: mysql:8.0
    container_name: mysql-slave-4
    volumes:
      - mysql-replica4-volume:/tmp
    command:
      [
        "mysqld",
        "--datadir=/tmp/slave/data",
        "--log-bin=bin.log",
        "--server-id=5"
      ]
    environment: *mysql-default-environment
    ports:
      - "3312:3306"

  mysql_slave_5:
    image: mysql:8.0
    container_name: mysql-slave-5
    volumes:
      - mysql-replica5-volume:/tmp
    command:
      [
        "mysqld",
        "--datadir=/tmp/slave/data",
        "--log-bin=bin.log",
        "--server-id=6"
      ]
    environment: *mysql-default-environment
    ports:
      - "3313:3306"                

volumes:
  mysql-master-volume:
  mysql-replica1-volume:
  mysql-replica2-volume:
  mysql-replica3-volume:
  mysql-replica4-volume:
  mysql-replica5-volume:
```

!["starting single master and five replicas"](one-master-five-replicas.png?raw=true)
