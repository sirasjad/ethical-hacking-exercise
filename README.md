# Ethical Hacking Exercise
Course: IN5290 - Ethical Hacking (UiO) \
Semester: Autumn 2021

## Objective
The goal of this exercise is to gain root access to a Linux server with no prior knowledge of existing security configurations and no prior access to the server. The exercise is complete when the contents of `/root/flag.txt` is read. 

## Solution
### Step 1 - Prepare virtual machines
The host machine is running an instance of Kali Linux on a virtual machine and the target/victim is a Ubuntu server, which is also running on a separate virtual machine on the same network as host. 

![VirtualBox environment](https://user-images.githubusercontent.com/8083228/138131002-ba2368a6-bd5d-4f2e-82a8-edb4458b2405.png)

### Step 2 - Identify attack surfaces on target machine
I started the exercise by identifying my own IP address. The host and target machines are running on a closed network, so it should be easy to identify the target IP address. Simply running `ifconfig` shows that my local IP address is `192.168.99.100`. 

![Identifying my own IP address](https://user-images.githubusercontent.com/8083228/138134863-ae69e3fb-ebf7-457c-84dd-177c4f0306ef.png)

The next step is to identify the target IP address. One approach is to scan the network range using `nmap`, which discovers all connected hosts and services on the network. Now I know that the target IP address is `192.168.99.101`. 

![Identifying target IP address](https://user-images.githubusercontent.com/8083228/138136615-0c6ce3c2-bca2-469f-9b1a-388d1744a293.png)

I also initiated a port scan on the target IP address in order to identify open network ports. Using `nmap` for port scanning, I know that there are three open ports on the target machine: `80` (http), `21` (ftp) and `22` (ssh). By making a wild guess I can assume that the target is a webserver. 

![Port scanning](https://user-images.githubusercontent.com/8083228/138137931-eed274b6-479e-4f08-b8e3-5003ede367ab.png)

Following my instincts, I tried to access the IP address from a web browser to check if port 80 is used by a webpage (utilizing http). At this point I had confirmed that the target is a web server and it is exposing a web page on port 80. The web page was simple and it had no functionality, and I did not find any hidden clues at first glance. 

![image](https://user-images.githubusercontent.com/8083228/138142857-e8fc789f-e04e-4d3c-a9f1-2fc1db6e7db8.png)

