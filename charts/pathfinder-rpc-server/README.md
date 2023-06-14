### Pathfinder rpc server chart

This chart includes the component in the table below in order to  isolate the pathfinder rpc server installation.

For a detailed description please refer to [land-local](https://github.com/CirclesUBI/land-local) 
 Repo                                                                     | Service               | Description                                                                              |
|--------------------------------------------------------------------------|-----------------------|------------------------------------------------------------------------------------------|
| [blockchain-indexer](https://github.com/CirclesUBI/blockchain-indexer)   | blockchain-indexer    | Indexes circles related on-chain events and writes them to a postgres db                 |
| [pathfinder2](https://github.com/CirclesUBI/pathfinder2)                 | pathfinder2           | Finds transitive payment paths between untrusted users                                   | 
| [pathfinder2-updater](https://github.com/CirclesUBI/pathfinder2-updater) | pathfinder2-updater   | Updates the 'pathfinder2' with data from the 'blockchain-indexer'                        | 
| [pathfinder-proxy](https://github.com/CirclesUBI/pathfinder-proxy)       | pathfinder-proxy      | Maintains statistics, load balances and filters requests to the 'pathfinder2'            |


### Currents caveats and usage 

#### Caveats
- The indexer db is empty and without proper schemas and there is not an indexer-db-init. It only exists for local development with ganache. 
- To syncronise the indexer fully at the start it requires a lot resources and its own rpc node. There is still the option to install your own rpc node if you want to go down this road and avoid the following workaround. However, for quicker sync and  resources saving  we recommend the workaround. 

#### Workaround 

When the chart is installed is necessary to restore a db dump. Currently there is one [here](https://rpc.helsinki.circlesubi.id/pathfinder-db/bak_indexer_db_20230529.dump)

To restore the database please execute this command

```
pg_restore -h <HOST_MACHINE> -p 30032 -U postgres -v  -d  index < ./bak_indexer_db_20230529.dump
```

This command will take some time, and you will see some error in the postgres pod, you can safely assume everything is fine when you see these logs in the blockchain indexer pod:

```
Round 5: Round 5 started at 06/14/2023 13:48:22.
Round 5:  Importing from staging tables ..
Round 5: Finding the last persisted block ..
Round 5: Last persisted block: 28202229
Round 5: Finding the latest blockchain block ..
Using the http connection for the next round.
Round 5: Latest blockchain block: 28450652
Round 5: Found 248423 blocks to catch up. Using the 'BulkSource'.
Round 5:  Writing batch to staging tables ..
Round 5:  Importing from staging tables ..
```

At this point you can cancel the pg_restore command and the indexer will continue indexing 🌻

It could be that the indexer throws errors such as: 

```
 Exception while reading from stream ....

 ```

 Do not worry to much since eventually it will correct itself and you will see the logs from above. 


 #### Gnosis Chain and Chiado Tesnet chain 