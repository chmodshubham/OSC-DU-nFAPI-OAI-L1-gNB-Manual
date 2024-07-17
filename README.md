---
title: 'OSC DU nFAPI + OAI L1 gNB Manual [oai_nfapi]'
tags: [lab]

---

# OSC DU nFAPI + OAI L1 gNB Manual [oai_nFAPI]
>[time=Mon, Jul 15, 2024 1:29 PM][name=陳奕全]

## Topology
![image](https://github.com/user-attachments/assets/4f5139ae-adb1-4924-bcd4-4675aea75d5f)
 
> [!IMPORTANT]
> change(based on):

* OSC-DU Rel-I
* [cisco open-nFAPI](https://github.com/cisco/open-nFAPI)  OAI用的版本 commit: b3bc579b1697eab829d5d8a2de59c93a61b88fa4 `Commits on Dec 20, 2017`
* Openairinterface branch [nfapi-fixes-Ming-TDD-table](https://gitlab.eurecom.fr/oai/openairinterface5g/-/tree/nfapi-fixes-Ming-TDD-table?ref_type=heads)
---
<!-- * [BMW's OAI CU](https://github.com/NTUST-BMW-Lab/bmwlab_tony_cu_du) -->

* [BMW's OAI CU](https://github.com/Richard-yq/bmwlab_tony_cu_du)

### Architecture
![image](https://github.com/user-attachments/assets/87e34543-2708-438a-b48a-f56620ee766a)


### Hardware Requirments
![image](https://github.com/user-attachments/assets/d7320c67-5879-4fef-9932-617aa68b8ca6)

#### OAI

![image](https://github.com/user-attachments/assets/9520e489-ee0f-4f6b-b450-e1a3cb472755)

#### OSC
:::

# How to start

Change the branch to `oai_nfapi`

```bash=
git clone https://github.com/Richard-yq/o-du-l2
cd o-du-l2
git checkout oai_nfapi
```

![image](https://github.com/user-attachments/assets/6d3bed40-b8a2-4788-a91e-c9ac2963699d)

Check make options:
```bash=
cd o-du-l2/build/odu
make help
```
![image](https://github.com/user-attachments/assets/aca4dd80-365a-4e6e-a69e-c550a8bc9b86)

>NFAPI

## 0. Modify PLMN ID
### 0.1 Assign IP address
```shell=
#!/bin/bash
echo "***** Assigning IP addresses *****"
INTERFACE=$(ip route | grep default | sed -e "s/^.*dev.//" -e "s/.proto.*//")
INTERFACE="$(echo -e "${INTERFACE}" | tr -d '[:space:]')"

# Clear state
sudo ifconfig $INTERFACE:RIC_STUB down
sudo ifconfig $INTERFACE:CU_STUB down
sudo ifconfig $INTERFACE:ODU down
sudo ifconfig $INTERFACE:OAI_CU down
sudo ifconfig lo:RIC_STUB down
sudo ifconfig lo:CU_STUB  down
sudo ifconfig lo:ODU  down
sudo ifconfig lo:OAI_CU down

sudo ifconfig $INTERFACE:RIC_STUB "192.168.130.80"
sudo ifconfig $INTERFACE:ODU  "192.168.130.81"
sudo ifconfig $INTERFACE:CU_STUB "192.168.130.82"
sudo ifconfig $INTERFACE:OAI_CU "192.168.130.83"

sudo ifconfig lo:RIC_STUB "192.168.130.80"
sudo ifconfig lo:ODU  "192.168.130.81"
sudo ifconfig lo:OAI_CU "192.168.130.83"

PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/home/gnb/
export PATH
echo  "O-RAN NIC Setting Complete!"
```
![image](https://github.com/user-attachments/assets/b5860869-9548-4d6f-ac26-b8cfb99c3210)

![image](https://github.com/user-attachments/assets/37016585-09e4-4274-982b-8b4e61f628c1)

#### BMW OAI CU

![image](https://github.com/user-attachments/assets/a2f11e4d-cae5-4f51-b604-d5de6e66c2e6)

---
### 0.2 Modifying configuration

* Modifying Configuration of OAI PHY

![image](https://github.com/user-attachments/assets/5fbbfcb9-cc2c-45c1-bac0-3f54c2a0bc64)

>-   Set PLMN, it needs to be the same as O-DU Configuration

* Modifying Configuration of OAI CU

![image](https://github.com/user-attachments/assets/a599b671-7827-46bb-8c0a-61017192c582)

>-   Set PLMN, it needs to be the same as O-DU Configuration
>-   Set local address for O-CU
>-   Set remote address for O-DU

* DL pointA
![image](https://github.com/user-attachments/assets/cf1f65b5-2614-40da-b7cb-df59824c03ee)

* UL pointA
![image](https://github.com/user-attachments/assets/efeb5203-fec8-4ce6-b12e-c9e2b30f9590)

**[Detail:](https://hackmd.io/@Richard-quan/BkxLE0Q5p/%2FziGxQy05S0i1TdsZaYG0HQ#OAI-CU-config-file)** -->


## 1. Build OAI gNB

### 1.1 OAI CU

> [!TIP]
> Reference: https://hackmd.io/@JoJoWei/S11LqgmHh#Tony%E2%80%99s-OAI-CU-and-OSC-L2-rel-G

> [!IMPORTANT]
> repo: https://github.com/Richard-yq/bmwlab_tony_cu_du

```bash=
# compile all of the code
git clone https://github.com/Richard-yq/bmwlab_tony_cu_du
cd ~/bmwlab_tony_cu_du
git checkout master
```
```bash=
cd ~/bmwlab_tony_cu_du/oai_cu/cmake_targets
./build_oai --gNB --nrUE -w SIMU

# sync up the format of F1AP
./f1ap_codec_mod.sh

# compile the part of RAN
cd ~/bmwlab_tony_cu_du/oai_cu/cmake_targets
./build_oai --gNB
```
![image](https://github.com/user-attachments/assets/6e8dda68-d0dd-485c-bde2-517ce7702d64)

### 1.2 OSC DU

> [!IMPORTANT]
> repo: https://github.com/Richard-yq/o-du-l2  

In this work, the flag ==NFAPI=YES== must be set.

```shell=
git clone https://github.com/Richard-yq/o-du-l2
cd o-du-l2
git checkout oai_nFAPI
```

```bash=
cd o-du-l2/build/odu
#clean
make clean_odu MACHINE=BIT64 MODE=TDD NFAPI=YES
make clean_ric MACHINE=BIT64 MODE=TDD NFAPI=YES

#compile
make odu MACHINE=BIT64 MODE=TDD NFAPI=YES
make ric_stub NODE=TEST_STUB MACHINE=BIT64 MODE=TDD NFAPI=YES
```


### 1.3 OAI PHY

> [!IMPORTANT]  
> repo: https://github.com/dong881/openairinterface5g-NTUST

```shell=
git clone https://github.com/dong881/openairinterface5g-NTUST.git
cd openairinterface5g
git checkout OSC-use-nfapi
```

```shell=
cd ~/openairinterface5g/cmake_targets

#compile
./build_oai -I -c -C
sudo ./build_oai -c --ninja --nrUE --gNB
```

![image](https://github.com/user-attachments/assets/dfda3c61-5728-4084-b322-8c7a9bed207a)

## 2. Execution

> [!CAUTION]
> **Note**: CU and RIC must be executed first, followed by starting DU, and finally executing OAI L1 (PNF). <br>
> And please check that you have already assigned IP : [ref](https://hackmd.io/kJDv7rgISu-VrbkH-ShXTw#01-Assign-IP-address) <br> <br>
> ![image](https://github.com/user-attachments/assets/d9da07ed-130a-4ff5-8224-62e038d2aae7)

#### OAI CU

```bash=
cd bmwlab_tony_cu_du/oai_cu/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/cu_fdd_gnb.sa.band66.fr1.106PRB.usrpb210.conf --sa
```

![image](https://github.com/user-attachments/assets/dedb7655-30c8-4c00-bd8d-16fbcbd13653)

#### RIC stub

```bash
cd o-du-l2/bin/ric_stub
sudo ./ric_stub
```

![image](https://github.com/user-attachments/assets/e7b14b85-2796-41c2-8bf2-ab731ffbce07)

#### OSC DU

```bash
cd o-du-l2/bin/odu
sudo ./odu
```

![image](https://github.com/user-attachments/assets/3344526f-0ef1-4ce0-8e24-641d0e8695ca)

#### OAI Layer1+RFsim

```bash
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O NTUST-OSC-DU-nFAPI/bmw/config/oaiL1.nfapi.usrpb210.conf --nfapi PNF --rfsim --rfsimulator.serveraddr server --sa
```

![image](https://github.com/user-attachments/assets/15aa7304-a136-4ac6-9484-75c38a0db534)

#### OAI UEsim
```bash=
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --sa --uicc0.imsi 001010000000001 --rfsim
```

