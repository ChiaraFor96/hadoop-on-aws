# hadoop-on-aws
In this repository I show you how to setup a plain Hadoop cluster, in particular using EC2 machines.
But you con consider it for a plain cluster of on premis machines.

## Single machine cluster set up
This part is expired from these tutorials:
- http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html
- https://www.edureka.co/blog/install-hadoop-single-node-hadoop-cluster

Start.
- Download hadoop: `wget https://apache.osuosl.org/hadoop/common/stable/hadoop-3.3.1.tar.gz`
- unpack the tar: `tar -xvf hadoop-3.3.1.tar.gz`
- Install _java_ `sudo apt-get -y install openjdk-8-jdk-headless`

From now assume that the upacked tar forlder name is HADOOP_FILES
- modify the file `HADOOP_FILES/etc/hadoop/hadoop-env.sh` adding `export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64`
- Change the `.bashrc` file adding these rows:
   ```
   #set hadoop home
   export HADOOP_HOME=$HOME/hadoop-3.3.1
   export HADOOP_CONF_DIR=$HOME/hadoop-3.3.1/etc/hadoop
   export HADOOP_MAPRED_HOME=$HOME/hadoop-3.3.1
   export HADOOP_COMMON_HOME=$HOME/hadoop-3.3.1
   export HADOOP_HDFS_HOME=$HOME/hadoop-3.3.1
   export YARN_HOME=$HOME/hadoop-3.3.1
   export PATH=$PATH:$HOME/hadoop-3.3.1/bin

   # set java home
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
   export PATH=/usr/lib/jvm/java-8-openjdk-amd64:$PATH
   export PDSH_RCMD_TYPE=ssh
    
- Run `source .bashrc` for reload the file
- modify the configurations file that are in `HADOOP_FILES/etc/hadoop/`
   -  for the file `core-site.xml`
      ```
      <configuration>
      <property>
      <name>fs.default.name</name>
      <value>hdfs://localhost:9000</value>
      </property>
      </configuration>
      ```

   -  for the file `hdfs-site.xml`
      ```
      <configuration>
      <property>
      <name>dfs.replication</name>
      <value>1</value>
      </property>
      <property>
      <name>dfs.permission</name>
      <value>false</value>
      </property>
      </configuration>
      ```
      
   -  for the file `yarn-site.xml`
      ```
      <configuration>
      <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
      </property>
      <property>
      <name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>
      <value>org.apache.hadoop.mapred.ShuffleHandler</value>
      </property>
      </configuration>
      ```
      
   -  for the file `mapred-site.xml` 
      ```
      <configuration>
      <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
      </property>
      <property>
      <name>yarn.app.mapreduce.am.env</name>
      <value>HADOOP_MAPRED_HOME=/home/ubuntu/HADOOP_FILES</value>
      </property>
      <property>
      <name>mapreduce.map.env</name>
      <value>HADOOP_MAPRED_HOME=/home/ubuntu/HADOOP_FILES</value>
      </property>
      <property>
      <name>mapreduce.reduce.env</name>
      <value>HADOOP_MAPRED_HOME=/home/ubuntu/HADOOP_FILES</value>
      </property>
      </configuration>
      ```
    
Open ports 9870 and 8088 to the IPs that you want to see web UIs of hadoop

### Run the single machine cluster
- http://machineIP:9870/dfshealth.html for see the cluster status

## Cluster set up
For this part I follow this two tutorials and some stackoverflow questions:
- https://www.edureka.co/blog/setting-up-a-multi-node-cluster-in-hadoop-2.X
- http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html

Start.

Download and upack the tar file in each machine, and install some programs:
- Download hadoop: `wget https://apache.osuosl.org/hadoop/common/stable/hadoop-3.3.1.tar.gz`
- unpack the tar: `tar -xvf hadoop-3.3.1.tar.gz`
- Install _java_ `sudo apt-get -y install openjdk-8-jdk-headless`
- `sudo apt install firewalld`
- `sudo apt install net-tools`

Disable firewall:
- `sudo systemctl stop firewalld`
- `sudo systemctl disable firewalld`
- `sudo service firewalld stop`
- `sudo ufw disable`

Change `/etc/hosts` file adding private IPs of all cluster machines like follow (in AWS the private ip not changes and the machines are in the same subnet)
```
172.31.4.195 master
172.31.11.53 slave1
172.31.5.5 slave2
```


Restart host service: `service sshd restart`

- Change the `.bashrc` file adding these rows:
  ```
  export HADOOP_HOME=$HOME/server/hadoop-3.3.1
  export HADOOP_CONF_DIR=$HOME/server/hadoop-3.3.1/etc/hadoop
  export HADOOP_MAPRED_HOME=$HOME/server/hadoop-3.3.1
  export HADOOP_COMMON_HOME=$HOME/server/hadoop-3.3.1
  export HADOOP_HDFS_HOME=$HOME/server/hadoop-3.3.1
  export HADOOP_YARN_HOME=$HOME/server/hadoop-3.3.1
  export PATH=$PATH:$HOME/server/hadoop-3.3.1/bin

  export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
  export PATH=/usr/lib/jvm/java-8-openjdk-amd64:$PATH
  ```

- Run `source .bashrc` for reload the file

From now assume that the upacked tar forlder name is HADOOP_FILES

Open ports:
- 9870 and 8088 to the IPs that you want to see web UIs of hadoop
- 0 - 65535 between hosts for communicate each other

- modify the configurations file that are in `HADOOP_FILES/etc/hadoop/`
   -  TODO

### Start the cluster
- the first time run `hadoop namenode -format`
- `HADOOP_FILES/sbin/start-all.sh`

### Stop the cluster
- `HADOOP_FILES/sbin/stop-all.sh`

### Before restart (ONLY if there are problems)
- `rm -r /tmp/` (consider to delete only the hadoop folders inside /tmp)
- `rm -r datanode/current` in each datanode and in the name node
- `rm -r namenode/current` in the name node


