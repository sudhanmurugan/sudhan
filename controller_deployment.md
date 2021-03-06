```text
SPDX-License-Identifier: Apache-2.0
Copyright © 2021 Intel Corporation
```

- [ITP/SEO/CN/01: Deploy and Verify Controller Deployment](#itpsecpa01-Deploy-and-Verify-Controller-Deployment)

  - [Test Summary](#test-summary)
  - [Prerequisites](#prerequisites)
  - [Hardware Requirements](#hardware-requirements)
  - [Test steps](#test-steps)

  
 
 ## ITP/SEO/CN/01: Deploy and Verify Controller Deployment
 
 ### Test Summary
 
Test focused on deployment of controller

### Prerequisites:

- Repository applications.services.smart-edge.virgo checked out.

### Hardware Requirements

- Dell R750 ICE Lake
- At least 64 GB RAM
- At least 512 GB hard drive
- Kernel Version> 5.11
- An Internet connection
- Ubuntu 20.04


### Test steps

1. Update ```inventory.yaml``` as shown below

 ```shell
all:
  vars:
    cluster_name: node_cluster    # NOTE: Use `_` instead of spaces.
    deployment: controller              # NOTE: Available deployment type: node and controller.
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
 
2. Modify ```inventory/default/group_vars/all/10-open.yml``` as below

  
 ```shell
### Proxy settings

proxy_env:
  # Proxy URLs to be used for HTTP, HTTPS and FTP
  # Comment out proxy settings except no_proxy while deploying verification_controller on AWS
  http_proxy: "http://<INTEL PROXY>:<PORT>"
  https_proxy: "http://<INTEL PROXY>:<PORT>"
  ftp_proxy: "http://<INTEL PROXY>:<PORT>"
  # No proxy setting contains addresses and networks that should not be accessed using proxy (e.g. local network, Kubernetes CNI networks)
  no_proxy: "http://<INTEL PROXY>:<PORT>"
  # all proxy need to use socks5 proxy to connect nats based servers(case with platform attestation components)
  all_proxy: ""
  
### Network Time Protocol (NTP)
# Enable machine's time synchronization with NTP server
ntp_enable: true
# Servers to be used by NTP instead of the default ones (e.g. 0.centos.pool.ntp.org)
ntp_servers: [ntp-host1.ir.intel.com]

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
   - "http://<DOCKER_REGISTRY_MIRROR>:<PORT>"

###############
## Platform attestation specific configurations

# Install isecl attestation components (TA, ihub, isecl k8s controller and scheduler extension)
platform_attestation_node: false
```

3. Add "git_repo_token" on ```inventory/default/group_vars/all/20-enhanced.yml```

4. Run the deployment using ```./deploy.sh ```

5. Once Deployment successful, verify only cert-manager and kube-system pods are running 

```shell
root@esi-xx:/home/smartedge-open/applications.services.smart-edge.virgo/experience-kit# kubectl get pods -A
NAMESPACE      NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager   cert-manager-779bc6bf59-mqrqm              1/1     Running   0          4d8h
cert-manager   cert-manager-cainjector-55765d687b-tf6l7   1/1     Running   0          4d8h
cert-manager   cert-manager-webhook-85d85f46c7-2ldz4      1/1     Running   0          4d8h
kube-system    calico-kube-controllers-5cdd5b4947-sx66q   1/1     Running   0          4d8h
kube-system    calico-node-zpcxz                          1/1     Running   0          4d8h
kube-system    coredns-64897985d-5h57v                    1/1     Running   0          4d8h
kube-system    coredns-64897985d-lb5c5                    1/1     Running   0          4d8h
kube-system    etcd-esi-xx                                1/1     Running   0          4d8h
kube-system    kube-apiserver-esi-xx                      1/1     Running   0          4d8h
kube-system    kube-controller-manager-esi-xx             1/1     Running   0          4d8h
kube-system    kube-proxy-8vhrh                           1/1     Running   0          4d8h
kube-system    kube-scheduler-esi-xx                      1/1     Running   0          4d8h
```
