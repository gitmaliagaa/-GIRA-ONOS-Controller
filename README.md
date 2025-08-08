# ONOS Controller Installation Guide

This document walks you through installing and configuring the ONOS SDN controller on a Linux environment.

Minimum System Requirements
- *Operating System: Ubuntu 20.04 or 22.04*
- *Hardware: 2 vCPU, 4 GB RAM, 20 GB of storage*
- *Software: Java 11*

## Step 1: Install Dependencies
Ensure your system is fully updated and equipped with the necessary tools.
```
sudo apt update && sudo apt install -y \
git curl unzip zip openjdk-11-jdk build-essential \
net-tools telnet ssh
```

## Step 2: Clone the ONOS Repository
Navigate to your home directory and clone the official ONOS repository.
```
cd ~
git clone https://gerrit.onosproject.org/onos
cd onos
```
## Step 3: Install Bazelisk (Bazel Wrapper)
Download Bazelisk and place it in your system's PATH to enable its use as the bazel command.
```
curl -L https://github.com/bazelbuild/bazelisk/releases/download/v1.17.0/bazelisk-linux-amd64 -o bazelisk
chmod +x bazelisk
sudo mv bazelisk /usr/local/bin/bazel
```
## Step 4: Compile ONOS
Utilize the ONOS build script to compile the project.
```
tools/build/onos-buck build onos
```
## Step 5: Execute ONOS
Initiate the ONOS controller within your terminal. This terminal session must remain active.
```
bazel run onos-local
```
ONOS will then start, listening on port 8101 (Karaf SSH) and port 6653 (OpenFlow).

## Step 6: Access the ONOS CLI
⚠️ If ONOS was executed using bazel run onos-local, you must connect via SSH to port 8101.
```
ssh karaf@localhost -p 8101
```
You will be prompted to accept the SSH key and provide a password. The default password is karaf.

## Step 7: Verify and Activate the OpenFlow App
Once inside the CLI, activate the OpenFlow application and confirm its operational status.

At the karaf@root > prompt:
```
app activate org.onosproject.openflow
```
Confirm the status of all applications:
```
apps -s
org.onosproject.openflow should be listed as ACTIVE.
```
## Step 8: Verify ONOS is Listening on OpenFlow Port (6653)
From a new terminal session (without closing the one running ONOS), verify that port 6653 is in a listening state.
```
sudo netstat -tulnp | grep 6653
```
Expected output:
```
tcp6        0      0 :::6653      :::* LISTEN       xxxx/java
```
---
## EXTRA: Keep ONOS Applications Always Active on Startup
## 1️⃣ Create the Application List
Based on your current configuration, these are the ONOS applications you want to start automatically:
```
org.onosproject.drivers.optical
org.onosproject.openflow-message
org.onosproject.drivers
org.onosproject.config
org.onosproject.drivers.ovsdb
org.onosproject.hostprovider
org.onosproject.lldpprovider
org.onosproject.gui2
org.onosproject.ovsdb-base
org.onosproject.ovsdb
org.onosproject.ovsdbhostprovider
org.onosproject.ofagent
org.onosproject.openflow-base
org.onosproject.openflow
org.onosproject.workflow.ofoverlay
org.onosproject.optical-model
org.onosproject.drivers.polatis.openflow
org.onosproject.fwd
org.onosproject.tunnel
org.onosproject.virtual
org.onosproject.workflow
org.onosproject.yang
```
ONOS requires them in a single line, separated by commas, with no spaces:
```
org.onosproject.drivers.optical,org.onosproject.openflow-message,org.onosproject.drivers,org.onosproject.config,org.onosproject.drivers.ovsdb,org.onosproject.hostprovider,org.onosproject.lldpprovider,org.onosproject.gui2,org.onosproject.ovsdb-base,org.onosproject.ovsdb,org.onosproject.ovsdbhostprovider,org.onosproject.ofagent,org.onosproject.openflow-base,org.onosproject.openflow,org.onosproject.workflow.ofoverlay,org.onosproject.optical-model,org.onosproject.drivers.polatis.openflow,org.onosproject.fwd,org.onosproject.tunnel,org.onosproject.virtual,org.onosproject.workflow,org.onosproject.yang
```
## 2️⃣ Create a Permanent Alias
This alias will let you start ONOS with all these apps enabled using a single command.
Edit your shell configuration file:
```
nano ~/.bashrc
```
Add the following line at the end of the file:
```
alias start-onos='bazel run onos-local -- --apps=org.onosproject.drivers.optical,org.onosproject.openflow-message,org.onosproject.drivers,org.onosproject.config,org.onosproject.drivers.ovsdb,org.onosproject.hostprovider,org.onosproject.lldpprovider,org.onosproject.gui2,org.onosproject.ovsdb-base,org.onosproject.ovsdb,org.onosproject.ovsdbhostprovider,org.onosproject.ofagent,org.onosproject.openflow-base,org.onosproject.openflow,org.onosproject.workflow.ofoverlay,org.onosproject.optical-model,org.onosproject.drivers.polatis.openflow,org.onosproject.fwd,org.onosproject.tunnel,org.onosproject.virtual,org.onosproject.workflow,org.onosproject.yang'
```
Save and exit (Ctrl+X, then Y, then Enter).

## 3️⃣ Apply the Changes and Start ONOS
Reload your shell configuration:
```
source ~/.bashrc
```
Start ONOS with all applications enabled:
```
start-onos
```
📌 Benefit:
With this alias, ONOS will always start with the same set of applications, without typing the full list every time.
