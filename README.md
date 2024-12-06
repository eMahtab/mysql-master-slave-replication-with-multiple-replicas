# MySQL Master Slave Replication with multiple replicas

This is an extension of https://github.com/eMahtab/mysql-master-slave-replication , where we had setup Master Slave replication from a single mysql master to a single mysql slave/replica.

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

## Step 2 : Create Replication user on Master with REPLICATION SLAVE privilege
Next we need to create a replication user on Master and grant that user `REPLICATION SLAVE` privilege.
To do this, we execute bash against master **`docker exec -it mysql-master bash`** and connect to mysql **`mysql -uroot -ptoor`** running on master, then execute below mysql commands.
```sql
CREATE USER 'replicator'@'%' IDENTIFIED BY 'rotacilper';
GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'%';
FLUSH PRIVILEGES;
```
Here we create a replication user called `replicator` with password `rotacilper` and grant this user **`REPLICATION SLAVE`** privilege, and finally flush privileges.

!["Create Replication user on Master"](create-replication-user.png?raw=true)

## Step 3 : Execute SHOW MASTER STATUS on MySQL Master 
Get the master status, execute the command **`SHOW MASTER STATUS;`** on Mysql Master to find the Binlog file and position.

!["Get Master status"](show-master-status.png?raw=true)

## Step 4 : Connect to all five MySQL slaves one by one, and execute CHANGE MASTER TO
Next we need to execute **`CHANGE MASTER TO`** command on all five MySQL slave one by one. Connect to MySQL slave To do this, we execute bash against mysql slave (**`docker exec -it mysql-slave-1 bash`** and connect to mysql **`mysql -uroot -ptoor`**) and then execute below command, update MASTER_LOG_FILE and MASTER_LOG_POS values which you got from executing SHOW MASTER STATUS command on MySQL master.
```sql
CHANGE MASTER TO
  MASTER_HOST='mysql_master',
  MASTER_PORT=3306,
  MASTER_USER='replicator',
  MASTER_PASSWORD='rotacilper',
  MASTER_LOG_FILE='bin.000003',
  MASTER_LOG_POS=868,
  GET_MASTER_PUBLIC_KEY=1;
```
!["Execute Change Master to command on MySQL Slave"](change-master-to.png?raw=true)

## Step 5 : Execute START SLAVE on MySQL slave
Execute the command **`START SLAVE;`** on MySQL slave to start the replication, after executing **`START SLAVE;`** you can optionally run **`SHOW REPLICA STATUS;`** to get the status of replica.
One of the most important parameter is **`Seconds_Behind_Source`** which tells how much behind, replica is from master, ideally **`Seconds_Behind_Source`** should always be 0, which means replica is up to date with master.

!["Start slave for replication"](start-five-replica.png?raw=true)

## Step 6 : Pause some of the containers running mysql replica (we pause the containers running mysql-slave-3, mysql-slave-4 and mysql-slave-5)

!["Pause containers running mysql-slave-3, mysql-slave-4, mysql-slave-5"](pause-containers.png?raw=true)

## Step 7 : Execute a procedure on MySQL master which inserts 10,000 records in the users table

Execute the below commands one by one on the MySQL master

`CREATE TABLE users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255) NOT NULL);`

```sql
DELIMITER $$

CREATE PROCEDURE insert_users()
BEGIN
    DECLARE i INT DEFAULT 1;
    WHILE i <= 10000 DO
        INSERT INTO users (name) VALUES (CONCAT('User_', i));
        SET i = i + 1;
    END WHILE;
END$$

DELIMITER ;
```

**`CALL insert_users();`**

It will insert 10,000 users to `users` table on master, and the data will be replicated to replicas asynchronously, since slave-3, slave-4 and slave-5 are disconnected, the data can't be replicated to disconnected slaves. **So slave-3, slave-4 and slave-5 will lag behind and won't have the records initially** , but whenever the disconnected slaves gets connected to master, the replication will be started and eventually the replicas will catch up with master after some time.  

## Step 8 : First start the Paused docker containers and then check the replica lag status

mysql-slave-3, mysql-slave-4 and mysql-slave-5 these three replicas are lagging behind master **(check the parameter `Seconds_Behind_Master` in the SHOW SLAVE STATUS; command output)** because they were disconnected from master for some time.

!["Check Replica lag Status"](replica-lag-status.png?raw=true)

## Step 9 : Replication progressing on previously disconnected slaves

!["Replication progressing on slaves"](slaves-replicating.png?raw=true)

## Step 10 : Replicas catching up with master

You would see over time all replicas will catch up with master and none of the replicas will be behind master.

!["Replicas catching up with master"](replicas-catching-up-with-master.png?raw=true)


## References :

1. https://github.com/eMahtab/mysql-master-slave-replication

