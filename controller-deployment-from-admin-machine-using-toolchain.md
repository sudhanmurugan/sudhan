```text
SPDX-License-Identifier: Apache-2.0
Copyright © 2021 Intel Corporation
```
- [SEC/INT/ESP/CTRL/DEPLOY/01: Verify controller deployment from admin machine using online provisioning toolchain](#secintespctrldeploy01-Verify-controller-deployment-from-admin-machine-using-online-provisioning-toolchain)

   - [Test Summary](#test-summary)
   - [Prerequisites](#prerequisites)
   - [Test steps](#test-steps)
  
 - [SEC/INT/ESP/CTRL/DEPLOY/02: Verify controller deployment from admin machine using provisioning tool chain in an air gap environment](#secintespctrldeploy02-Verify-controller-deployment-from-admin-machine-using-provisioning-tool-chain-in-an-air-gap-environment)

   - [Test Summary](#test-summary-1)
   - [Prerequisites](#prerequisites-1)
   - [Test steps](#test-steps-1)

## SEC/INT/ESP/CTRL/DEPLOY/01: Verify controller deployment from admin machine using online provisioning toolchain
  
### Test Summary
 
Test focused on deployment of controller from admin machine using online provisioning toolchain

### Prerequisites:

- Prepare admin machine

### Test steps

1. Configure provisioning setup in admin machine, refer the link below

```shell
https://github.com/intel-innersource/applications.services.smart-edge.virgo/blob/main/get-started.md
```

2. Check out virgo/staging repo by executing the below command:

```shell
 git clone --recursive --branch main https://github.com/intel-innersource/applications.services.smart-edge.staging.git
```
3. Navigate to experience kit directory and generate platfrom config yaml by executing below command

```shell
[admin_machine]# ./se_config.py > custom.yaml
```
4. Configure the "custom.yaml" file

5. Start online toolchain deployment by executing the command

```shell
[admin_machine]# ./se_install.py -c custom.yaml
```
6. Goto the target machine and check the pods were up and running.

 ```shell
 [target_machine]# kubectl get pods -A
 ```
 
 ## SEC/INT/ESP/CTRL/DEPLOY/02: Verify controller deployment from admin machine using provisioning tool chain in an air gap environment
  
### Test Summary
 
Test focused on deployment of controller from admin machine using provisioning tool chain in an air gap environment

### Prerequisites:

- Prepare admin machine

### Test steps

1. Configure provisioning setup in admin machine, refer the link below

```shell
https://github.com/intel-innersource/applications.services.smart-edge.virgo/blob/main/get-started.md
```

2. Check out virgo/staging repo by executing the below command:

```shell
 git clone --recursive --branch main https://github.com/intel-innersource/applications.services.smart-edge.staging.git
```
3. Navigate to experience kit directory and generate platfrom config yaml by executing below command

```shell
[admin_machine]# ./se_config.py > custom.yaml
```
4. Configure the "custom.yaml" file

5. Build the offline build package by executing the below command

```shell
[admin_machine]# ./se_build.py
```

6.  In air gap environment start the deployment using below command

```shell
[admin_machine]# ./se_install.py -c custom.yaml
```
7. In target machine, check the pods were up and running.

 ```shell
 [target_machine]# kubectl get pods -A
 ```
