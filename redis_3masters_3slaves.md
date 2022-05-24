```sh
docker run -d --name redis-node-1 --net host --privileged=true -v /data/redis/share/redis-node-1:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6381

docker run -d --name redis-node-2 --net host --privileged=true -v /data/redis/share/redis-node-2:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6382

docker run -d --name redis-node-3 --net host --privileged=true -v /data/redis/share/redis-node-3:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6383

docker run -d --name redis-node-4 --net host --privileged=true -v /data/redis/share/redis-node-4:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6384

docker run -d --name redis-node-5 --net host --privileged=true -v /data/redis/share/redis-node-5:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6385

docker run -d --name redis-node-6 --net host --privileged=true -v /data/redis/share/redis-node-6:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6386
```

```sh
# in node1
redis-cli --cluster create 192.148.124.129:6381 192.148.124.129:6382 192.148.124.129:6383 192.148.124.129:6384 192.148.124.129:6385 192.148.124.129:6386 --cluster-replicas 1
```

=>

```
>>> Performing Cluster Check (using node 192.148.124.129:6381)
M: 45e7e8741e2bd535d1c09b7f5dcbc92522ab03b5 192.148.124.129:6381
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 8e6c48c175420fe25d8f64e95ce4f1a014135cf3 192.148.124.129:6385
   slots: (0 slots) slave
   replicates 7a83cce7b8334057b1d5065ff2471418a73735c1
S: 71b834857742817527ade645e0e298c1af91dd78 192.148.124.129:6386
   slots: (0 slots) slave
   replicates 45e7e8741e2bd535d1c09b7f5dcbc92522ab03b5
M: 7a83cce7b8334057b1d5065ff2471418a73735c1 192.148.124.129:6383
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: 539b6de7180aab6e6e246b557cb186eb2c4762fc 192.148.124.129:6382
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: d398457f7065f7333cc55945695afefe74963db5 192.148.124.129:6384
   slots: (0 slots) slave
   replicates 539b6de7180aab6e6e246b557cb186eb2c4762fc
[OK] All nodes agree about slots configuration.
>>> Check for open slots...                                                                  
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

```sh
# in node1
redis-cli -p 6381
cluster info
cluster nodes
```

```
127.0.0.1:6381> cluster nodes
8e6c48c175420fe25d8f64e95ce4f1a014135cf3 192.148.124.129:6385@16385 slave 7a83cce7b8334057b1d5065ff2471418a73735c1 0 1653203506000 3 connected
45e7e8741e2bd535d1c09b7f5dcbc92522ab03b5 192.148.124.129:6381@16381 myself,master - 0 1653203505000 1 connected 0-5460
71b834857742817527ade645e0e298c1af91dd78 192.148.124.129:6386@16386 slave 45e7e8741e2bd535d1c09b7f5dcbc92522ab03b5 0 1653203504000 1 connected
7a83cce7b8334057b1d5065ff2471418a73735c1 192.148.124.129:6383@16383 master - 0 1653203505925 3 connected 10923-16383
539b6de7180aab6e6e246b557cb186eb2c4762fc 192.148.124.129:6382@16382 master - 0 1653203504917 2 connected 5461-10922
d398457f7065f7333cc55945695afefe74963db5 192.148.124.129:6384@16384 slave 539b6de7180aab6e6e246b557cb186eb2c4762fc 0 1653203506933 2 connected

master:slaver
node1:node6
node2:node4
node3:node5
```

```
127.0.0.1:6381> keys *
(empty array)
127.0.0.1:6381> set k1 v1
(error) MOVED 12706 192.148.124.129:6383
127.0.0.1:6381>
```

```
# for cluster it does not work single machine to connect
redis-cli -p 6381
127.0.0.1:6381> set k1 v1
(error) MOVED 12706 192.148.124.129:6383
127.0.0.1:6381> set k2 v2
(error) MOVED 449 192.148.124.129:6386
```

```
root@lezhang-Lubuntu:/data# redis-cli -p 6381 -c
127.0.0.1:6381> FLUSHALL
(error) READONLY You can't write against a read only replica.
127.0.0.1:6381> set k1 v1
-> Redirected to slot [12706] located at 192.148.124.129:6383
OK
```
```
root@lezhang-Lubuntu:/data# redis-cli --cluster check 192.148.124.129:6381
192.148.124.129:6386 (71b83485...) -> 2 keys | 5461 slots | 1 slaves.
192.148.124.129:6383 (7a83cce7...) -> 1 keys | 5461 slots | 1 slaves.
192.148.124.129:6384 (d398457f...) -> 1 keys | 5462 slots | 1 slaves.
[OK] 4 keys in 3 masters.
0.00 keys per slot on average.                                                               
>>> Performing Cluster Check (using node 192.148.124.129:6381)
S: 45e7e8741e2bd535d1c09b7f5dcbc92522ab03b5 192.148.124.129:6381
   slots: (0 slots) slave
   replicates 71b834857742817527ade645e0e298c1af91dd78
S: 8e6c48c175420fe25d8f64e95ce4f1a014135cf3 192.148.124.129:6385
   slots: (0 slots) slave
   replicates 7a83cce7b8334057b1d5065ff2471418a73735c1
M: 71b834857742817527ade645e0e298c1af91dd78 192.148.124.129:6386
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 7a83cce7b8334057b1d5065ff2471418a73735c1 192.148.124.129:6383
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 539b6de7180aab6e6e246b557cb186eb2c4762fc 192.148.124.129:6382
   slots: (0 slots) slave
   replicates d398457f7065f7333cc55945695afefe74963db5
M: d398457f7065f7333cc55945695afefe74963db5 192.148.124.129:6384
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...                                                                  
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

switch machine (one master machine crashed)

```
127.0.0.1:6383> cluster nodes
539b6de7180aab6e6e246b557cb186eb2c4762fc 192.148.124.129:6382@16382 slave d398457f7065f7333cc55945695afefe74963db5 0 1653279493824 8 connected
71b834857742817527ade645e0e298c1af91dd78 192.148.124.129:6386@16386 master - 0 1653279491000 9 connected 0-5460
7a83cce7b8334057b1d5065ff2471418a73735c1 192.148.124.129:6383@16383 myself,master - 0 1653279492000 12 connected 10923-16383
45e7e8741e2bd535d1c09b7f5dcbc92522ab03b5 192.148.124.129:6381@16381 slave 71b834857742817527ade645e0e298c1af91dd78 0 1653279491000 9 connected
d398457f7065f7333cc55945695afefe74963db5 192.148.124.129:6384@16384 master - 0 1653279492000 8 connected 5461-10922
8e6c48c175420fe25d8f64e95ce4f1a014135cf3 192.148.124.129:6385@16385 slave 7a83cce7b8334057b1d5065ff2471418a73735c1 0 1653279492802 12 connected
```

```sh
docker stop reids-node-4
```

```
127.0.0.1:6384> cluster nodes
45e7e8741e2bd535d1c09b7f5dcbc92522ab03b5 192.148.124.129:6381@16381 slave 71b834857742817527ade645e0e298c1af91dd78 0 1653279591463 9 connected
71b834857742817527ade645e0e298c1af91dd78 192.148.124.129:6386@16386 master - 0 1653279589437 9 connected 0-5460
8e6c48c175420fe25d8f64e95ce4f1a014135cf3 192.148.124.129:6385@16385 master - 0 1653279590451 13 connected 10923-16383
d398457f7065f7333cc55945695afefe74963db5 192.148.124.129:6384@16384 myself,master - 0 1653279588000 8 connected 5461-10922
7a83cce7b8334057b1d5065ff2471418a73735c1 192.148.124.129:6383@16383 master,fail - 1653279537847 1653279534782 12 disconnected
539b6de7180aab6e6e246b557cb186eb2c4762fc 192.148.124.129:6382@16382 slave d398457f7065f7333cc55945695afefe74963db5 0 1653279587418 8 connected
```

recover redis-node-3

```
lezhang@lezhang-Lubuntu:/mydata$ docker start redis-node-3
redis-node-3
lezhang@lezhang-Lubuntu:/mydata$ docker exec -it redis-node-3 /bin/bash
root@lezhang-Lubuntu:/data# redis-cli -p 6383 -c
127.0.0.1:6383> cluster nodes
8e6c48c175420fe25d8f64e95ce4f1a014135cf3 192.148.124.129:6385@16385 master - 0 1653280079000 13 connected 10923-16383
539b6de7180aab6e6e246b557cb186eb2c4762fc 192.148.124.129:6382@16382 slave d398457f7065f7333cc55945695afefe74963db5 0 1653280080137 8 connected
71b834857742817527ade645e0e298c1af91dd78 192.148.124.129:6386@16386 master - 0 1653280079129 9 connected 0-5460
d398457f7065f7333cc55945695afefe74963db5 192.148.124.129:6384@16384 master - 0 1653280078000 8 connected 5461-10922
7a83cce7b8334057b1d5065ff2471418a73735c1 192.148.124.129:6383@16383 myself,slave 8e6c48c175420fe25d8f64e95ce4f1a014135cf3 0 1653280077000 13 connected
45e7e8741e2bd535d1c09b7f5dcbc92522ab03b5 192.148.124.129:6381@16381 slave 71b834857742817527ade645e0e298c1af91dd78 0 1653280078121 9 connected
```

recover original master and slave relashenship

```
lezhang@lezhang-Lubuntu:/mydata$ docker stop redis-node-5
redis-node-5
lezhang@lezhang-Lubuntu:/mydata$ docker start redis-node-5
redis-node-5
lezhang@lezhang-Lubuntu:/mydata$ docker exec -it redis-node-5 /bin/bash
root@lezhang-Lubuntu:/data# redis-cli -p 6383 -c
127.0.0.1:6383> cluster nodes
8e6c48c175420fe25d8f64e95ce4f1a014135cf3 192.148.124.129:6385@16385 slave 7a83cce7b8334057b1d5065ff2471418a73735c1 0 1653280370000 14 connected
539b6de7180aab6e6e246b557cb186eb2c4762fc 192.148.124.129:6382@16382 slave d398457f7065f7333cc55945695afefe74963db5 0 1653280370000 8 connected
71b834857742817527ade645e0e298c1af91dd78 192.148.124.129:6386@16386 master - 0 1653280371000 9 connected 0-5460
d398457f7065f7333cc55945695afefe74963db5 192.148.124.129:6384@16384 master - 0 1653280372778 8 connected 5461-10922
7a83cce7b8334057b1d5065ff2471418a73735c1 192.148.124.129:6383@16383 myself,master - 0 1653280369000 14 connected 10923-16383
45e7e8741e2bd535d1c09b7f5dcbc92522ab03b5 192.148.124.129:6381@16381 slave 71b834857742817527ade645e0e298c1af91dd78 0 1653280371772 9 connected
```

add 1 mater and 1 slave to cluster

```sh
docker run -d --name redis-node-7 --net host --privileged=true -v /data/redis/share/redis-node-7:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6387

docker run -d --name redis-node-8 --net host --privileged=true -v /data/redis/share/redis-node-8:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6388
```

add redis-node-7 to cluster, as redis-node-3

```
# in node 7
lezhang@lezhang-Lubuntu:/mydata$ docker exec -it redis-node-7 /bin/bash
root@lezhang-Lubuntu:/data# redis-cli --cluster add-node 192.148.124.129:6387 192.148.124.129:6383
>>> Adding node 192.148.124.129:6387 to cluster 192.148.124.129:6383
>>> Performing Cluster Check (using node 192.148.124.129:6383)
M: 7a83cce7b8334057b1d5065ff2471418a73735c1 192.148.124.129:6383
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 8e6c48c175420fe25d8f64e95ce4f1a014135cf3 192.148.124.129:6385
   slots: (0 slots) slave
   replicates 7a83cce7b8334057b1d5065ff2471418a73735c1
S: 539b6de7180aab6e6e246b557cb186eb2c4762fc 192.148.124.129:6382
   slots: (0 slots) slave
   replicates d398457f7065f7333cc55945695afefe74963db5
M: 71b834857742817527ade645e0e298c1af91dd78 192.148.124.129:6386
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: d398457f7065f7333cc55945695afefe74963db5 192.148.124.129:6384
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 45e7e8741e2bd535d1c09b7f5dcbc92522ab03b5 192.148.124.129:6381
   slots: (0 slots) slave
   replicates 71b834857742817527ade645e0e298c1af91dd78
[OK] All nodes agree about slots configuration.
>>> Check for open slots...                                                                  
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 192.148.124.129:6387 to make it join the cluster.              
[OK] New node added correctly.
```

redistribute slots

```
redis-cli --cluster reshard 192.148.124.129:6383
How many slots do you want to move ? 4096
What is the receiving node ID? CLUSTER_ID 6387
Source node #1: all
...

root@lezhang-Lubuntu:/data# redis-cli --cluster check 192.148.124.129:6387
192.148.124.129:6387 (c524533a...) -> 1 keys | 4096 slots | 0 slaves.
192.148.124.129:6383 (7a83cce7...) -> 1 keys | 4096 slots | 1 slaves.
192.148.124.129:6384 (d398457f...) -> 1 keys | 4096 slots | 1 slaves.
192.148.124.129:6386 (71b83485...) -> 1 keys | 4096 slots | 1 slaves.
[OK] 4 keys in 4 masters.
0.00 keys per slot on average.                                                               
>>> Performing Cluster Check (using node 192.148.124.129:6387)
M: c524533a11f47de7848a4018727d8e9da81f3987 192.148.124.129:6387
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
S: 539b6de7180aab6e6e246b557cb186eb2c4762fc 192.148.124.129:6382
   slots: (0 slots) slave
   replicates d398457f7065f7333cc55945695afefe74963db5
S: 45e7e8741e2bd535d1c09b7f5dcbc92522ab03b5 192.148.124.129:6381
   slots: (0 slots) slave
   replicates 71b834857742817527ade645e0e298c1af91dd78
M: 7a83cce7b8334057b1d5065ff2471418a73735c1 192.148.124.129:6383
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
M: d398457f7065f7333cc55945695afefe74963db5 192.148.124.129:6384
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
M: 71b834857742817527ade645e0e298c1af91dd78 192.148.124.129:6386
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
S: 8e6c48c175420fe25d8f64e95ce4f1a014135cf3 192.148.124.129:6385
   slots: (0 slots) slave
   replicates 7a83cce7b8334057b1d5065ff2471418a73735c1
[OK] All nodes agree about slots configuration.
>>> Check for open slots...                                                                  
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

add slave 6388 to 6387

```
root@lezhang-Lubuntu:/data# redis-cli --cluster add-node 192.148.124.129:6388 192.148.124.129:6387 --cluster-slave --cluster-master-id c524533a11f47de7848a4018727d8e9da81f3987
>>> Adding node 192.148.124.129:6388 to cluster 192.148.124.129:6387
>>> Performing Cluster Check (using node 192.148.124.129:6387)
M: c524533a11f47de7848a4018727d8e9da81f3987 192.148.124.129:6387
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
S: 539b6de7180aab6e6e246b557cb186eb2c4762fc 192.148.124.129:6382
   slots: (0 slots) slave
   replicates d398457f7065f7333cc55945695afefe74963db5
S: 45e7e8741e2bd535d1c09b7f5dcbc92522ab03b5 192.148.124.129:6381
   slots: (0 slots) slave
   replicates 71b834857742817527ade645e0e298c1af91dd78
M: 7a83cce7b8334057b1d5065ff2471418a73735c1 192.148.124.129:6383
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
M: d398457f7065f7333cc55945695afefe74963db5 192.148.124.129:6384
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
M: 71b834857742817527ade645e0e298c1af91dd78 192.148.124.129:6386
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
S: 8e6c48c175420fe25d8f64e95ce4f1a014135cf3 192.148.124.129:6385
   slots: (0 slots) slave
   replicates 7a83cce7b8334057b1d5065ff2471418a73735c1
[OK] All nodes agree about slots configuration.
>>> Check for open slots...                                                                  
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 192.148.124.129:6388 to make it join the cluster.              
Waiting for the cluster to join

>>> Configure node as replica of 192.148.124.129:6387.
[OK] New node added correctly.


root@lezhang-Lubuntu:/data# redis-cli --cluster check 192.148.124.129:6387
192.148.124.129:6387 (c524533a...) -> 1 keys | 4096 slots | 1 slaves.
192.148.124.129:6383 (7a83cce7...) -> 1 keys | 4096 slots | 1 slaves.
192.148.124.129:6384 (d398457f...) -> 1 keys | 4096 slots | 1 slaves.
192.148.124.129:6386 (71b83485...) -> 1 keys | 4096 slots | 1 slaves.
[OK] 4 keys in 4 masters.
0.00 keys per slot on average.                                                               
>>> Performing Cluster Check (using node 192.148.124.129:6387)
M: c524533a11f47de7848a4018727d8e9da81f3987 192.148.124.129:6387
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
   1 additional replica(s)
S: 539b6de7180aab6e6e246b557cb186eb2c4762fc 192.148.124.129:6382
   slots: (0 slots) slave
   replicates d398457f7065f7333cc55945695afefe74963db5
S: 45e7e8741e2bd535d1c09b7f5dcbc92522ab03b5 192.148.124.129:6381
   slots: (0 slots) slave
   replicates 71b834857742817527ade645e0e298c1af91dd78
M: 7a83cce7b8334057b1d5065ff2471418a73735c1 192.148.124.129:6383
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
M: d398457f7065f7333cc55945695afefe74963db5 192.148.124.129:6384
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
M: 71b834857742817527ade645e0e298c1af91dd78 192.148.124.129:6386
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
S: 8e6c48c175420fe25d8f64e95ce4f1a014135cf3 192.148.124.129:6385
   slots: (0 slots) slave
   replicates 7a83cce7b8334057b1d5065ff2471418a73735c1
S: 3dfd7f1b29b800d79f8ebd689dfa58e2e410e925 192.148.124.129:6388
   slots: (0 slots) slave
   replicates c524533a11f47de7848a4018727d8e9da81f3987
[OK] All nodes agree about slots configuration.
>>> Check for open slots...                                                                  
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

schrink redis cluster

```
# remove slave 6388
root@lezhang-Lubuntu:/data# redis-cli --cluster del-node 192.148.124.129:6388 3dfd7f1b29b800d79f8ebd689dfa58e2e410e925
>>> Removing node 3dfd7f1b29b800d79f8ebd689dfa58e2e410e925 from cluster 192.148.124.129:6388
>>> Sending CLUSTER FORGET messages to the cluster...
>>> Sending CLUSTER RESET SOFT to the deleted node.
```

```
# redistribute all slots from 6387 to 6383
redis-cli --cluster reshard 192.148.124.129:6383
How many slots do you want to move ? 4096
What is the receiving node ID? CLUSTER_ID 6383
Source node #1: CLUSTER_ID 6387
Source node #2: done
...
```

```
# remove master 6087
```
