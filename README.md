# Proxy-Cloud
Simple instructions on how to create a proxy server on the Cloud VM (GCP)



#### - Create a Google Cloud account, and create a new project (Google giving 90 days of free Cloud usage, so at least three months of free proxy server) 


#### - Go to the dashboard and select "Compute Engine" in the Resources panel
<img width="462" alt="Screenshot 2023-03-10 at 4 01 13 pm" src="https://user-images.githubusercontent.com/71220725/224227735-da81e37b-0fef-42cd-a01d-afdb0a8e838a.png">


#### - Click "Enable" and wait until everything is set up, The compute engine API is enabled but there are no instances so click "Create instance" to add a new instance.


#### - Set up your Vm: Select preferred region,  N1 instance will be okay for this, Allow HTTP and HTTPS traffic. Click 'Create' and wait a few minutes for the instance to be created


<img width="598" alt="Screenshot 2023-03-10 at 4 28 20 pm" src="https://user-images.githubusercontent.com/71220725/224231305-86f04522-7d06-4316-8b46-8e767d45800f.png">
<img width="598" alt="Screenshot 2023-03-10 at 4 28 43 pm" src="https://user-images.githubusercontent.com/71220725/224231360-c431c73e-ff44-41f0-b9e8-9b5154760140.png">


#### - After that hover over SSH and select 'Open in Browser window', make sure that your browser is not blocking pop-up windows
 
 You will have access to the shell of your VM,
<img width="1602" alt="Screenshot 2023-03-10 at 4 35 06 pm" src="https://user-images.githubusercontent.com/71220725/224232225-ff173a40-833f-40d7-86c1-f264e1d97732.png">

  - In shell run ```sudo apt-get update``` to update packages, 
  - ```sudo apt-get install squid``` to instal [Squid](http://www.squid-cache.org), 
  - After downloading squid, start it running ```sudo systemctl start squid```,
  - And enable it running ```sudo systemctl enable squid``` 
<img width="1603" alt="Screenshot 2023-03-10 at 4 39 35 pm" src="https://user-images.githubusercontent.com/71220725/224232797-fe877047-47c6-48e4-b05f-ed15261b288e.png">
 - To check if it is running execute ```systemctl status squid```, and it should say that it's active 
<img width="1603" alt="Screenshot 2023-03-10 at 4 45 26 pm" src="https://user-images.githubusercontent.com/71220725/224233594-0af77de5-1fe4-4d3a-a71e-a0843a354784.png">

#### - Configure proxy 

  - Add your IP to allowed connections: run ```sudo vim /etc/squid/squid.conf```, then type ```/localhost```(search command) most likely it will head you to list of allowed IP adresses where you need to add IP adress that you want to use to connect to proxy in this format : ```acl localnet src 000.000.000.0/24```( replace it with your IP
  <img width="1603" alt="Screenshot 2023-03-10 at 4 56 50 pm" src="https://user-images.githubusercontent.com/71220725/224235254-7c1ea641-b812-4dd0-b845-a19027816559.png">
  most likely it will head you to list of allowed IP adresses where you need to add IP adress that you want to use to connect to proxy in this format : ```acl localnet src 000.000.000.0/24``` ( replace it with your IP)
 
 - Allow all HTTP traffic, in same vim /etc/squid/squid.conf search for ```/http_traffic deny all``` and change it to ```allow all```.
 
 - That's it, you can quit vim by executing ```:wx```
  
  <img width="1603" alt="Screenshot 2023-03-10 at 5 04 10 pm" src="https://user-images.githubusercontent.com/71220725/224236364-3f7afc0f-7635-4df6-9878-947a931db56b.png">

 - Restart server after changes ```sudo systemctl restart squid```.
 
 - Now you need to save your VM IP( you can get it on your VM instances page on GCP console) and port on which squid is running (its 3128 by default)
 
 #### - Go to Compute engine dashboard in GCP and hover over 3 dots at your instance row, select 'view network details'
 
 <img width="293" alt="Screenshot 2023-03-10 at 5 45 10 pm" src="https://user-images.githubusercontent.com/71220725/224243322-d133da52-20ef-4334-b1a1-51dfcb764fac.png">
 
 You will be redirected to the VPC networks page, select Firewall, and click 'Create firewall rule'.
 <img width="1603" alt="Screenshot 2023-03-10 at 5 44 20 pm" src="https://user-images.githubusercontent.com/71220725/224243163-0574fb53-f6e5-49c0-aed0-422683aacc34.png">
 
 
Then choose a name for your rule and adjust everything as per this screenshot, in IP4 ranges add your IP 
<img width="700" alt="Screenshot 2023-03-10 at 5 49 08 pm" src="https://user-images.githubusercontent.com/71220725/224243991-50305905-0f0d-41d6-96e6-dc1e8434a61f.png">

Then you can test your proxy server on your machine, by running ```curl -x http://<VM external IP>:3128  -I https://google.com ```

<img width="567" alt="Screenshot 2023-03-10 at 7 07 27 pm" src="https://user-images.githubusercontent.com/71220725/224259874-84125b5d-a8ff-4961-ba82-aa1efad91e82.png">


If you encounter errors, try to restart squid on your VM, check the config file, check the Firewall rules


#### - Blocking websites 

You can add websites to the 'not allowed list', to do this:
 - Run ```sudo vim /etc/squid/blocked_sites```
 - Add the websites to be blocked in the file
 - Open squid config ```sudo vi /etc/squid/squid.conf```
 - Add the following to the ACL list: 
 ```
 acl blocked_sites dstdomain "/etc/squid/blocked_sites"
 http_access deny blocked_sites
 ```
 - Restart squid
