# docker_microservice_practice
 
## mysql master slave copy

Slave machine backups the data from master machine

## redis cluster

huge amount of data to buffer

- Hashing
	- hash(key)/3 -> store the data to machine 0 - 2
	- advantages
		- sample
	- disadvantages
		- When one of the machines crashed, all data must be redistributed
		- Scale cluster
- Consistent hashing
	- target: reduce the switching of hash mapping by scale the cluster
	- steps
		- buid a cycle of hash
			- hashValue % (2 ^ 32)
		- mapping ip node
		- principle of landing the key
- Hash Slots
