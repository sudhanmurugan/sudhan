```text
SPDX-License-Identifier: Apache-2.0
Copyright © 2021 Intel Corporation
```
- [SEC/INT/ESP/EN6/DEPLOY/01: Verify edgenode deployment on x12 machine using toolchain](#secintespen6deploy01-Verify-edgenode-deployment-on-x12-machine-using-toolchain)

   - [Test Summary](#test-summary)
   - [Prerequisites](#prerequisites)
   - [Hardware Requirements](#hardware-requirements)
   - [Test steps](#test-steps)

- [SEC/INT/ESP/EN6/DEPLOY/02: Verify the edgeapps are working on Dell XR12 server with pre-configured profile](#secintespen6deploy02-Verify-the-edgeapps-are-working-on-Dell-XR12-server-with-pre-configured-profile)

   - [Test Summary](#test-summary-1)
   - [Prerequisites](#prerequisites-1)
   - [Hardware Requirements](#hardware-requirements-1)
   - [Test steps](#test-steps-1)
   - 
## SEC/INT/ESP/EN6/DEPLOY/01: Verify edgenode deployment on x12 machine using toolchain
  
### Test Summary
 
Test focused on deployment of controller from admin machine using online provisioning toolchain

### Prerequisites:

- Prepare admin machine
- Target machine shoud be XR12

### Hardware Requirements:
For target machine
- Dell XR12
- Xeon-SP 4314
- 64GB RAM (4x16GB DDR4-3200)
- Two (2) Columbiaville E810-DA2 (N3 N2 - One physical port, N6 + CNI Port- One physical Port, two additional ports)
- 25G connectivity per E810 port
- 4 total ports of E810
- Two storage devices - SATA SSD (no RAID)
- /dev/sda = 480GB
- /dev/sdb = 3.84TB

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
4. Configure the "custom.yaml" file for edgenode

5. Start online toolchain deployment by executing the command

```shell
[admin_machine]# ./se_install.py -c custom.yaml
```
6. Goto the target machine and check the pods were up and running.

 ```shell
 [target_machine]# kubectl get pods -A
 ```
 
 ## SEC/INT/ESP/EN6/DEPLOY/02: Verify the edgeapps are working on Dell XR12 server with pre-configured profile
  
### Test Summary
 
Test focused on deployment of controller from admin machine using online provisioning toolchain

### Prerequisites:

- Prepare admin machine
- Target machine shoud be XR12

### Hardware Requirements:
For target machine
- Dell XR12
- Xeon-SP 4314
- 64GB RAM (4x16GB DDR4-3200)
- Two (2) Columbiaville E810-DA2 (N3 N2 - One physical port, N6 + CNI Port- One physical Port, two additional ports)
- 25G connectivity per E810 port
- 4 total ports of E810
- Two storage devices - SATA SSD (no RAID)
- /dev/sda = 480GB
- /dev/sdb = 3.84TB

### Test steps

1. Deploy EdgeApps into the edgenode and verify the edgeapps running successfully
