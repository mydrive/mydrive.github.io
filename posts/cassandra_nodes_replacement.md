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
2. The new node will be given **the exact same** configuration as the replaced one. Therefore, the replacement node will be responsible for the same tokens as the replaced one, and will also have the same Host-ID, so, when it joins the ring, the other nodes won't even notice the difference!

All our infrastructure is in AWS, therefore, we used EBS volumes to backup and restore cassandra data. You may use a different data transfer method which suits you better in your infrastructure.

### Steps

1. Create the external volume you're going to use to transfer the data from the old node to the new one.
2. Setup the new node, paying special attention to the following configuration parameters:
  1. `listen_address`
  2. `rpc_address`
  3. `seeds`
3. After mounting the external volume created in step 1 into the old node, `rsync` the whole cassandra data directory to it. `rsync -av --progress --delete /var/lib/cassandra/data /mnt/backup/` (Assuming external volume is mounted on `/mnt/backup` and `/var/lib/cassandra/data` is Cassandra's data directory on the old machine).
4. Once rsync has finished, unmount and disconnect the external volume from the old node and connect and mount it into the new one. Now rsync the backed up cassandra data directory into the new Cassandra installation `rsync -av --progress --delete /mnt/backup /var/lib/cassandra/data`
5. Mount the external volume into the old node again.
6. Drain the old node: `nodetool drain`
7. Stop Cassandra in the old node: `sudo service cassandra stop` (NOTE: we use chef in our infrastructure so make sure that the node it is using the right cookbook, which makes cassandra service stopped, or stop chef service in that node. Otherwise, cassandra will be started by chef and it will try to join the cluster)
8. Do a final rsync. This one is to catch any last changes.
  1. `rsync -av --progress --delete /var/lib/cassandra/data /mnt/backup`
  2. Unmount and disconnect volume from the old node
  3. Connect and mount it into the new one
  4. `rsync -av --progress --delete /mnt/backup /var/lib/cassandra/data`
  5. Ensure cassandra data folder is owned by the cassandra user (rsync copies the owner's UID along with the data and that UID may not be Cassandra's user in the new machine).
9. Start the new node. `sudo service cassandra start`
10. Check that everything is working properly:
  1. In the replacement's logs you should see a message like this: `WARN <time> Not updating host ID <host ID> for /<replaced node IP address> because it's mine` Indicating that the new node is replacing the old one.
  2. In the replacement's logs you should also see one message like the following per token: `INFO <time> Nodes /<old IP address> and /<new IP address> have the same token <a token>.  Ignoring /<old IP address>` Indicating that the new node is becoming primary owner of the replaced's tokens.
  3. In the other nodes' logs you should see a message like: `Host ID collision for <Host ID> between /<replaced IP address> and /<replacement IP address>; /<replacement IP address> is the new owner` Indicating that the other nodes acknowledge the change
  4. `nodetool [status]` should show the new node's IP owning the replaced Host ID and the old one shouldn't appear anymore.
  5. Everything should look normal.
11. Update other nodes' `seeds` list if the replaced node was a seed one.
13. You can now safely destroy you old machine.

And voilà! By following these steps carefully you will be able to replace nodes and have them running quickly, avoiding any tokens movement or streaming.
