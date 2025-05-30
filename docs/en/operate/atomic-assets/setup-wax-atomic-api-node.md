---
title: How to Set Up a WAX Atomic API Node
order: 4
---

The AtomicAssets NFT standard orginally developed by Pink.gg has become synonymous with the WAX Protocol Network and the widely utilised  [WAX Atomic Hub](https://wax.atomichub.io/).

The AtomicAssets NFT standard stands apart from it’s alternatives by not requiring any RAM from users and offering native two sided trade offers as well as backing NFTs with tokens.

Of course no awesome NFT framework would be complete without the ability for users and developers to access real-time data in a simple and effective way through a scalable API. This is where Pink.gg developed the [eosio-contract-api](https://github.com/pinknetworkx/eosio-contract-api) also known as the  **Atomic API**.

This guide will walk through the process of building and running a WAX Atomic API Node enabling a Guild or Developer to offer a very useful API service for the ecosystem… or maybe just for you.

_This article has been updated to reflect the current Atomic API Node deployment in March 2025._

# How to Set Up a WAX Atomic API Node

Public Atomic API nodes are extremely utilised on the WAX Mainnet and are used by users and DApps to query state and history of atomicassets and the atomicmarket.

A solid mainnet node may be run in docker if you wish to containerise the instance or via PM2 if you prefer to run it at the system level.

This guide goes through the process of building the application dependencies and running the node with PM2.

# Mainnet Requirements

**Hardware**

-   8 Core CPU / 5_Ghz+ recommended_
-   2TB+ Disk /  _Enterprise Grade SSD or NVMe_

Currently (March 2025) a full Atomic API Node postgres DB is **1.1TB**

-   64GB+ RAM

**Operating System**

-   Ubuntu 22.04  **_(Recommended)_**

**SHIP Node**

-   Access to an up to date v5.0.3wax01 WAX State-History node
-   SHIP node on the same LAN  **_(Highly Recommended)_**

**Network**

-   Modern Internet Broadband / Fibre Connection (100Mb/s synchronous and above)
-   1Gb/s+ LAN

# Build the Software

At time of writing the current version of the  [eosio-contract-api](https://github.com/pinknetworkx/eosio-contract-api)  is  `v1.3.24`

Application Dependencies **_(Some our our recommendations)_**

-   NodeJS >= 22.14.0
-   PM2 >= 5.4.3
-   PostgreSQL >= 17.0
-   Redis >= 7.4.2
-   Yarn >= 1.22.22

## Application Installation and Configuration Process

**NodeJS**

```
#Download and import the Nodesource GPG key#
> sudo apt update

> sudo apt install -y ca-certificates curl gnupg

> sudo mkdir -p /etc/apt/keyrings

> curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg

#Create .deb repository#
> NODE_MAJOR=22

> echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list

#Install Node.js#
> sudo apt update 

> sudo apt-get install -y nodejs

**Check Version**
> node -v
```

**PM2**

```
> sudo apt update

#Install PM2#
> sudo npm install pm2@latest -g

**Check Version**
> pm2 -v
```

**Postgres**

Install Postgres as below:

```
> sudo apt-get install wget ca-certificates

> wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

> sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

> sudo apt update

> sudo apt install postgresql postgresql-contrib

#Check Version#  
> apt show postgresql
```

After installation a new database needs to be created and permissions assigned if necessary, this example will use the default postgres user.

```
> sudo su - postgres

> psql

#Assign a password to the postgres account#  
postgres=# \password  
Enter new password:  
Enter it again:

#Create the database#  
postgres=# \l (lists databases)
postgres=# CREATE DATABASE "api-wax-mainnet-atomic-1";
postgres=# \l+ (lists databases - including size)
postgres=# \q (quit)

> exit
```

**Redis**

```
> curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

> echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

> sudo apt update

> sudo apt install redis

> sudo systemctl enable redis-server

#Check Version#  
> redis-server --version

#If there is a unmask error then#  
> sudo systemctl unmask redis-server.service
```

**Yarn**

```
> sudo npm install --global yarn

#Check Version#  
> yarn --version
```

## Atomic API

The latest build can be found on the  [Pink.gg Github](https://github.com/pinknetworkx/eosio-contract-api), ```v1.3.24``` must just be built from the master branch.

**Installation**

```
> git clone https://github.com/pinknetworkx/eosio-contract-api.git

> cd eosio-contract-api

> yarn install
```

**Configuration**

There are three configuration files to be adjusted for your specific deployment

```
connections.config.json  
server.config.json  
readers.config.json
```

Below are basic configurations for this example please tailor to your own deployment:

**_connections.config.json_**

```
> cd  ~/eosio-contract-api/config

> cp example-connections.config.json connections.config.json

> nano connections.config.json
{
  "postgres": {
    "host": "127.0.0.1",
    "port": 5432,
    "user": "postgres",
    "password": "YOUR POSGRESS PASSWORD",
    "database": "api-wax-mainnet-atomic-1"
  },
  "redis": {
    "host": "127.0.0.1",
    "port": 6379
  },
  "chain": {
    "name": "wax-mainnet",
    "chain_id": "1064487b3cd1a897ce03ae5b6a865651747e2e152090f99c1d19d44e01aea5a4",
    "http": "http://10.0.0.65:8888",
    "ship": "ws://10.0.0.65:8080"
  }
}
```

**_server.config.json_**

```
> cd  ~/eosio-contract-api/config

> cp example-server.config.json server.config.json

> nano server.config.json
{
  "provider_name": "EOSphere Guild",
  "provider_url": "https://eosphere.io",
"server_addr": "0.0.0.0",
  "server_name": "wax-atomic-api.eosphere.io",
  "server_port": 9000,
"cache_life": 5,
  "trust_proxy": true,
"rate_limit": {
    "interval": 60,
    "requests": 240,
    "bill_execution_time": true
  },
"ip_whitelist": [],
  "slow_query_threshold": 7500,
"max_query_time_ms": 10000,
  "max_db_connections": 50,
"namespaces": [
    {
      "name": "atomicassets",
      "path": "/atomicassets",
      "args": {
        "atomicassets_account": "atomicassets",
        "connected_reader": "atomic-1",
        "socket_features": {
          "asset_update": false
        }
      }
    },
    {
      "name": "atomicmarket",
      "path": "/atomicmarket",
      "args": {
        "atomicmarket_account": "atomicmarket",
        "connected_reader": "atomic-1",
        "socket_features": {
          "asset_update": false
        },
       "api_features": {
          "disable_v1_sales": true
        }
      }
    },
    {
      "name": "atomictools",
      "path": "/atomictools",
      "args": {
        "atomictools_account": "atomictoolsx",
        "atomicassets_account": "atomicassets",
        "connected_reader": "atomic-1"
      }
    }
  ]
}
```

**_NB:_** _Pay special attention to the rate limit : requests this policy is based on the number request per IP address. If your Atomic API node is behind a reverse proxy the policy may detect all requests from a single IP and start blocking before you anticipate._

**_readers.config.json_**

Advice from the Pink team was that indexing only needs to start from block 64000000

```
> cd  ~/eosio-contract-api/config

> cp example-readers.config.json readers.config.json

> nano readers.config.json
[
  {
    "name": "atomic-1",
"server_addr": "0.0.0.0",
    "server_port": 9000,
"start_block": 64000000,
    "stop_block": 0,
    "irreversible_only": false,
"ship_prefetch_blocks": 50,
    "ship_min_block_confirmation": 30,
    "ship_ds_queue_size": 20,
"db_group_blocks": 10,
"ds_ship_threads": 4,
"modules": [],
"contracts": [
      {
        "handler": "atomicassets",
        "args": {
          "atomicassets_account": "atomicassets",
          "store_transfers": true,
          "store_logs": true
        }
      },
      {
        "handler": "delphioracle",
        "args": {
          "delphioracle_account": "delphioracle"
        }
      },
      {
        "handler": "atomicmarket",
        "args": {
          "atomicmarket_account": "atomicmarket",
          "store_logs": true
        }
      },
      {
        "handler": "atomictools",
        "args": {
          "atomictools_account": "atomictoolsx",
          "store_logs": true
        }
      }
    ]
  }
]
```

## Start/Stop the Atomic API Service

In this example the Atomic API Service will start indexing from block 64000000 and will build a fresh postgres database to the current headblock.

This indexing action will take a significant amount of time for the WAX Mainnet (Months), it is advisable to rather restore the postgres db from a [Trusted Guild Backup](https://snapshots.eosphere.io) as described in [Optimise & Restore a WAX Atomic API Node](https://docs.wax.io/operate/atomic-assets/optimise-restore-wax-atomic-api-node).

This will take sometime and it is advisable to ensure indexing has caught up before you start offering the API publicly.

```
> cd ~/eosio-contract-api

#Start Postgres Indexing Filler#  
> pm2 start ecosystems.config.json --only eosio-contract-api-filler

#Start the API Server#  
> pm2 start ecosystems.config.json --only eosio-contract-api-server

#Monitor Filler and Server#  
> pm2 log
```

**_NB:_** _Stopping the filler service can occasionally be problematic where restarting too soon can cause two instances of the filler to run causing a DB corruption. To get around this ensure that the filler has in fact stopped indexing to postgres by monitoring the active services with top or htop before you restart._

```
#Stop Postgres Indexing Filler#  
> pm2 stop eosio-contract-api-filler

#Stop the API Server#  
> pm2 stop eosio-contract-api-server

#Stop All#  
> pm2 stop all

#Check using top or htop#
```

Your node will be accessible via  **http port 9000**  which when offering the service publicly will be best served from behind a SSL offloading loadbalancer such as HAProxy or nginx.

Query your node’s status as below:

```
> curl http://x.x.x.x:9000/health
> curl http://x.x.x.x:9000/atomicassets/v1/config
> curl http://x.x.x.x:9000/atomicmarket/v1/config
> curl https://x.x.x.x:9000/atomictools/v1/config
```

---

These **WAX Developer Technical Guides** are created using source material from the [EOSphere WAX Technical How To Series](https://medium.com/eosphere/wax-technical-how-to/home)

Be sure to ask any questions in the  [EOSphere Telegram](https://t.me/eosphere_io)
