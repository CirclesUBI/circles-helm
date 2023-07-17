### Pathfinder rpc server chart

This chart includes the component in the table below in order to  isolate the pathfinder rpc server installation.

For a detailed description please refer to [land-local](https://github.com/CirclesUBI/land-local)
 Repo                                                                     | Service               | Description                                                                              |
|--------------------------------------------------------------------------|-----------------------|------------------------------------------------------------------------------------------|
| [blockchain-indexer](https://github.com/CirclesUBI/blockchain-indexer)   | blockchain-indexer    | Indexes circles related on-chain events and writes them to a postgres db                 |
| [pathfinder2](https://github.com/CirclesUBI/pathfinder2)                 | pathfinder2           | Finds transitive payment paths between untrusted users                                   |
| [pathfinder2-updater](https://github.com/CirclesUBI/pathfinder2-updater) | pathfinder2-updater   | Updates the 'pathfinder2' with data from the 'blockchain-indexer'                        |
| [pathfinder-proxy](https://github.com/CirclesUBI/pathfinder-proxy)       | pathfinder-proxy      | Maintains statistics, load balances and filters requests to the 'pathfinder2'            |


#### Notes
Since the installation of pathfinder and updater are in the same container, this charts needs to explicitely set the ports for both in the service so these can be located by other services on the network, such as the pathfinder-proxy.

For an in detail explanation of the every component and variable please refer to the repos above.

### Pre-requisities

Create the following secrets in your cluster:
    - blockchain-indexer-connection-string: connection string for the indexer to the db
    - pathfinder-updater-connection-string: connection string for the pathfinder to the db
    - indexer-db-secret: `password` for the `doadmin` user and `postgres-password`  for the `postgres` user

Manage your secrets with your preferred tool.


### Currents caveats and usage

#### Caveats
- The indexer database is empty and without proper schemas, indexes and reference since there is no indexer-db-init. It only exists for local development with ganache.
- To synchronise the indexer fully at the start requires a lot of resources and its own RPC node. There is still the option to install your own RPC node, if you want to go down this road and avoid the following workaround. However, for quicker sync and resource savings we recommend the workaround.

#### Workaround

When the chart is installed it is necessary to perform a complete database restore (indexes and references included). Currently there is [a dump from 2023-05-29](https://rpc.helsinki.circlesubi.id/pathfinder-db/bak_indexer_db_20230529.dump) available.

You will need to download [this `0.0.64.sql` file](https://github.com/CirclesUBI/blockchain-indexer/blob/dev/CirclesLand.BlockchainIndexer/DbMigrations/0.0.64.sql) from the `dev` branch. We assume this is the latest schema, but this should be confirmed with the upstream repository, in case of doubt.

To restore the database, please execute the following commands:
```
kubectl -n pathfinder exec -it pod/pathfinder-rpc-server-postgresql-0 -- sh -c 'PGPASSWORD=${POSTGRES_PASSWORD} psql -h localhost -p 5432 -U ${POSTGRES_USER} -v -d ${POSTGRES_DB}' < ./0.0.64.sql
```

```
 kubectl -n pathfinder exec -it pod/pathfinder-rpc-server-postgresql-0 -- sh -c 'PGPASSWORD=${POSTGRES_PASSWORD} pg_restore -h localhost -p 5432 -U ${POSTGRES_USER} -v -d ${POSTGRES_DB}' < indexer.dump
```

This command will take some time  (a few hours) wait until the pg_restore finishes. The last table synced is `transactions_2`.



 #### Gnosis Chain and Chiado Tesnet chain

 The current indexer was written only for gnosis chain, but it can be used for chiado.
