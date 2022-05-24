```text
SPDX-License-Identifier: Apache-2.0
Copyright © 2021 Intel Corporation
```

- [ITP/SEO/HP/01: Set “default_hugepage as 1G” and “amount of hugepage_1G as 4” and verify the system resource and memory info](#itpseohp01-Set-“default_hugepage-as-1G”-and-“amount-of-hugepage_1G-as-4”-and-verify-the-system-resource-and-memory-info)

  - [Test Summary](#test-summary)
  - [Prerequisites](#prerequisites)
  - [Hardware Requirements](#hardware-requirements)
  - [Test steps](#test-steps)

- [ITP/SEO/HP/02: Set “default_hugepage as 2M” and “amount of hugepage_2M as 150” and verify the system resource and memory info](#itpseohp02-Set-“default_hugepage-as-2M”-and-“amount-of-hugepage_2M-as-150”-and-verify-the-system-resource-and-memory-info)

  - [Test Summary](#test-summary-1)
  - [Prerequisites](#prerequisites-1)
  - [Hardware Requirements](#hardware-requirements-1)
  - [Test steps](#test-steps-1)

- [ITP/SEO/HP/03: Configure hugepage to 1G in an already deployed system with 2M configured, and check if both the sizes are updated in the cluster](#itpseohp03-Configure-hugepage-to-1G-in-an-already-deployed-system-with-2M-configured,-and-check-if-both-the-sizes-are-updated-in-the-cluster)

  - [Test Summary](#test-summary-2)
  - [Prerequisites](#prerequisites-2)
  - [Hardware Requirements](#hardware-requirements-2)
  - [Test steps](#test-steps-2)

  
 
 ## ITP/SEO/HP/01: Set “default_hugepage as 1G” and “amount of hugepage_1G as 4” and verify the system resource and memory info
 
 ### Test Summary
 
Test focused on deployment of dek with hugepage_1G enabled

### Prerequisites:

- Repository applications.services.smart-edge-open.developer-experience-kits-open checked out.
- Refer below link to verify whether the server has met all generic deployment preconditions - https://github.com/otcshare/specs/blob/master/doc/getting-started/openness-cluster-setup.md#preconditions

### Hardware Requirements

- Dell R750 ICE Lake
- At least 64 GB RAM
- At least 512 GB hard drive
- Kernel Version> 5.11
- An Internet connection
- Ubuntu 20.04


### Test steps

1. Update inventory.yaml as shown below

 ```shell
all:
  vars:
    cluster_name: node_cluster    # NOTE: Use `_` instead of spaces.
    deployment: dek              # NOTE: Available deployment type: node and controller.
    single_node_deployment: true  # Request single node deployment (true/false).
    limit:                        # Limit ansible deployment to certain inventory group or hosts
controller_group:
  hosts:
    controller:
      ansible_host: <IP>
      ansible_user: smartedge-open
edgenode_group:
  hosts:
    node01:
      ansible_host: <IP>
      ansible_user: smartedge-open
 ```     
 
2. Modify inventory/default/group_vars/all/10-open.yml as below

  
 ```shell
### Proxy settings

proxy_env:
  # Proxy URLs to be used for HTTP, HTTPS and FTP
  # Comment out proxy settings except no_proxy while deploying verification_controller on AWS
  http_proxy: "http://proxy-mu.intel.com:911"
  https_proxy: "http://proxy-mu.intel.com:912"
  ftp_proxy: "http://proxy-mu.intel.com:911"
  # No proxy setting contains addresses and networks that should not be accessed using proxy (e.g. local network, Kubernetes CNI networks)
  no_proxy: "localhost,virt-api,.svc,.svc.cluster.local,cdi-api,127.0.0.1,10.96.0.0/12,10.32.0.0/12,10.245.0.0/16,10.66.253.112,10.66.253.112,10.190.212.225,localhost,virt-api,.svc,cdi-api,127.0.0.1,10.244.0.0/16,10.96.0.0/16,10.16.0.0/16,192.168.10.0/24,10.190.212.0/23,10.166.31.0/24,10.190.204.128/25,10.66.253.112"
  # all proxy need to use socks5 proxy to connect nats based servers(case with platform attestation components)
  all_proxy: ""
  
### Network Time Protocol (NTP)
# Enable machine's time synchronization with NTP server
ntp_enable: true
# Servers to be used by NTP instead of the default ones (e.g. 0.centos.pool.ntp.org)
ntp_servers: [ntp-host1.ir.intel.com]

# Enable hugepages
hugepages_enabled: true
# Size of a single hugepage (2M or 1G)
default_hugepage_size: 1G
# Amount of hugepages
hugepages_1G: 4
hugepages_2M: 0

## SR-IOV Network Operator
sriov_network_operator_enable: false

## SR-IOV Network Operator configuration
sriov_network_operator_configure_enable: false

## SR-IOV Network Nic's Detection Application
sriov_network_detection_application_enable: false

### Software Guard Extensions
# SGX requires kernel 5.11+, SGX enabled in BIOS and access to PCC service
sgx_enabled: false

## Docker registry mirrors
## https://docs.docker.com/registry/recipes/mirror/
docker_registry_mirrors:
   - "http://10.190.212.225:5000"

###############
## Platform attestation specific configurations

# Install isecl attestation components (TA, ihub, isecl k8s controller and scheduler extension)
platform_attestation_node: false
```
3. Run the deployment using ```./deploy.sh ```

4. Once Deployment successful, verify the system resource and memory info 

```shell
root@esi-xx:~# kubectl describe nodes esi-xx | grep hug
  hugepages-1Gi:                     4Gi
  hugepages-2Mi:                     0
  hugepages-1Gi:                     4Gi
  hugepages-2Mi:                     0
  hugepages-1Gi                     1Gi (25%)    1Gi (25%)
  hugepages-2Mi                     0 (0%)       0 (0%)
  
root@esi-xx:~# cat /proc/meminfo  | grep Hug
AnonHugePages:     94208 kB
ShmemHugePages:        0 kB
FileHugePages:         0 kB
HugePages_Total:       4
HugePages_Free:        3
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:    1048576 kB
Hugetlb:         4194304 kB

```
5. Create a pod with size as 5G 

```shell
apiVersion: v1
kind: Pod
metadata:
  name: huge-pages-example
spec:
  containers:
  - name: example
    image: fedora:latest
    command:
    - sleep
    - inf
    volumeMounts:
    - mountPath: /hugepages
      name: hugepage
    resources:
      limits:
        hugepages-1Gi: 5Gi
        memory: 5Gi
      requests:
        memory: 5Gi
  volumes:
  - name: hugepage
    emptyDir:
      medium: HugePages
 ```
 6. Check whether the pod should not be running

```shell
root@esi-xx:~# kubectl describe pod huge-pages-example
```

 ## ITP/SEO/HP/02: Set “default_hugepage as 2M” and “amount of hugepage_2M as 150” and verify the system resource and memory info
 
 ### Test Summary
 
Test focused on deployment of dek with hugepage_2M enabled

### Prerequisites:

- Repository applications.services.smart-edge-open.developer-experience-kits-open checked out.
- Refer below link to verify whether the server has met all generic deployment preconditions - https://github.com/otcshare/specs/blob/master/doc/getting-started/openness-cluster-setup.md#preconditions

### Hardware Requirements

- Dell R750 ICE Lake
- At least 64 GB RAM
- At least 512 GB hard drive
- Kernel Version> 5.11
- An Internet connection
- Ubuntu 20.04


### Test steps

1. Update inventory.yaml as shown below

 ```shell
all:
  vars:
    cluster_name: node_cluster    # NOTE: Use `_` instead of spaces.
    deployment: dek              # NOTE: Available deployment type: node and controller.
    single_node_deployment: true  # Request single node deployment (true/false).
    limit:                        # Limit ansible deployment to certain inventory group or hosts
controller_group:
  hosts:
    controller:
      ansible_host: <IP>
      ansible_user: smartedge-open
edgenode_group:
  hosts:
    node01:
      ansible_host: <IP>
      ansible_user: smartedge-open
 ```     
 
2. Modify inventory/default/group_vars/all/10-open.yml as below

  
 ```shell
### Proxy settings

proxy_env:
  # Proxy URLs to be used for HTTP, HTTPS and FTP
  # Comment out proxy settings except no_proxy while deploying verification_controller on AWS
  http_proxy: "http://proxy-mu.intel.com:911"
  https_proxy: "http://proxy-mu.intel.com:912"
  ftp_proxy: "http://proxy-mu.intel.com:911"
  # No proxy setting contains addresses and networks that should not be accessed using proxy (e.g. local network, Kubernetes CNI networks)
  no_proxy: "localhost,virt-api,.svc,.svc.cluster.local,cdi-api,127.0.0.1,10.96.0.0/12,10.32.0.0/12,10.245.0.0/16,10.66.253.112,10.66.253.112,10.190.212.225,localhost,virt-api,.svc,cdi-api,127.0.0.1,10.244.0.0/16,10.96.0.0/16,10.16.0.0/16,192.168.10.0/24,10.190.212.0/23,10.166.31.0/24,10.190.204.128/25,10.66.253.112"
  # all proxy need to use socks5 proxy to connect nats based servers(case with platform attestation components)
  all_proxy: ""
  
### Network Time Protocol (NTP)
# Enable machine's time synchronization with NTP server
ntp_enable: true
# Servers to be used by NTP instead of the default ones (e.g. 0.centos.pool.ntp.org)
ntp_servers: [ntp-host1.ir.intel.com]

# Enable hugepages
hugepages_enabled: true
# Size of a single hugepage (2M or 1G)
default_hugepage_size: 2M
# Amount of hugepages
hugepages_1G: 0
hugepages_2M: 150

## SR-IOV Network Operator
sriov_network_operator_enable: false

## SR-IOV Network Operator configuration
sriov_network_operator_configure_enable: false

## SR-IOV Network Nic's Detection Application
sriov_network_detection_application_enable: false

### Software Guard Extensions
# SGX requires kernel 5.11+, SGX enabled in BIOS and access to PCC service
sgx_enabled: false

## Docker registry mirrors
## https://docs.docker.com/registry/recipes/mirror/
docker_registry_mirrors:
   - "http://10.190.212.225:5000"

###############
## Platform attestation specific configurations

# Install isecl attestation components (TA, ihub, isecl k8s controller and scheduler extension)
platform_attestation_node: false
```
3. Run the deployment using ```./deploy.sh ```

4. Once Deployment successful, verify the system resource and memory info 

```shell
root@esi-xx:~# kubectl describe nodes esi-xx | grep hug
  hugepages-1Gi:                     0
  hugepages-2Mi:                     300Mi
  hugepages-1Gi:                     0
  hugepages-2Mi:                     300Mi
  hugepages-1Gi                     0 (0%)    0 (0%)
  hugepages-2Mi                     300Mi (100%)       300Mi (1000%)
  
root@esi-xx:~# cat /proc/meminfo  | grep Hug
AnonHugePages:     223232 kB
ShmemHugePages:        0 kB
FileHugePages:         0 kB
HugePages_Total:       150
HugePages_Free:        122
HugePages_Rsvd:        66
HugePages_Surp:        0
Hugepagesize:    2048 kB
Hugetlb:         307200 kB

```
5. Create a pod with size as 400M

```shell
apiVersion: v1
kind: Pod
metadata:
  name: huge-pages-example
spec:
  containers:
  - name: example
    image: fedora:latest
    command:
    - sleep
    - inf
    volumeMounts:
    - mountPath: /hugepages
      name: hugepage
    resources:
      limits:
        hugepages-2Mi: 400Mi
        memory: 400Mi
      requests:
        memory: 400Mi
  volumes:
  - name: hugepage
    emptyDir:
      medium: HugePages
 ```
 6. Check whether the pod should not be running

```shell
root@esi-xx:~# kubectl describe pod huge-pages-example
```

## ITP/SEO/HP/03: Configure hugepage to 1G in an already deployed system with 2M configured, and check if both the sizes are updated in the cluster
 
 ### Test Summary
 
Test the hugepage size has been changed from 2M to 1G

### Prerequisites:

- Repository applications.services.smart-edge-open.developer-experience-kits-open checked out.
- Refer below link to verify whether the server has met all generic deployment preconditions - https://github.com/otcshare/specs/blob/master/doc/getting-started/openness-cluster-setup.md#preconditions

### Hardware Requirements

- Dell R750 ICE Lake
- At least 64 GB RAM
- At least 512 GB hard drive
- Kernel Version> 5.11
- An Internet connection
- Ubuntu 20.04


### Test steps

1. Update inventory.yaml as shown below

 ```shell
all:
  vars:
    cluster_name: node_cluster    # NOTE: Use `_` instead of spaces.
    deployment: dek              # NOTE: Available deployment type: node and controller.
    single_node_deployment: true  # Request single node deployment (true/false).
    limit:                        # Limit ansible deployment to certain inventory group or hosts
controller_group:
  hosts:
    controller:
      ansible_host: <IP>
      ansible_user: smartedge-open
edgenode_group:
  hosts:
    node01:
      ansible_host: <IP>
      ansible_user: smartedge-open
 ```     
 
2. Modify inventory/default/group_vars/all/10-open.yml as below

  
 ```shell
### Proxy settings

proxy_env:
  # Proxy URLs to be used for HTTP, HTTPS and FTP
  # Comment out proxy settings except no_proxy while deploying verification_controller on AWS
  http_proxy: "http://proxy-mu.intel.com:911"
  https_proxy: "http://proxy-mu.intel.com:912"
  ftp_proxy: "http://proxy-mu.intel.com:911"
  # No proxy setting contains addresses and networks that should not be accessed using proxy (e.g. local network, Kubernetes CNI networks)
  no_proxy: "localhost,virt-api,.svc,.svc.cluster.local,cdi-api,127.0.0.1,10.96.0.0/12,10.32.0.0/12,10.245.0.0/16,10.66.253.112,10.66.253.112,10.190.212.225,localhost,virt-api,.svc,cdi-api,127.0.0.1,10.244.0.0/16,10.96.0.0/16,10.16.0.0/16,192.168.10.0/24,10.190.212.0/23,10.166.31.0/24,10.190.204.128/25,10.66.253.112"
  # all proxy need to use socks5 proxy to connect nats based servers(case with platform attestation components)
  all_proxy: ""
  
### Network Time Protocol (NTP)
# Enable machine's time synchronization with NTP server
ntp_enable: true
# Servers to be used by NTP instead of the default ones (e.g. 0.centos.pool.ntp.org)
ntp_servers: [ntp-host1.ir.intel.com]

# Enable hugepages
hugepages_enabled: true
# Size of a single hugepage (2M or 1G)
default_hugepage_size: 2M
# Amount of hugepages
hugepages_1G: 0
hugepages_2M: 150

## SR-IOV Network Operator
sriov_network_operator_enable: false

## SR-IOV Network Operator configuration
sriov_network_operator_configure_enable: false

## SR-IOV Network Nic's Detection Application
sriov_network_detection_application_enable: false

### Software Guard Extensions
# SGX requires kernel 5.11+, SGX enabled in BIOS and access to PCC service
sgx_enabled: false

## Docker registry mirrors
## https://docs.docker.com/registry/recipes/mirror/
docker_registry_mirrors:
   - "http://10.190.212.225:5000"

###############
## Platform attestation specific configurations

# Install isecl attestation components (TA, ihub, isecl k8s controller and scheduler extension)
platform_attestation_node: false
```
3. Run the deployment using ```./deploy.sh ```

4. Once Deployment successful, verify the system resource and memory info 

```shell
root@esi-xx:~# kubectl describe nodes esi-xx | grep hug
  hugepages-1Gi:                     0
  hugepages-2Mi:                     300Mi
  hugepages-1Gi:                     0
  hugepages-2Mi:                     300Mi
  hugepages-1Gi                     0 (0%)    0 (0%)
  hugepages-2Mi                     300Mi (100%)       300Mi (1000%)
  
root@esi-xx:~# cat /proc/meminfo  | grep Hug
AnonHugePages:     223232 kB
ShmemHugePages:        0 kB
FileHugePages:         0 kB
HugePages_Total:       150
HugePages_Free:        122
HugePages_Rsvd:        66
HugePages_Surp:        0
Hugepagesize:    2048 kB
Hugetlb:         307200 kB

```
5. Now change hugepage to 1G in an already deployed system with 2M configured

```shell
root@esi-xx:~# vi /etc/default/grub

append:
GRUB_CMDLINE_LINUX_DEFAULT="default_hugepagesz=1G hugepagesz=1G hugepages=8 intel_iommu=on iommu=pt rdt=l3cat"



