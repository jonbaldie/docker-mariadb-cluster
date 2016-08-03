# docker-mariadb-cluster
Dockerized MariaDB Galera Cluster

Build for use with Docker __1.12__+

# WORK in Progress!!

## Setup
### Init Swarm Nodes/Cluster

Swarm Master:
		
	docker swarm init
		
Additional Swarm Node(s):

	docker swarm join <MasterNodeIP>:2377

### Create DB network

	docker network create -d overlay mydbnet

### Fire up Bootstrap node
		
	docker service create --name bootstrap \
	--network mydbnet \
	--replicas=1 \
	--env MYSQL_ALLOW_EMPTY_PASSWORD=0 \
	--env MYSQL_ROOT_PASSWORD=rootpass \
	--env DB_BOOTSTRAP_NAME=bootstrap \
	toughiq/mariadb-cluster --wsrep-new-cluster

### Fire up Cluster Members

	docker service create --name dbcluster \
	--network mydbnet \
	--replicas=3 \
	--env DB_SERVICE_NAME=dbcluster \
	--env DB_BOOTSTRAP_NAME=bootstrap \
	toughiq/mariadb-cluster

### Startup MaxScale Proxy

	docker service create --name maxscale \
	--network mydbnet \
	--env DB_SERVICE_NAME=dbcluster \
	--env ENABLE_ROOT_USER=1 \
	--publish 3306:3306 \
	toughiq/maxscale

### Check successful startup of Cluster & MaxScale
Execute the following command. Just use autocompletion to get the `SLOT` and `ID`.

	docker exec -it maxscale.<SLOT>.<ID> maxadmin -pmariadb list servers

The result should report you cluster up and running:

	-------------------+-----------------+-------+-------------+--------------------
	Server             | Address         | Port  | Connections | Status              
	-------------------+-----------------+-------+-------------+--------------------
	10.0.0.9           | 10.0.0.9        |  3306 |           0 | Slave, Synced, Running
	10.0.0.8           | 10.0.0.8        |  3306 |           0 | Slave, Synced, Running
	10.0.0.10          | 10.0.0.10       |  3306 |           0 | Master, Synced, Running
	-------------------+-----------------+-------+-------------+--------------------


### Remove Bootstrap Service

	docker service rm bootstrap
