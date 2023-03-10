# Proxy-Cloud
Simple instruction how to create proxy server on Cloud VM (GCP)



#### - Create Google Cloud account , and create new project (Google giving 90 days of free Cloud usage, so at least three months of free proxy server) 


#### - Go to dashboard and select "Compute Engine" in Resources panel
<img width="462" alt="Screenshot 2023-03-10 at 4 01 13 pm" src="https://user-images.githubusercontent.com/71220725/224227735-da81e37b-0fef-42cd-a01d-afdb0a8e838a.png">


#### - Click "Enable" and wait until everything will be setted up, now compute engine API enabled but there is no instances so click "Create instance" to add new instance.


#### - Set up your Vm: Select prefered region,  N1 instance will be okay for this, Allow HTTP and HTTPS traffic. Click 'Create' and wait few minutes for instance to be created


<img width="598" alt="Screenshot 2023-03-10 at 4 28 20 pm" src="https://user-images.githubusercontent.com/71220725/224231305-86f04522-7d06-4316-8b46-8e767d45800f.png">
<img width="598" alt="Screenshot 2023-03-10 at 4 28 43 pm" src="https://user-images.githubusercontent.com/71220725/224231360-c431c73e-ff44-41f0-b9e8-9b5154760140.png">


#### - After that hover over SSH and select 'Open in Browser window', make shure that your browser not blocking pop-up windows
 
 You will have access to shell of your VM,
<img width="1602" alt="Screenshot 2023-03-10 at 4 35 06 pm" src="https://user-images.githubusercontent.com/71220725/224232225-ff173a40-833f-40d7-86c1-f264e1d97732.png">

  - In shell run ```sudo apt-get update``` to update packages, 
  - ```sudo apt-get install squid``` to instal [Squid](http://www.squid-cache.org), 
  - After downloading squid, start it running ```sudo systemctl start squid```,
  - And enable it running ```sudo systemctl enable squid``` 
<img width="1603" alt="Screenshot 2023-03-10 at 4 39 35 pm" src="https://user-images.githubusercontent.com/71220725/224232797-fe877047-47c6-48e4-b05f-ed15261b288e.png">
 - To check if its runningg execeute ```systemctl status squid ```, and it shoud say that its active 
<img width="1603" alt="Screenshot 2023-03-10 at 4 45 26 pm" src="https://user-images.githubusercontent.com/71220725/224233594-0af77de5-1fe4-4d3a-a71e-a0843a354784.png">

#### - Configure proxy 

  - Add your IP to allowed connections: run ```sudo vim /etc/squid/squid.conf```, then type ```/localhost``` most likely it will head you to list of allowed IP adresses where you need to add IP adress that you want to use to connect to proxy in this format : ```acl localnet src 000.000.000.0/24```( replace it with your IP
  <img width="1603" alt="Screenshot 2023-03-10 at 4 56 50 pm" src="https://user-images.githubusercontent.com/71220725/224235254-7c1ea641-b812-4dd0-b845-a19027816559.png">
  most likely it will head you to list of allowed IP adresses where you need to add IP adress that you want to use to connect to proxy in this format : ```acl localnet src 000.000.000.0/24``` ( replace it with your IP)
  
  <img width="1603" alt="Screenshot 2023-03-10 at 5 04 10 pm" src="https://user-images.githubusercontent.com/71220725/224236364-3f7afc0f-7635-4df6-9878-947a931db56b.png">

 - Restart server after changes ```sudo systemctl restart squid```.
 
 - Now you need to save your VM IP( you can get it on your VM instances page on GCP console) and port on which squid is running (its 3128 by default)
 
 #### - Go to Compute engine dashboard in GCP and hover over 3 dots at your instance row, select 'view network details'
 
 <img width="293" alt="Screenshot 2023-03-10 at 5 45 10 pm" src="https://user-images.githubusercontent.com/71220725/224243322-d133da52-20ef-4334-b1a1-51dfcb764fac.png">
 
 You will be redirected to VPC networks page, select Firewall, and click 'Create firewall rule'.
 <img width="1603" alt="Screenshot 2023-03-10 at 5 44 20 pm" src="https://user-images.githubusercontent.com/71220725/224243163-0574fb53-f6e5-49c0-aed0-422683aacc34.png">
 
 
Than chose a name for your rule and adjust everything as per this screenshot, in IP4 ranges add your IP 
<img width="700" alt="Screenshot 2023-03-10 at 5 49 08 pm" src="https://user-images.githubusercontent.com/71220725/224243991-50305905-0f0d-41d6-96e6-dc1e8434a61f.png">

Than you can test your proxy server on your machine, by running ```curl -x http://<VM external IP>:3128  -I http://google.com ```
<img width="703" alt="Screenshot 2023-03-10 at 6 18 38 pm" src="https://user-images.githubusercontent.com/71220725/224249318-db044b92-71f1-43e2-a11c-ae1c54d8ef7d.png">

If you encounter errors , try to restart squid on your VM, check config file, check Firewall rules


#### - Blocking websites 

You can add websites to 'not allowed list', to do this:
 - Run ```sudo vim /etc/squid/blocked_sites```
 - Add the websites to be blocked in the file
 - Open squid config ```sudo vi /etc/squid/squid.conf```
 - Add the following to the ACL list: 
 ```
 acl blocked_sites dstdomain "/etc/squid/blocked_sites"
 http_access deny blocked_sites
 ```
 - Restart squid
