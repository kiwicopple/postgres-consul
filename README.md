# Postgres with Consul

Monitoring 3 Postgres clusters using Postgres.



## Usage:

```bash
# Start docker, force full refresh
docker-compose up --force-recreate --remove-orphans --build

# Tear down
docker-compose down --volumes --remove-orphans
```

Open [localhost:8500](http://localhost:8500) to see Consul UI. You can kill and restart any Postgres server to see the state change.


## What this does

- Docker-compose uses 3 Postgres clusters and one consul server/agent.
- All clusters are monitored by consul using a healthcheck (see: `/consul/consul.d/postgres{1,2,3}.json`)
- Consul docker image is built with `postgres-client` so that it can probe Postgres from the server
- Database is checked every 3s [here](https://github.com/kiwicopple/consul-postgres/blob/cb186d3243b86aa4fdd96286c3220c332459c6cc/consul/consul.d/postgres1.json#L7)
    - This script checks `pg_isready` [here](https://github.com/kiwicopple/consul-postgres/blob/main/consul/consul.d/pg_check.sh)
- If the datbase goes down then it triggers a watch script [here](https://github.com/kiwicopple/consul-postgres/blob/cb186d3243b86aa4fdd96286c3220c332459c6cc/consul/consul.d/postgres1.json#L37)
  - A change in the database state can do anything (like call any endpoint). For this I have just got it inserting the state change into Consul KV


## Production readiness

- Consul should be split into 3 for HA
- Postgres instances should have agents installed so that they can signal when they go offline faster than the 3s interval