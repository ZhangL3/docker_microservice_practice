# docker_microservice_practice
 
## mysql master slave copy

Slave machine backups the data from master machine

## redis cluster

huge amount data to buffer

- Modulo Hashing
	- hash(key)/3 -> store the data to machine 0 - 2
	- advantages
		- sample
	- disadvantages
		- When one of the machines crashed, all data must be redistributed
		- expandability
			- Scale cluster
- Consistent hashing
	- target: reduce the switching of hash mapping by scale the cluster
	- steps
		- buid a cycle of hash
			- cycle long: [0, 2 ^ 32 - 1]
		- mapping ip as hash value
			- hashValue % (2 ^ 32)
		- principle of landing the key
			- clockwise go along the cycle, the first hit Server is the server tu land
	- advantages
		- fault-tolerance
			- A, B, C, D are in a cycle. If c is crashed, only the data between B and C are invlved
		- expandability
	- disadvantages
		- uneven
			- If there is only few servers, the data store can be uneven
- Hash Slots
	- Slots ist a layer between data and redis servers.
	- Max 16384 slots noted in number [0, 2 ^ 14 - 1], max 1000 node
	- Every redis server have same amount slots, and it is responsible to store the data withing the slots
	
