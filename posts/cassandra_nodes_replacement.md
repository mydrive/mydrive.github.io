---
title: Cassandra instantaneous in place node replacement
author: carlos
date: "2016-01-22"
description: A quick process to replace a Cassandra node and have it immediately running.
tags: ["Cassandra", "Operations", "Node replacement"]
categories: ["operations"]
---

At some point everyone using Cassandra faces the situation of having to replace nodes.
Either because the cluster needs to scale and some nodes are too small or because a node has failed or because your virtualisation provider is going to remove it.

Whichever the reason the situation we face is that a new node has to be streamed in, to hold the **exact same data** the old one had. Why do we need to wait for a whole streaming process, with the network and CPU overhead this requires when we could just copy the data into the new node and have it join the ring replacing the old one?

That's what we, at [MyDrive](https://twitter.com/_mydrive) have been doing for a while and we want to share the exact process we follow with the community shall it help someone.

### Summary

They main idea behind this process is to have the replacement node up and running as quick as possible by cutting down the process where it takes longer, streaming data.

The key points of the process are:

1. Data will be copied from the old node to the new one using an external volume instead of transmitting it through the network.
2. The new node will receive also the schema, so that it opens all tables before accepting any query.
3. The new node will be given the tokens by configuration, so the replacement will be responsible for the exact same data of the node being replaced, and as it already holds the data, no bootstrap process is required.

### Steps

1. Create the external volume you're going to use to transfer the data from the old node to the new one.
2. Setup the new node, paying special attention to the following configuration parameters:
  1. `listen_address`
  2. `rpc_address`
  3. `seeds`
  4. `initial_token`: This is one of the key ones. The initial token has to match the exact token(s) the old node is responsible for. You can get them by running `nodetool ring` if single token nodes or `nodetool info -T` if vnodes.
  5. auto_bootstrap: false
3. After mounting the external volume created in step 1 into the old node, `rsync` the whole cassandra data directory to it. `rsync -av --progress --delete /var/lib/cassandra/data /mnt/backup/` (Assuming external volume is mounted on `/mnt/backup` and `/var/lib/cassadnra/data` is Cassandra's data directory on the old machine).
4. Once rsync has finished, unmount and disconnect the external volume from the old node and connect and mount it into the new one. Now rsync the backed up cassandra data directory into the new Cassandra installation `rsync -av --progress --delete /mnt/backup /var/lib/cassandra/data`
5. Mount the external volume into the old node again.
6. Drain the old node: `nodetool drain`
7. Stop Cassandra in the old node: `sudo service cassandra stop`
8. Do a final rsync. This one is to catch any last changes.
  1. `rsync -av --progress --delete /var/lib/cassandra/data /mnt/backup`
  2. Unmount and disconnect volume from the old node
  3. Connect and mount it into the new one
  4. `rsync -av --progress --delete /mnt/backup /var/lib/cassandra/data`
  5. Ensure cassandra data folder is owned by the cassandra user (rsync copies the owner's UID along with the data and that UID may not be Cassandra's user in the new machine).
9. In the new node, delete all system tables except for the schema ones. This will ensure that the new Cassandra node will not have any corrupt or previous configuration assigned.
  1. `sudo cd /var/lib/cassandra/data/system && sudo ls | grep -v schema | xargs -I {} sudo rm -rf {}`
  2. `sudo rm -rf /var/lib/cassandra/data/system_traces`
  3. `sudo rm -rf /var/lib/cassandra/commitlog`
  4. `sudo rm -rf /var/lib/cassandra/saved_caches`
10. Start the new node. `sudo service cassandra start`
11. Check that everything is working properly:
  1. In the other nodes's logs you should see a message like this: `WARN <time> Token <your token> changing ownership from <old node's IP> to <new node's IP>` Indicating that they recognise the new node and that is taking over the other's token.
  2. `nodetool [ring/status]` should show the new node's IP owning the specified token(s) and the old one shouldn't appear anymore.
  3. Everything should look normal.
12. Update new node's `cassandra.yaml` to remove the `auto_bootstrap` config and to leave `initial_token` empty.
13. You can now safely destroy you old machine.

And voilà! By following these steps carefully you will be able to replace nodes and have them running quickly, avoiding any tokens movement or streaming.
