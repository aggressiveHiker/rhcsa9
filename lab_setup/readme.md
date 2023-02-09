# Lab Setup

The lab consists of three servers, all RHEL 9.0 minimal installations, which consist of one repo server and two exam servers. Once setup, the repo server should be left untouched, except for running a quick permissions reset on the NFS shares when resetting the lab.

Servers in my lab setup are on the following IPs:
- Repo: 192.168.55.47
- Server1: 192.168.55.71
- Server2: 192.168.55.72

Only the repo server is configured with an IP up front. The other two servers are configured during the tasks.

Once the exam servers are setup, a VM snapshot should be taken. After running through the tasks in the task list, the VM snapshots can be rolled back to run through it all again.