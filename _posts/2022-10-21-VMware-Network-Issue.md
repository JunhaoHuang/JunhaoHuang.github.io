---
layout: post
title: VMware Virtual Machine cannot connect to the Internet 
subtitle: VMware virtual machine network issue
gh-repo: daattali/beautiful-jekyll
gh-badge: [star, fork, follow]
tags: [test]
comments: true
---

Recently, I accidently shut down my virtual machine: Ubuntu 22.04. After restart the virtual machine, the network icon disappeared and the machine could not connect to the Internet. After spending several painful day searching everything related in the Internet, I finally found two posts related to this issue and it helped to solve my problem. This problem can be solved by combining the ideas from the following two posts: [1](<https://blog.csdn.net/qhy6518338/article/details/104694026>) and [2](<https://askubuntu.com/questions/1371275/where-has-the-network-manager-service-in-21-10-gone>).

It seems that we need to restart the **Network manager** of the linux. To do this, I first tried the steps in [1](<https://blog.csdn.net/qhy6518338/article/details/104694026>). However, the command line showed that the Network-manager could not found. It turns out that the NetworkManager service has been named NetworkManager for a [while now](https://packages.ubuntu.com/impish/amd64/network-manager/filelist) according to [2](<https://askubuntu.com/questions/1371275/where-has-the-network-manager-service-in-21-10-gone>). 

By combining these two posts, I was able to fixed it by:

```
sudo service NetworkManager stop
sudo rm /var/lib/NetworkManager/NetworkManager.state
sudo gedit /etc/NetworkManager/NetworkManager.conf
```
Then, change
```
managed = true
```
And finally,
```
sudo service NetworkManager start
reboot
```

Just record this in case it still happens in the future!