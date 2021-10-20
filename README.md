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

Since the index page had no actual purpose (it only displayed an image), I naturally expect some hidden clues behind the scenes. Using `dirb`, which is a penetration tool used for web content scanning, I was able to scan the web server for sub-directories and hidden files. I found two sub-directories: `/server-status` and `/review/index.php`. 

![image](https://user-images.githubusercontent.com/8083228/138144585-5073f649-90a6-416e-81eb-3ebc50c04d93.png)

The first sub-directory did not work and I was unable to access this page. The second sub-directory worked fine and it appears to be an interactive review forum for this Cyberpunk video game. (I obviously had some fun injecting garbage values to identify potential attack surfaces before this screenshot, hence the random entries on this page).

![image](https://user-images.githubusercontent.com/8083228/138146444-3c20fcf8-3c32-4f18-b09c-77d0157127af.png)

Playing around with the different web pages, I noticed that the `viewcomment.php` page is using POST to request and retrieve information from an external source, which most likely is a database or an API. 

![image](https://user-images.githubusercontent.com/8083228/138147468-59b69976-44cd-4bcb-96f2-4977c626d055.png)

Then I launched Burpsuite for further inspection of this web application and to identify specific attack surfaces by URL tampering and attempting SQL injection. I had to enable manual proxy in my web browser in order to forward all traffic to Burpsuite. 

![image](https://user-images.githubusercontent.com/8083228/138149469-a2b32e96-3d65-4c39-a214-f0b1c69d0d6a.png)

Proceeding from the `Proxy` tab in Burpsuite, I enabled `Intercept mode` to inspect the network traffic and found a POST `id` attribute, which is forwarded to the SQL database. By sending this message to the `Repeater` in Burpsuite, I was able to simulate this traffic and play around with the `id` attribute to examine how the web server reacts to my random entries. 

![image](https://user-images.githubusercontent.com/8083228/138150817-805ef730-7988-4ad5-82f8-160369c54b45.png)

I was able to change the `id` body parameter with different values and retrieve different information from the database. At this point I was confident that the POST attribute could be exploited using various attack vectors (such as SQL injection), and that is exactly what I proceeded with.

![image](https://user-images.githubusercontent.com/8083228/138151419-8cd8ebc0-f33c-4206-ae03-033e5302633e.png)


At this point I had confirmed at least one attack surface.

### Step 3 - Initiate a SQL injection
