# Virtual Apache ZooKeeper cluster

This script creates a virtual 3-node [Apache ZooKeeper](http://zookeeper.apache.org/)
cluster on your local machine using [Vagrant](https://www.vagrantup.com/), [VirtualBox](https://www.virtualbox.org/) and [Ansible](http://www.ansible.com/home).

The Zookeeper service will run in a truly replicated mode over several machines, so you can experiment with host failures, client connections, misc cluster configurations, etc.

Also, if you want to create a virtual cluster for other services built on top of Zookeeper (e.g. Apache Kafka, HBase, Solr, Neo4j), this script might help as a starting point.

## Cluster specs

- System: 3x Ubuntu 12.02 server
- Memory: 256 MB each host
- IP address: 192.168.5.100-102
- Hostnames: `node-[x]` with `x` have values 1, 2 or 3
- Zookeeper version: 3.4.6
- JVM: Oracle Java-7

You can easily customize the cluster parameters in the following files:
- `Vagrantfile` contains parameters for the system, memory, IP addresses and hostnames
- `provision.yml` contains parameters for Zookeeper version, installation directory, mirror and the JVM version.

*Note regarding the cluster size*: It is recommended to run the cluster on an odd number of hosts (3, 5, etc). Zookeeper is designed to survive failure of minority of hosts. Zookeeper running on 3 or 4 hosts can survive failure of 1 host while Zookeper running on 5 hosts can survive failure of 2 hosts.

## Usage

Depending on your hardware and network connection, the script execution might take between 10-20 minutes.

### 1. Install Vagrant, VirtualBox and Ansible on your machine

For Mac, this can be done with Homebrew:
```
brew install caskroom/cask/brew-cask
brew cask install virtualbox
brew install vagrant
brew install ansible
```

Make sure you are running Ansible v1.7.2 or higher with `ansible --version`.

For other systems, checkout the installation pages of [Vagrant](https://docs.vagrantup.com/v2/installation/), [VirtualBox](https://www.virtualbox.org/wiki/Downloads) and [Ansible](http://docs.ansible.com/intro_installation.html).

### 2. Clone this repo

```
git clone https://github.com/mkrcah/virtual-zookeeper-cluster.git
cd virtual-zookeeper-cluster
```


### 3. Start the cluster with Vagrant

```
vagrant up
```

This command will create and boot the VMs (using VirtualBox), provision each node with JVM and Zookeeper and start the Zookeeper service on each node (using Ansible). Note, that this command might take a while
since it needs to download the Ubuntu image and the JVM.

### 4. Test if the Zookeeper is running

Each VM should now have the Zookeeper running on port 2181. Test that the service is running in non-error state by:
```
echo ruok | nc  192.168.5.100 2181
```

The server should respond with `imok`.

*If interested, checkout the [list of all our-letter commands](http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_zkCommands) supported by the Zookeeper service*


## First steps with distributed Zookeeper

The easiest way to experiment with Zookeeper is to log into one of the machines
and play with the [command-line tool](http://zookeeper.apache.org/doc/current/zookeeperStarted.html#sc_ConnectingToZooKeeper) shipped with the Zookeeper:

##### 1. Log into one of the host machines, e.g. `node-1`
```
vagrant ssh node-1
```

##### 2. Connect to Zookeeper running on localhost with the `zkCli.sh` command-line tool
```
vagrant@node-1:~$ cd /opt/zookeeper-3.4.6/bin/
vagrant@node-1:/opt/zookeeper-3.4.6/bin$ ./zkCli.sh -server localhost:2181
```

##### 3. Create a test znode in the Zookeeper console
```
[zk: localhost:2181(CONNECTED) 3] create /zk_test my_data
Created /zk_test
```

##### 4. Exit the console and connect to Zookeeper running on `node-2` (which runs on `192.168.5.101`)
```
vagrant@node-1:/opt/zookeeper-3.4.6/bin$ ./zkCli.sh -server 192.168.5.101:2181
```

##### 5. Check if the znode `zk_test` is seen on `node-2`
```
[zk: 192.168.5.101:2181(CONNECTED) 0] ls /
[zookeeper, zk_test]
[zk: 192.168.5.101:2181(CONNECTED) 1] get /zk_test
my_data
```

---

Happy Zookeeping!

In case of any questions, drop me an [email](mailto://marcel.krcah@gmail.com) or ping me on [twitter](http://twitter.com/mkrcah).
