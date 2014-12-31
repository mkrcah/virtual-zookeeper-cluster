# Virtual Apache ZooKeeper cluster

This script creates a virtual 3-node [Apache ZooKeeper](http://zookeeper.apache.org/)
cluster on your local machine using [Vagrant](https://www.vagrantup.com/), [VirtualBox](https://www.virtualbox.org/) and [Ansible](http://www.ansible.com/home)


### 1. Install Vagrant, VirtualBox and Ansible on your machine

For Mac, it can be done with Homebrew:
```
brew install caskroom/cask/brew-cask
brew cask install virtualbox
brew install vagrant
brew install ansible
```

### 2. Clone this repo

```
git clone https://github.com/mkrcah/virtual-zookeeper-cluster.git
cd virtual-zookeeper-cluster
```

### 3. Customize cluster settings if you want to

By default, the cluster is set to contain 3 nodes, with each node having 256 MB memory.
If you want to change the default settings (more nodes, more memory, different ip addresses or hostnames),
edit the following section of the `Vagrantfile`:

```
memory_mb = 256

cluster = {
  'node-1' => "192.168.5.100",
  'node-2' => "192.168.5.101",
  'node-3' => "192.168.5.102",
}
```

Note that Zookeeper cluster should optimally consist of an odd number of hosts
since it is designed to survive failure of minority of hosts. For example,
Zookeeper running on 3 hosts can survive 1 host failure. Zookeeper running on
4 hosts can also survive only 1 host failure. Zookeper on 5 hosts can survive 2 host failures.

### 4. Create the cluster using Vagrant

```
vagrant up
```

This will create a cluster of VMs, install JVM and Zookeeper to each VM and
start the Zookeeper service. Note, that this command might take a while
since it needs to download the Ubuntu image and the JVM.

### 5. Test if the Zookeeper is running

If everythin runs succesfully, each VM should have the Zookeeper running on port 2181.

You can test that the service is running in non-error state by executing:
```
echo ruok | nc  192.168.5.100 2181
```

The server should respond with `imok`.

(Checkout [the list of all four-letter commands](http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_zkCommands)
that the service supports)

### 6. Go and experiment with the CLI tool shipped with the Zookeeper:

The easiest way to experiment with Zookeeper is to log into one of the machines
and play with the [command-line tool](http://zookeeper.apache.org/doc/current/zookeeperStarted.html#sc_ConnectingToZooKeeper) which comes shipped with the Zookeeper:

1. Log to one of the host machines, e.g. `node-1`, with

```
vagrant ssh node-1
```

2. If you are curious, checkout the Zookeeper log:

```
vagrant@node-1:~$ tail -f -n 100 zookeeper.out
```

3. Connect to `node-1` and create a znode:

Connect to Zookeeper running on localhost:
```
vagrant@node-1:~$ cd /opt/zookeeper-3.4.6/bin/
vagrant@node-1:/opt/zookeeper-3.4.6/bin$ ./zkCli.sh -server localhost:2181
```

And create a znode in the console:
```
[zk: localhost:2181(CONNECTED) 3] create /zk_test my_data
Created /zk_test
```

4. Connect to `node-2` and verify that `node-2` also sees the znode:

Exit the console and connect to `node-2` which runs on `192.168.5.101`:
```
vagrant@node-1:/opt/zookeeper-3.4.6/bin$ ./zkCli.sh -server 192.168.5.101:2181
```

Check if the znode `zk_test` is there:
```
[zk: 192.168.5.101:2181(CONNECTED) 0] ls /
[zookeeper, zk_test]

[zk: 192.168.5.101:2181(CONNECTED) 1] get /zk_test
my_data
...
```

---

Happy Zookeeping!
