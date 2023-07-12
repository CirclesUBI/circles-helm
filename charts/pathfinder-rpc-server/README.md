### Pathfinder rpc server chart

This chart includes the component in the table below in order to  isolate the pathfinder rpc server installation.

For a detailed description please refer to [land-local](https://github.com/CirclesUBI/land-local)
 Repo                                                                     | Service               | Description                                                                              |
|--------------------------------------------------------------------------|-----------------------|------------------------------------------------------------------------------------------|
| [blockchain-indexer](https://github.com/CirclesUBI/blockchain-indexer)   | blockchain-indexer    | Indexes circles related on-chain events and writes them to a postgres db                 |
| [pathfinder2](https://github.com/CirclesUBI/pathfinder2)                 | pathfinder2           | Finds transitive payment paths between untrusted users                                   |
| [pathfinder2-updater](https://github.com/CirclesUBI/pathfinder2-updater) | pathfinder2-updater   | Updates the 'pathfinder2' with data from the 'blockchain-indexer'                        |
| [pathfinder-proxy](https://github.com/CirclesUBI/pathfinder-proxy)       | pathfinder-proxy      | Maintains statistics, load balances and filters requests to the 'pathfinder2'            |


### Pre-requisities

Create the following secrets in your cluster:
    - blockchain-indexer-connection-string: connection string for the indexer to the db
    - pathfinder-updater-connection-string: connection string for the pathfinder to the db
    - indexer-db-secret: `password` for the `doadmin` user and `postgres-password`  for the `postgres` user

Manage your secrets with your prefered tool.

### Currents caveats and usage

#### Caveats
- The indexer db is empty and without proper schemas, indexes and reference since there is not an indexer-db-init. It only exists for local development with ganache.
- To syncronise the indexer fully at the start it requires a lot resources and its own rpc node. There is still the option to install your own rpc node if you want to go down this road and avoid the following workaround. However, for quicker sync and  resources saving  we recommend the workaround.

#### Workaround

When the chart is installed is necessary to restore DB fully (indexes and references included). Currently there is one [here](https://rpc.helsinki.circlesubi.id/pathfinder-db/bak_indexer_db_20230529.dump)

You will find the `0.0.64.sql` file [here](https://github.com/CirclesUBI/blockchain-indexer/blob/dev/CirclesLand.BlockchainIndexer/DbMigrations/0.0.64.sql) from the `dev` branch. We assume this is latest but this should be confirmed with the repo mantainer in case of any changes.

To restore the database please execute the following commands
```
kubectl -n pathfinder exec -it pod/pathfinder-rpc-server-postgresql-0 -- sh -c 'PGPASSWORD=${POSTGRES_PASSWORD} psql -h localhost -p 5432 -U ${POSTGRES_USER} -v -d ${POSTGRES_DB}' < ./0.0.64.sql
```

```
 kubectl -n pathfinder exec -it pod/pathfinder-rpc-server-postgresql-0 -- sh -c 'PGPASSWORD=${POSTGRES_PASSWORD} pg_restore -h localhost -p 5432 -U ${POSTGRES_USER} -v -d ${POSTGRES_DB}' < indexer.dump
```

This command will take some time  (a few hours) wait until the pg_restore finishes. The last table synced is `transactions_2`.



 #### Gnosis Chain and Chiado Tesnet chain

 The current indexer was written only for gnosis chain, but it can be used for chiado.
