```text
SPDX-License-Identifier: Apache-2.0
Copyright © 2021 Intel Corporation
```

- [SEC/INT/ESP/PERFORMANCE/DEPLOY/01: Check if the installation aborts in event of failure of one SE cluster in multi profile provisioning](#secintespperformancedeploy01-Check-if-the-installation-aborts-in-event-of-failure-of-one-SE-cluster-in-multi-profile-provisioning)

   - [Test Summary](#test-summary)
   - [Prerequisites](#prerequisites)
   - [Test steps](#test-steps)

## SEC/INT/ESP/PERFORMANCE/DEPLOY/01: Check if the installation aborts in event of failure of one SE cluster in multi profile provisioning
  
### Test Summary
 
Test focused on the status of deployment if any failure happened on any one of the SE cluster

### Prerequisites:

- Prepare admin machine

### Test steps

1. Configure provisioning setup in admin machine refer the link below

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
4. Configure the "custom.yaml" file with multiple profile - Controller, 5GC and Apps

5. Start online toolchain deployment by executing the command

```shell
[admin_machine]# ./se_install.py -c custom.yaml
```

6. Interupt the deployment at point, while deploying controller, 5GC or Apps

7. Goto the target machines one by one and check the pods were up and running.

 ```shell
 [target_machine]# kubectl get pods -A
 ```
 
 8. Note down the each target machine deployment status
