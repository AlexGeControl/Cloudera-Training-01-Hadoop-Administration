# README

## 0. Network Configuration

---

### Overview

This exercise will:

1. Add public IP addresses to external access point's '/etc/hosts' file to enable monitoring access.
2. Add private IP addresses to each cluster instance's '/etc/hosts' and '/etc/sysconfig/network' files to enable internal communication.
3. Verify:

- External access point is able to access all cluster instances
- Elephant instance, which will be used for Cloudera Manager installation, is able to ping & ssh-tunnel into other cluster instances

4. Launch SOCKS5 proxy server for Cloudera Manager web UI access on exteral access point.

---

### Workflow:

```shell
# 1. Public IP addresses for external access point:
CM_config_local_hosts_file.sh

# 2. Private IP addresses for each cluster instance:
connect_to_elephant.sh
CM_config_hosts.sh
exit

# 3. Verification:
#    On external access point:
connect_to_elephant.sh
connect_to_tiger.sh
connect_to_lion.sh
connect_to_horse.sh
connect_to_monkey.sh

#    On elephant instance:
#    a. Communication:
ping elephant
ping tiger
ping lion
ping horse
ping monkey
#    b. SSH tunnel:
for n in elephant tiger lion horse monkey; do echo $n; ssh training@$n ip addr; done;
#    c. Hostname:
for n in elephant tiger lion horse monkey; do echo $n; ssh training@$n uname -n; done;
#    d. $HOSTNAME:
for n in elephant tiger lion horse monkey; do echo $n; ssh training@$n 'echo $HOSTNAME'; done;

# 4. Start SOCKS5 proxy server for elephant Cloudera Manager instance
start_SOCKS5_proxy.sh
```

---

### External Access Point Configuration

```shell
#!/bin/bash

removeCruft() {
  rm -f /home/training/.ssh/known_hosts
  sudo rm -f /root/.ssh/known_hosts
  sudo sed -i '/elephant/d' /etc/hosts
  sudo sed -i '/tiger/d' /etc/hosts
  sudo sed -i '/horse/d' /etc/hosts
  sudo sed -i '/monkey/d' /etc/hosts
  sudo sed -i '/lion/d' /etc/hosts
}

updateHostsFile() {
  echo ""
  echo "Please supply the EC2 public IP addresses of your five EC2 instances."
  echo ""
  invalidIP=0
  echo "What is the EC2 public IP address of your first machine (elephant)?"
  read ip1
  if [[ $ip1 == 10.* || $ip1 == 172.* || $ip1 == 192.168.* ]]; then
    invalidIP=1
  fi
  echo "What is the EC2 public IP address of your second machine (tiger)?"
  read ip2
  if [[ $ip2 == 10.* || $ip2 == 172.* || $ip2 == 192.168.* ]]; then
    invalidIP=1
  fi
  echo "What is the EC2 public IP address of your third machine (horse)?"
  read ip3
  if [[ $ip3 == 10.* || $ip3 == 172.* || $ip3 == 192.168.* ]]; then
    invalidIP=1
  fi
  echo "What is the EC2 public IP address of your fourth machine (monkey)?"
  read ip4
  if [[ $ip4 == 10.* || $ip4 == 172.* || $ip4 == 192.168.* ]]; then
    invalidIP=1
  fi
  echo "What is the EC2 public IP address of your fifth machine (lion)?"
  read ip5
  if [[ $ip5 == 10.* || $ip5 == 172.* || $ip5 == 192.168.* ]]; then
  invalidIP=1
  fi

  echo
  echo "Please verify that these are the correct EC2 public IP addresses:"
  echo "elephant:" $ip1
  echo "tiger:" $ip2
  echo "horse:" $ip3
  echo "monkey:" $ip4
  echo "lion:" $ip5

  echo
  echo "Type Y if these are the correct IP addresses, N if not."
  read answer
  if [[ $answer != "Y" && $answer != "y" ]]; then
    echo
    echo "Please restart this command and provide correct IP addresses."
    echo
    exit
  fi

  if [[ $invalidIP == "1" ]] ; then
    echo
    echo "You have entered one or more public IP addresses"
    echo "that start with 10, 172, or 192.168."
    echo "OK to Continue? Type Y if you are sure want to proceed."
    read validation
    if [[ $validation != "Y" && $validation != "y" ]]; then
      echo
      echo "Please restart this command and provide correct IP addresses."
      echo
      exit
    fi
  fi

  sudo sh -c "echo $ip1 elephant >> /etc/hosts"
  sudo sh -c "echo $ip2 tiger >> /etc/hosts"
  sudo sh -c "echo $ip3 horse >> /etc/hosts"
  sudo sh -c "echo $ip4 monkey >> /etc/hosts"
  sudo sh -c "echo $ip5 lion >> /etc/hosts"

  return 0
}

MYHOST="`hostname`: "
echo
echo $MYHOST "Running " $0"."
removeCruft
updateHostsFile
echo
echo $MYHOST $0 "done."
```

---

### Cluster Instances Communication Configuration
```shell
#!/bin/bash

removeCruft() {
  if [ -f /home/training/.ssh/known_hosts ]; then
    rm /home/training/.ssh/known_hosts
  fi
  sudo sed -i '/elephant/d' /etc/hosts
  sudo sed -i '/tiger/d' /etc/hosts
  sudo sed -i '/horse/d' /etc/hosts
  sudo sed -i '/monkey/d' /etc/hosts
  sudo sed -i '/lion/d' /etc/hosts
}

determineEnvironment(){
#  if grep -q "K=vmware" /etc/vmbuild.info
#  then
#    classenv=vmware
#  else
    classenv=ec2
#  fi
}

updateHostsFile() {
  if [ $classenv == "vmware" ]; then
    sudo sh -c "echo 192.168.123.1 elephant >> /etc/hosts"
    sudo sh -c "echo 192.168.123.2 tiger >> /etc/hosts"
    sudo sh -c "echo 192.168.123.3 horse >> /etc/hosts"
    sudo sh -c "echo 192.168.123.4 monkey >> /etc/hosts"
    sudo sh -c "echo 192.168.123.5 lion >> /etc/hosts"
  else
    echo ""
    echo "Please supply the EC2 private IP addresses of your four EC2 instances."
    echo "These are the IP addresses that usually start with 10, 172, and 192.168."
    echo ""
    invalidIP=0
    echo "What is the EC2 private IP address of your first machine (elephant)?"
    read ip1
    if [[ $ip1 != 10.* && $ip1 != 172.* && $ip1 != 192.168.* ]]; then
      invalidIP=1
    fi
    echo "What is the EC2 private IP address of your second machine (tiger)?"
    read ip2
    if [[ $ip2 != 10.* && $ip2 != 172.* && $ip2 != 192.168.* ]]; then
      invalidIP=1
    fi
    echo "What is the EC2 private IP address of your third machine (horse)?"
    read ip3
    if [[ $ip3 != 10.* && $ip3 != 172.* && $ip3 != 192.168.* ]]; then
      invalidIP=1
    fi
    echo "What is the EC2 private IP address of your fourth machine (monkey)?"
    read ip4
    if [[ $ip4 != 10.* && $ip4 != 172.* && $ip4 != 192.168.* ]]; then
      invalidIP=1
    fi
    echo "What is the EC2 private IP address of your fifth machine (lion)?"
    read ip5
    if [[ $ip5 != 10.* && $ip5 != 172.* && $ip5 != 192.168.* ]]; then
    invalidIP=1
    fi

    echo
    echo "Please verify that these are the correct IP addresses:"
    echo "elephant:" $ip1
    echo "tiger:" $ip2
    echo "horse:" $ip3
    echo "monkey:" $ip4
    echo "lion:" $ip5

    echo
    echo "Type Y if these are the correct IP addresses, N if not."
    read answer
    if [[ $answer != "Y" && $answer != "y" ]]; then
      echo
      echo "Please restart this command and provide correct IP addresses."
      echo
      exit
    fi

    if [[ $invalidIP == "1" ]] ; then
      echo
      echo "You have entered one or more private IP addresses"
      echo "that do not start with 10, 172, or 192.168."
      echo "OK to Continue? Type Y if you are sure want to proceed."
      read validation
      if [[ $validation != "Y" && $validation != "y" ]]; then
        echo
        echo "Please restart this command and provide correct IP addresses."
        echo
        exit
      fi
    fi

    sudo sh -c "echo $ip1 elephant >> /etc/hosts"
    sudo sh -c "echo $ip2 tiger >> /etc/hosts"
    sudo sh -c "echo $ip3 horse >> /etc/hosts"
    sudo sh -c "echo $ip4 monkey >> /etc/hosts"
    sudo sh -c "echo $ip5 lion >> /etc/hosts"
    return 0
  fi
}

confirmConnectivity() {
for n in $(awk '/\<(elephant|tiger|horse|monkey|lion)\>/{print $2}' /etc/hosts)
do
  if ! ssh -o StrictHostKeyChecking=no -o ConnectTimeout=3 $n '/bin/true'
    then
    echo "ERROR: no response from $n; retry later."
    exit 1
  fi
done
}

copyHostsFile() {
  echo $MYHOST "Copying the /etc/hosts files to the other four hosts in the cluster."
  scp /etc/hosts root@tiger:/etc/hosts
  scp /etc/hosts root@horse:/etc/hosts
  scp /etc/hosts root@monkey:/etc/hosts
  scp /etc/hosts root@lion:/etc/hosts
}

renameHosts() {
  echo
  echo "Changing the /etc/sysconfig/network files on all the hosts in the cluster."
  echo
  sudo sed -i s/localhost.localdomain/elephant/ /etc/sysconfig/network
  ssh training@tiger 'sudo sed -i s/localhost.localdomain/tiger/ /etc/sysconfig/network'
  ssh training@horse 'sudo sed -i s/localhost.localdomain/horse/ /etc/sysconfig/network'
  ssh training@monkey 'sudo sed -i s/localhost.localdomain/monkey/ /etc/sysconfig/network'
  ssh training@lion 'sudo sed -i s/localhost.localdomain/lion/ /etc/sysconfig/network'

  #If no hostname entry found in the network file, add it now
  hostEntryFound=$(cat /etc/sysconfig/network | grep -i 'HOSTNAME')
  if [ "$hostEntryFound" == "" ]; then
 	 sudo sed -i "\$a\HOSTNAME=elephant" /etc/sysconfig/network
  fi
  hostEntryFound=$(ssh training@tiger cat /etc/sysconfig/network | grep -i 'HOSTNAME')
  if [ "$hostEntryFound" == "" ]; then
 	 ssh training@tiger 'sudo sed -i "\$a\HOSTNAME=tiger" /etc/sysconfig/network'
  fi
  hostEntryFound=$(ssh training@horse cat /etc/sysconfig/network | grep -i 'HOSTNAME')
  if [ "$hostEntryFound" == "" ]; then
 	 ssh training@horse 'sudo sed -i "\$a\HOSTNAME=horse" /etc/sysconfig/network'
  fi
  hostEntryFound=$(ssh training@monkey cat /etc/sysconfig/network | grep -i 'HOSTNAME')
  if [ "$hostEntryFound" == "" ]; then
 	 ssh training@monkey 'sudo sed -i "\$a\HOSTNAME=monkey" /etc/sysconfig/network'
  fi
  hostEntryFound=$(ssh training@lion cat /etc/sysconfig/network | grep -i 'HOSTNAME')
  if [ "$hostEntryFound" == "" ]; then
 	 ssh training@lion 'sudo sed -i "\$a\HOSTNAME=lion" /etc/sysconfig/network'
  fi

  echo
  echo "Resetting the host names for the current sessions on all hosts in the cluster."
  echo
  sudo hostname elephant
  ssh training@tiger 'sudo hostname tiger'
  ssh training@horse 'sudo hostname horse'
  ssh training@monkey 'sudo hostname monkey'
  ssh training@lion 'sudo hostname lion'
}

MYHOST="`hostname`: "
# Avoid "sudo: cannot get working directory" errors by
# changing to a directory owned by the training user
cd ~
echo
echo $MYHOST "Running " $0"."
removeCruft
determineEnvironment
updateHostsFile
confirmConnectivity
copyHostsFile
renameHosts
echo
echo $MYHOST $0 "done."
```

---

### SSH tunnel in:
```shell
#!/bin/bash

ssh -p 443 training@elephant
```

---

### SOCKS5 Proxy Server Configuration
```shell
#!/bin/bash

PID="`/usr/bin/pgrep -f '\<[s]sh .*-D 80'`"
if [ -n "$PID" ]
then
  echo 'Aborting - one or more SOCKS5 proxies are already running.'
  echo 'Process ID(s):' $PID'.'
  exit 1
else
  echo -e "\n$HOSTNAME Running ${0##*/}."
  sudo ssh -p 443 -D 80 -g training@elephant
  echo -e "\n$HOSTNAME ${0##*/} done."
fi
```
