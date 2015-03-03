# Hadoop1.04_SingleNodeSetup
Install instructions for Hadoop1.04 in Psudo Distributed mode.

Hadoop 1.0.4 Single node setup instructions:

#Prereq:
sudo apt-get install openjdk-7-jdk
sudo apt-get install emacs    #(optional)
sudo apt-get install ssh

NOTE: nk is root(Admin) user and huser is hadoop user(Standard)
#Add User:    
sudo addgroup hadoop
sudo adduser --ingroup hadoop huser
#setup a password for huser when prompted.
#add huser to sudoers file
nk@ubuntu:/home/huser$ sudo visudo
# add this line to the end of file
huser ALL=(ALL:ALL) ALL
Configuring SSH
nk@ubuntu:~$ su huser
huser@ubuntu:/home/nk$ cd ~
huser@ubuntu:~$ 
huser@ubuntu:~$ ssh-keygen -t rsa
huser@ubuntu:~$ ls -l ~/.ssh/
total 8
-rw------- 1 huser hadoop 1679 Oct 27 11:31 id_rsa
-rw-r--r-- 1 huser hadoop  394 Oct 27 11:31 id_rsa.pub
huser@ubuntu:~$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

huser@ubuntu:~$ ssh localhost
huser@ubuntu:~$ logout

Disabling IPv6
huser@ubuntu:~$ su nk
nk@ubuntu:~$sudo emacs /etc/sysctl.conf

# Add these line to the bottom of the file to disable ipv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
#Restart the service
nk@ubuntu:~$ sudo sysctl -p
#check if restarted
nk@ubuntu:~$ cat /proc/sys/net/ipv6/conf/all/disable_ipv6
#1:restarted disabling ipv6  0:not disabled

Hadoop package installation
download hadoop tar file and unzip and place in home directory
nk@ubuntu:~$ sudo chown -R huser:hadoop /home/nk/hadoop-1.0.4/

Update $HOME/.bashrc

nk@ubuntu:~$ sudo emacs -nw /home/huser/.bashrc
#Add these line to the end of bashrc file
#NOTE: no space after HADOOP_HOME and before ‘=’
export HADOOP_HOME=/home/nk/hadoop-1.0.4
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
export PATH=$PATH:$HADOOP_HOME/bin
#Now apply the changes in current running environment(Terminal)
nk@ubuntu:~$ source /home/huser/.bashrc

Configuration

hadoop-env.sh

nk@ubuntu:~$ sudo emacs -nw /home/nk/hadoop-1.0.4/conf/hadoop-env.sh
#Add this to the end of file
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64

conf/*-site.xml
In this section, we configure the directory where Hadoop will store its data files, the network ports it listens to

#create directories for hadoop and give perm for huser
nk@ubuntu:~$ sudo mkdir -p /app/hadoop/tmp
nk@ubuntu:~$ sudo chown huser:hadoop /app/hadoop/tmp
core-site.xml
nk@ubuntu:~$ sudo emacs -nw /home/nk/hadoop-1.0.4/conf/core-site.xml
#add these lines between <configuration> tags
<property>
  <name>hadoop.tmp.dir</name>
  <value>/app/hadoop/tmp</value>
  <description>A base for other temporary directories.</description>
</property>

<property>
  <name>fs.default.name</name>
  <value>hdfs://localhost:54310</value>
  <description>The name of the default file system.  A URI whose
  scheme and authority determine the FileSystem implementation.  The
  uri's scheme determines the config property (fs.SCHEME.impl) naming
  the FileSystem implementation class.  The uri's authority is used to
  determine the host, port, etc. for a filesystem.</description>
</property>

mapred-site.xml
nk@ubuntu:~$ sudo emacs -nw /home/nk/hadoop-1.0.4/conf/mapred-site.xml
#add btw <configuration> tags
<property>
  <name>mapred.job.tracker</name>
  <value>localhost:54311</value>
  <description>The host and port that the MapReduce job tracker runs
  at.  If "local", then jobs are run in-process as a single map
  and reduce task.
  </description>
</property>

hdfs-site.xml
nk@ubuntu:~$ sudo emacs -nw /home/nk/hadoop-1.0.4/conf/hdfs-site.xml
#add btw <configuration> tags
<property>
  <name>dfs.replication</name>
  <value>1</value>
  <description>Default block replication.
  The actual number of replications can be specified when the file is created.
  The default is used if replication is not specified in create time.
  </description>
</property>


Formatting the HDFS filesystem via the NameNode

nk@ubuntu:~$ su huser
huser@ubuntu:~$ hadoop namenode -format

Starting single-node cluster
huser@ubuntu:~$ /home/nk/hadoop-1.0.4/bin/start-all.sh

Checking:
huser@ubuntu:~$ jps
13327 SecondaryNameNode
13652 TaskTracker
13802 Jps
12811 NameNode
13081 DataNode
13408 JobTracker

nk@ubuntu:/home/huser$  sudo netstat -plten | grep java

Stopping single-node cluster
huser@ubuntu:~$ /home/nk/hadoop-1.0.4/bin/stop-all.sh










REFERENCE:
http://www.michael-noll.com/tutorials/running-hadoop-on-ubuntu-linux-single-node-cluster/

