# MySQL Master Slave Replication with multiple replicas

This is just an extension of https://github.com/eMahtab/mysql-master-slave-replication , where we had setup Master Slave replication from a single mysql master to a single mysql slave/replica.

In this example we will setup one mysql master and five mysql replicas. So it will be exactly similar to https://github.com/eMahtab/mysql-master-slave-replication, we will run exact same commands but now for five replicas. We will disconnect some of the replicas manually (by pausing the docker containers on which mysql replica is running), this will cause replicas to lag behind master, and we will see how replica catches up with master in very short time (the time it takes for a replica to catch up with master depends on how many operations have been performed on master since replica got disconnected, if the number of operations are large and replica was disconnected for long time, then it will take more time for replica to catch up with master).


