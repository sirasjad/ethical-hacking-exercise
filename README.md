# Ethical Hacking Exercise
Course: IN5290 - Ethical Hacking (UiO) \
Semester: Autumn 2021

## Objective
The goal of this exercise is to gain root access to a Linux server with no prior knowledge of existing security configurations and no prior access to the server. The exercise is complete when the contents of `/root/flag.txt` is read. 

## Solution
### Step 1 - Prepare virtual machines
The host machine is running an instance of Kali Linux on a virtual machine and the target/victim is a Ubuntu server, which is also running on a separate virtual machine on the same network as host. 

![](https://user-images.githubusercontent.com/8083228/138131002-ba2368a6-bd5d-4f2e-82a8-edb4458b2405.png)

### Step 2 - Identify attack surfaces on target machine
I started the exercise by identifying my own IP address. The host and target machines are running on a closed network, so it should be easy to identify the target IP address. Simply running `ifconfig` shows that my local IP address is `192.168.99.100`. 

![](https://user-images.githubusercontent.com/8083228/138134863-ae69e3fb-ebf7-457c-84dd-177c4f0306ef.png)

The next step is to identify the target IP address. One approach is to scan the network range using `nmap`, which discovers all connected hosts and services on the network. Now I know that the target IP address is `192.168.99.101`. 

![](https://user-images.githubusercontent.com/8083228/138136615-0c6ce3c2-bca2-469f-9b1a-388d1744a293.png)

I also initiated a port scan on the target IP address in order to identify open network ports. Using `nmap` for port scanning, I know that there are three open ports on the target machine: `80` (http), `21` (ftp) and `22` (ssh). By making a wild guess I can assume that the target is a webserver. 

![](https://user-images.githubusercontent.com/8083228/138137931-eed274b6-479e-4f08-b8e3-5003ede367ab.png)

Following my instincts, I tried to access the IP address from a web browser to check if port 80 is used by a webpage (utilizing http). At this point I had confirmed that the target is a web server and it is exposing a web page on port 80. The web page was simple and it had no functionality, and I did not find any hidden clues at first glance. 

![](https://user-images.githubusercontent.com/8083228/138142857-e8fc789f-e04e-4d3c-a9f1-2fc1db6e7db8.png)

Since the index page had no actual purpose (it only displayed an image), I naturally expect some hidden clues behind the scenes. Using `dirb`, which is a penetration tool used for web content scanning, I was able to scan the web server for sub-directories and hidden files. I found two sub-directories: `/server-status` and `/review/index.php`. 

![](https://user-images.githubusercontent.com/8083228/138144585-5073f649-90a6-416e-81eb-3ebc50c04d93.png)

The first sub-directory did not work and I was unable to access this page. The second sub-directory worked fine and it appears to be an interactive review forum for this Cyberpunk video game. (I obviously had some fun injecting garbage values to identify potential attack surfaces before this screenshot, hence the random entries on this page).

![](https://user-images.githubusercontent.com/8083228/138146444-3c20fcf8-3c32-4f18-b09c-77d0157127af.png)

Playing around with the different web pages, I noticed that the `viewcomment.php` page is using POST to request and retrieve information from an external source, which most likely is a database or an API. 

![](https://user-images.githubusercontent.com/8083228/138147468-59b69976-44cd-4bcb-96f2-4977c626d055.png)

Then I launched Burpsuite for further inspection of this web application and to identify specific attack surfaces by URL tampering and attempting SQL injection. I had to enable manual proxy in my web browser in order to forward all traffic to Burpsuite. 

![](https://user-images.githubusercontent.com/8083228/138149469-a2b32e96-3d65-4c39-a214-f0b1c69d0d6a.png)

Proceeding from the `Proxy` tab in Burpsuite, I enabled `Intercept mode` to inspect the network traffic and found a POST `id` attribute, which is forwarded to the SQL database. By sending this message to the `Repeater` in Burpsuite, I was able to simulate this traffic and play around with the `id` attribute to examine how the web server reacts to my random entries. 

![](https://user-images.githubusercontent.com/8083228/138150817-805ef730-7988-4ad5-82f8-160369c54b45.png)

I was able to change the `id` body parameter with different values and retrieve different information from the database. At this point I had confirmed at least one potential attack surface and I was confident that the POST attribute could be exploited using various attack vectors (such as SQL injection), and that is exactly what I proceeded with.

![](https://user-images.githubusercontent.com/8083228/138151419-8cd8ebc0-f33c-4206-ae03-033e5302633e.png)

### Step 3 - Initiate a SQL injection
A simple SQL injection trick is to append `OR 1=1` to the POST attribute and see how the database handles this parameter. The web server returned all rows from the database table and displayed all this information, instead of just one row related to one specific ID. I just found a major vulnerability in the web application and this was my entrance ticket to exploit the web server. At this point I had confirmed that the SQL implementation was insecure and it could be exploited by SQL injections. 

![](https://user-images.githubusercontent.com/8083228/138153174-ad89f170-bd8e-4746-84c1-6d520f10b4f0.png)

Then I disabled `Intercept mode` in Burpsuite and switched to SQLMap, which automates the process of SQL injection by tampering the input attributes with a pre-defined payload existing of invalid and malicious characters. I exported the raw POST request to a file and imported this dump file in SQLMap. By simply running `sqlmap -r export.req -- dump` I was able to initiate a SQL injection attack and the penetration tool successfully exploited and retrieved data from the database. The results were exported to a log file and the SQL injection process was complete.

![](https://user-images.githubusercontent.com/8083228/138155830-a62f810a-84e7-47d8-9b4d-3e7171df9103.png)

I found a SQL table consisting of FTP login information and this was my second entrance ticket to gain shell access. 

![](https://user-images.githubusercontent.com/8083228/138156552-b5fe987b-81ce-4838-baea-ee8f1477341c.png)

### Step 4 - FTP login
I tried to login using FTP with the given username `johnny` and password `s1lverh4nd`, and surprisingly it worked. At this point I had access to the server files through FTP and I was able to navigate around. I downloaded the server files to my local machine as a backup using `wget`, and then I disconnected from the server. 

![](https://user-images.githubusercontent.com/8083228/138161326-6729b53c-a867-4425-8040-d28ab7e1db91.png)

At this point I had a full backup of the server files on my local machine and I could safely navigate around without worrying about leaving any traces or activity logs on the server.

![](https://user-images.githubusercontent.com/8083228/138162483-abac8770-ee15-4aa7-8d4e-62b299848a31.png)

Then I examined the different directories and found a `.ssh` directory consisting of some SSH configuration files, including a private RSA encryption key! This was a major step to actually break into the server, as this private key is used for authentication and secure login to the server. 

![](https://user-images.githubusercontent.com/8083228/138163276-78be8dd5-def1-4565-9c1e-5d8e5149a017.png)

I examined the `authorized_keys` file to see which user the private key belongs to, and at this point I had everything I needed to actually log into the server (or at least attempt to login). 

![](https://user-images.githubusercontent.com/8083228/138164296-44edf054-4b10-478b-b541-7c29210fd61e.png)

### Step 5 - SSH login
The next step was to actually log into the server using SSH, and this was possible by using the private SSH key stored in `id_rsa`. I also had to change the file permissions using `chmod 600 id_rsa`, since the server was warning me about an unprotected private key file. After setting the file permissions, I was able to  log into the server and I officially had access to a shell! 

![](https://user-images.githubusercontent.com/8083228/138165087-824b5e52-b80a-455b-b4e7-2c86daa5f3b6.png)

### Step 6 - Getting root access
Even though I had shell access at this point, I did not have root permissions on the server (which is the goal of this exercise). The next step was obviously to get root permissions, so that I could access all files and initiate all root commands on the server. I tried to read the `/etc/sudoers` file, but did not have sufficient permissions to access this file.

![](https://user-images.githubusercontent.com/8083228/138166786-89c7b683-1851-410a-b70d-71d1c0a31d6a.png)

The current user permissions were limited and I was unable to use every `sudo` command I tried. However in some cases there are some specific sudo commands that are enabled for non-root users, and this was something I had to investigate. I used the commands `find / -perm -u=s -type f 2>/dev/null` and `sudo -l` to identify which sudo commands my user account was able to run as a non-root user. 

![](https://user-images.githubusercontent.com/8083228/138168178-c21d40a0-9e60-403d-a238-5a6ac302398b.png)

Bingo! I noticed that my user account could run `tedit` with root privileges and this was my entrance ticket to the `/etc/sudoers` file. Tedit is a text editor (alternative to `vim` and `nano`) and it was really everything I needed in order to give myself root privileges on the server.

![](https://user-images.githubusercontent.com/8083228/138169116-3140f94f-5d77-40ba-8f41-410412f2780e.png)

I opened the `/etc/sudoers` file and changed my user permissions to match the root user account (the commented line shows my old user privileges). Since I had permissions to run `tedit` as sudo, I was also allowed to save the file as sudo (even though my user account did not have permissions to read the sudoers file in the first place). 

![](https://user-images.githubusercontent.com/8083228/138170411-6a8b7f83-a026-49cb-b2ee-c4d078e1c43d.png)

And that's it! At this point I officially had root shell access on the server and I confirmed this by running `sudo su`. 

![](https://user-images.githubusercontent.com/8083228/138170972-9da27d05-9725-4e42-b4e7-27b6c4c5bbb7.png)

### Step 7 - Reading the /root/flag.txt file
Finally it was time to complete the last step of this exercise: access the `/root` directory and read the `flag.txt` file. I was able to complete this step without any issues, as I already had root shell privileges at this point. 

![](https://user-images.githubusercontent.com/8083228/138171400-f1f640de-56ee-4448-b751-3a2f92d1a79d.png)

The file content is: `FLAG{ju5t_4_s1mul4tion_AnyW4y}` 

Thanks for this awesome exercise. It was really fun! :)
