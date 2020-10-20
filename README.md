# r2dbc-demo

### Step 1: create a postgresql cluster with pgpool
#### Create a network
```
docker network create my-network
```
#### Create the initial primary node
```
docker run --detach --name pg-0 \
  --network my-network \
  --env REPMGR_PARTNER_NODES=pg-0,pg-1 \
  --env REPMGR_NODE_NAME=pg-0 \
  --env REPMGR_NODE_NETWORK_NAME=pg-0 \
  --env REPMGR_PRIMARY_HOST=pg-0 \
  --env REPMGR_PASSWORD=repmgrpass \
  --env POSTGRESQL_POSTGRES_PASSWORD=adminpassword \
  --env POSTGRESQL_USERNAME=customuser \
  --env POSTGRESQL_PASSWORD=custompassword \
  --env POSTGRESQL_DATABASE=customdatabase \
  -p 5433:5432 \
  bitnami/postgresql-repmgr:latest
```
#### Create a standby node
```
docker run --detach --name pg-1 \
  --network my-network \
  --env REPMGR_PARTNER_NODES=pg-0,pg-1 \
  --env REPMGR_NODE_NAME=pg-1 \
  --env REPMGR_NODE_NETWORK_NAME=pg-1 \
  --env REPMGR_PRIMARY_HOST=pg-0 \
  --env REPMGR_PASSWORD=repmgrpass \
  --env REPMGR_PASSWORD=repmgrpass \
  --env POSTGRESQL_POSTGRES_PASSWORD=adminpassword \
  --env POSTGRESQL_USERNAME=customuser \
  --env POSTGRESQL_PASSWORD=custompassword \
  --env POSTGRESQL_DATABASE=customdatabase \
  bitnami/postgresql-repmgr:latest
```
#### Create the pgpool instance
```
docker run --detach --rm --name pgpool \
  --network my-network \
  --env PGPOOL_BACKEND_NODES=0:pg-0:5432,1:pg-1:5432 \
  --env PGPOOL_SR_CHECK_USER=postgres \
  --env PGPOOL_SR_CHECK_PASSWORD=adminpassword \
  --env PGPOOL_ENABLE_LDAP=no \
  --env PGPOOL_USERNAME=customuser \
  --env PGPOOL_PASSWORD=custompassword \
  -p 5432:5432 \
  bitnami/pgpool:latest
```

### Step 2 Modify application.properties
modify the spring.r2dbc.url to your url to connect to pgpool

### Step 3 Run Test
run test, like com.example.demo.CustomerRepositoryIntegrationTests.executesFindAll.
queries just hang forever, if you modify url to connect to pg-0, the test will passed.
if use jdbc to connect to pgpool, there will be no problems
