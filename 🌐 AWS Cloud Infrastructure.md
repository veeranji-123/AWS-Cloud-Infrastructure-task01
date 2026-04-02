#                                     **🌐 AWS Cloud Infrastructure**

​                                                                        **Step-by-Step Project Documentation**

​                                                                           

   

**Task 1: Highly Available 3-Tier Web Application on AWS**

 

| **👤 Author**  Veeranji | **📅 Date**  30 March 2026 | **🌍 Region**  us-east-1 | **⭐ Level**  Advanced |
| ---------------------- | ------------------------- | ----------------------- | --------------------- |
|                        |                           |                         |                       |

 

# What is This Project?

  **💡 What is this?**     Imagine you own an online shop (like Amazon or Flipkart).  Thousands of people visit your website every day — and during a big sale,  that number jumps to millions! If your server crashes, you lose money and  customers. This project solves that problem by building a website that NEVER  goes down, automatically grows when traffic increases, and shrinks back when  traffic is low — all on Amazon's cloud (AWS).  

 

## The Problem We Are Solving

•    A normal website runs on ONE computer. If that computer crashes → website is down.

•    During big sales, one computer cannot handle millions of visitors.

•    You pay for the big computer even when only 10 people are visiting.

 

## Our Solution

•    Run the website on MULTIPLE computers across different locations (Availability Zones).

•    A "Smart Traffic Director" (Load Balancer) sends visitors to whichever computer is free.

•    When traffic goes up → automatically add more computers. When traffic goes down → remove them.

•    You only pay for what you use!

 

## Services We Used (Simple Explanation)

| **VPC  (Virtual Private Cloud)** | Your own  private section of Amazon's cloud — like renting a floor in a building |
| -------------------------------- | ------------------------------------------------------------ |
| **Subnets**                      | Rooms inside  your floor — some public (anyone can enter), some private (staff only) |
| **Internet  Gateway**            | The main door  of your building — connects your floor to the internet |
| **Route  Tables**                | The directory  board in the lobby — tells traffic where to go |
| **EC2  Instances**               | The actual  computers (servers) that run your website        |
| **EBS Volume**                   | The hard disk  attached to each computer — stores your website data |
| **AMI (Golden  AMI)**            | A perfect  "photo" of your computer setup — use it to quickly make identical  copies |
| **ALB (Load  Balancer)**         | The  receptionist who divides visitors equally among all your computers |
| **Target  Group**                | The list of  computers the receptionist can send visitors to |
| **Auto  Scaling Group**          | The manager  who adds/removes computers based on how busy things are |
| **CloudWatch  Alarms**           | The smoke  detector — alerts the manager when CPU is too high or too low |
| **S3 Bucket**                    | Online storage  for photos, CSS files, and other static content on your website |

 

 

# How Everything Connects — Architecture Overview

Here is the flow of how a visitor accesses your website:

 

  👤 VISITOR  opens website in browser  ↓  🌐   INTERNET GATEWAY   (my-e-commers-ig)  ↓  **⚖️ APPLICATION  LOAD BALANCER (my-ecommers-apl)**  *Distributes traffic equally to healthy  servers*  ↙   ↘  🖥️ EC2 Server (AZ-1a)     🖥️ EC2 Server (AZ-1b)  us-east-1a 10.0.1.0/24      us-east-1b 10.0.2.0/24  ↕   ↕  📊 AUTO  SCALING GROUP (Min=2 Max=6)  *Adds more servers when busy, removes  when quiet*  

 

 

| **STEP 1** | **Creating the  VPC — Your Private Space on AWS** |
| ---------- | ------------------------------------------------- |
|            |                                                   |

 

  **💡 What is this?**     Think of AWS as a giant apartment building with thousands of  floors. A VPC (Virtual Private Cloud) is YOUR private floor that nobody else  can enter. Everything we build — servers, databases, load balancers — lives  inside this VPC. We named ours my-e-Commers-vpc and gave it the address range  10.0.0.0/16 (this means we can have up to 65,536 devices inside).  

 

## What We Did

1. Went to AWS Console → VPC → Your VPCs
2. Clicked 'Create VPC'
3. Named it: my-e-Commers-vpc
4. Set CIDR block: 10.0.0.0/16
5. Clicked Create — got success message!

 

## Settings Used

| **VPC Name**                 | my-e-Commers-vpc                           |
| ---------------------------- | ------------------------------------------ |
| **VPC ID  (auto-generated)** | vpc-09f7c6b46a291696c                      |
| **CIDR Block**               | 10.0.0.0/16 (supports 65,536 IP addresses) |
| **State**                    | Available ✓                                |
| **Region**                   | us-east-1 (N. Virginia, USA)               |

 

  **✅ VPC created successfully! Our private  network is ready.**  

 

  📸  **SCREENSHOT 1**  *VPC Created — showing my-e-Commers-vpc with Status:  Available*  

![](Screenshot%202026-03-30%20120229.png)  

 

 

| **STEP 2** | **Creating  Subnets — Dividing Our Space into Rooms** |
| ---------- | ----------------------------------------------------- |
|            |                                                       |

 

  **💡 What is this?**     Inside our VPC floor, we created 4 rooms called Subnets. • PUBLIC Subnets = The lobby and reception  area — internet traffic can reach here. This is where the Load Balancer  lives. • PRIVATE Subnets = The back office — no direct internet access. This  is where our actual web servers live (safer!). We created 2 public + 2 private subnets,  spread across 2 different locations (us-east-1a and us-east-1b). If one  location has a power cut, the other keeps running!  

 

## What We Did

6. Went to VPC → Subnets → Create Subnet
7. Created 4 subnets inside vpc-09f7c6b46a291696c
8. Spread them across 2 Availability Zones for safety

 

## Subnets Created

| **Subnet Name**   | **Type**   | **Address   Range** | **Location** |
| ----------------- | ---------- | ------------------- | ------------ |
| my-e-commers-pub1 | 🌐  PUBLIC  | 10.0.1.0/24         | us-east-1a   |
| my-e-commers-pub2 | 🌐  PUBLIC  | 10.0.2.0/24         | us-east-1b   |
| my-e-commers-pvt1 | 🔒  PRIVATE | 10.0.3.0/24         | us-east-1a   |
| my-e-commers-pvt2 | 🔒  PRIVATE | 10.0.4.0/24         | us-east-1b   |

 

  **✅ 4 subnets created across 2 Availability  Zones — high availability achieved!**  

 

  📸  **SCREENSHOT 2**  *Subnets Created — showing all 4 subnets: pub1, pub2,  pvt1, pvt2*  [ ![](Screenshot%202026-03-30%20121102.png) ]  

 

 

| **STEP 3** | **Setting Up  Route Tables — Teaching Traffic Where to Go** |
| ---------- | ----------------------------------------------------------- |
|            |                                                             |

 

  **💡 What is this?**     A Route Table is like a GPS or directory board inside your  building. It tells network traffic 'if you want to go to the internet, use  this door' or 'if you want to stay inside, use that corridor'. We created 2 route tables: • PUBLIC route  table → has a route to the Internet Gateway (so ALB can talk to the internet)  • PRIVATE route table → only knows about the internal VPC (so app servers  stay hidden from internet)  

 

## Public Route Table — my-pub-rt

We associated pub1 and pub2 subnets to this route table:

 

  📸  **SCREENSHOT 3**  *Edit Subnet Associations — public subnets (pub1 + pub2)  selected for public route table*  [ ![](Screenshot%202026-03-30%20121408.png) ]  

 

## Private Route Table — my-pvt-rt

We associated pvt1 and pvt2 subnets to this route table:

 

  📸  **SCREENSHOT 4**  *Edit Subnet Associations — private subnets (pvt1 +  pvt2) selected for private route table*  [![](Screenshot%202026-03-30%20121510.png) ]  

 

## Adding the Internet Route to Public Table

We added a route: 0.0.0.0/0 → Internet Gateway. This means: 'any traffic going to the internet must use our Internet Gateway door'.

 

  📸  **SCREENSHOT 5**  *Edit Routes — showing 0.0.0.0/0 pointing to  igw-0ffb973d65cf4895e (Internet Gateway)*  [ ![](Screenshot%202026-03-30%20121603.png) ]  

 

## Route Summary

| **Public RT  Name**     | my-pub-rt (rtb-0b9a7d6b079678fd9)                          |
| ----------------------- | ---------------------------------------------------------- |
| **Public RT  Route 1**  | 10.0.0.0/16 →  local (internal traffic stays inside)       |
| **Public RT  Route 2**  | 0.0.0.0/0 →  Internet Gateway (internet traffic  goes out) |
| **Public RT  Subnets**  | my-e-commers-pub1 +   my-e-commers-pub2                    |
| **Private RT  Name**    | my-pvt-rt (rtb-01e38407193d927ed)                          |
| **Private RT  Route**   | 10.0.0.0/16 →  local only (no internet access)             |
| **Private RT  Subnets** | my-e-commers-pvt1 +   my-e-commers-pvt2                    |

 

## VPC Overview — Everything Connected

  📸  **SCREENSHOT 6**  *VPC Map — showing all subnets in AZ-1a and AZ-1b,  connected to route tables and Internet Gateway*  [ ![](Screenshot%202026-03-30%20122234.png) ]  

 

  **✅ Route tables configured! Public subnets can  reach the internet. Private subnets are secured.**  

 

 

| **STEP 4** | **Launching an  EC2 Server — Your First Web Server** |
| ---------- | ---------------------------------------------------- |
|            |                                                      |

 

  **💡 What is this?**     EC2 (Elastic Compute Cloud) is simply a computer/server that  runs in Amazon's data center. We launched one server and used 'User Data' — a  startup script that automatically: 1. Installs Nginx (a web server software —  like Apache) 2. Downloads the website code from GitHub 3. Starts the web  server So the moment the computer  turns on, it is ALREADY a working web server — no manual setup needed!  

 

## Instance Configuration

| **Base Image  (AMI)** | ami-0ec10929233384c7f (Amazon Linux)            |
| --------------------- | ----------------------------------------------- |
| **Instance  Type**    | t2.micro (1 CPU, 1 GB RAM — Free Tier eligible) |
| **Firewall**          | New Security  Group (will allow HTTP traffic)   |
| **User Data**         | Automatic  setup script (runs on first boot)    |

 

## User Data Script — What It Does Automatically

This script runs automatically when the server starts. It sets up everything without us touching the server:

```bash
 !/bin/bash                  
 # Start  of script 
 apt-get update -y              
 # Update the  server 
 apt-get install -y nginx  git         
 # Install web  server + git  
 rm -rf  /var/www/html/*              
 # Clear default web files  
 git clone  https://github.com/Akiranred/pacman.git /var/www/html/                          
 # Download website code  
 chown -R www-data:www-data  /var/www/html/  
 # Set file  ownership  
 chmod -R 755  /var/www/html/          
 # Set  file permissions 
 systemctl restart  nginx           
 # Start  the web server!  

 
```

  📸  **SCREENSHOT 7**  *EC2 Launch — User Data script entered, Summary panel  showing t2.micro, Launch Instance button*  [ ![](Screenshot%202026-03-30%20124607.png) ]  

 

  **✅ EC2 server launched with automatic Nginx  setup via User Data script!**  

 

 

| **STEP 5** | **Attaching an  EBS Volume — Adding a Hard Disk to the Server** |
| ---------- | ------------------------------------------------------------ |
|            |                                                              |

 

  **💡 What is this?**     Every computer needs a hard disk to store data. In AWS, EBS  (Elastic Block Store) is the hard disk for your EC2 server. We  configured: • Size: 8 GB (enough for  our app) • Type: gp2 (General Purpose SSD — fast and reliable) • Encryption:  ON — so even if someone steals the disk, they cannot read the data • KMS Key:  AWS manages the encryption key for us automatically  

 

## EBS Volume Settings

| **Storage  Type**          | EBS (Elastic  Block Store)                        |
| -------------------------- | ------------------------------------------------- |
| **Device Name**            | /dev/sda1 (the main system disk)                  |
| **Volume Type**            | gp2 (General Purpose SSD)                         |
| **Size**                   | 8 GiB                                             |
| **IOPS**                   | 100 /  3000 (Input/Output operations per  second) |
| **Encryption**             | ENABLED (data is encrypted at rest)               |
| **KMS Key**                | aws/ebs (AWS default key — free)                  |
| **Delete on  Termination** | Yes (disk is removed when server is deleted)      |

 

  📸  **SCREENSHOT 9**  *EBS Volume Configuration — showing 8 GiB, gp2,  Encrypted, KMS key settings*  [ ![](Screenshot%202026-03-30%20130107.png)]  

 

  **✅ Encrypted EBS volume attached! Server data  is secure.**  

 

| **STEP 6** | **Creating a  Golden AMI — Saving the Perfect Server Setup** |
| ---------- | ------------------------------------------------------------ |
|            |                                                              |

 

  **💡 What is this?**     Once our server is set up perfectly (Nginx installed, website  running, EBS configured), we take a 'photograph' of it. This photograph is  called an AMI (Amazon Machine Image) — also called a Golden AMI. Why? Because when the Auto Scaling Group  needs to add more servers quickly during a traffic spike, instead of setting  up each one from scratch (which takes time), it just 'copies the photograph'  and creates an identical server in seconds!  

 

## Our Golden AMI

| **AMI Name**          | task1                             |
| --------------------- | --------------------------------- |
| **AMI ID**            | ami-0a643783e623443b4             |
| **Operating  System** | Linux/UNIX (x86_64)               |
| **Root Disk**         | /dev/sda1 (8 GiB, gp2, encrypted) |
| **Status**            | Available  (ready to use)         |
| **Created On**        | 30 March 2026                     |

 

  📸  **SCREENSHOT 8**  *AMI Created — showing AMI name 'task1',  ami-0a643783e623443b4, Status: Available*  [ ![](Screenshot%202026-03-30%20125654.png) ]  

 

  **✅ Golden AMI 'task1' saved! We can now create  identical servers in seconds.**  

 

 

| **STEP 7** | **Creating a  Target Group — The List of Servers for the Load Balancer** |
| ---------- | ------------------------------------------------------------ |
|            |                                                              |

 

  **💡 What is this?**     A Target Group is like a contact list that the Load Balancer  (receptionist) uses to know which servers (EC2 instances) to send visitors  to. The Load Balancer constantly  checks 'are you alive?' (health checks) to each server. If a server does not  respond, the Load Balancer stops sending visitors to it and only uses the  healthy ones.  

 

## Target Group Settings

| **Target  Group Name** | my-e-commers-tg                              |
| ---------------------- | -------------------------------------------- |
| **Target Type**        | Instance (we add EC2 servers to this list)   |
| **Protocol :  Port**   | HTTP : 80 (web traffic on port 80)           |
| **Protocol  Version**  | HTTP1                                        |
| **VPC**                | vpc-09f7c6b46a291696c (our VPC)              |
| **Health  Check**      | HTTP GET  / — checks if server is responding |

 

  📸  **SCREENSHOT 10**  *Target Group Created — my-e-commers-tg, showing Details  with HTTP:80, VPC linked*  [ ![](Screenshot%202026-03-30%20143316.png) ]  

 

  **✅ Target group created! Load balancer now has  a list to route traffic to.**  

 

| **STEP 8** | **Creating the  Application Load Balancer — The Smart Traffic Director** |
| ---------- | ------------------------------------------------------------ |
|            |                                                              |

 

  **💡 What is this?**     The Application Load Balancer (ALB) is the front door of your  website. When millions of visitors come, the ALB splits them equally across  all your servers so no single server gets overloaded. Key features we enabled: • Internet-facing:  visible to the public internet • Cross-Zone Load Balancing: sends traffic to  BOTH zones (us-east-1a AND us-east-1b) • Sticky Sessions: a visitor who adds  items to their shopping cart stays on the SAME server for 1 hour (so their  cart doesn't disappear!)  

 

## ALB Basic Settings

| **Load  Balancer Name**  | my-ecommers-apl                        |
| ------------------------ | -------------------------------------- |
| **Type**                 | Application  Load Balancer (ALB)       |
| **Scheme**               | Internet-facing (public can access it) |
| **IP Address  Type**     | IPv4                                   |
| **Availability  Zone 1** | us-east-1a →  my-e-commers-pub1 subnet |
| **Availability  Zone 2** | us-east-1b →  my-e-commers-pub2 subnet |

 

  📸  **SCREENSHOT 11**  *ALB Basic Configuration — showing name  'my-ecommers-apl', Internet-facing selected, IPv4*  [ ![](Screenshot%202026-03-30%20144403.png) ]  

 

## Listener & Sticky Session Setup

We configured how the ALB handles incoming traffic and enabled Sticky Sessions for shopping cart support:

| **Listener  Action**     | Forward to  Target Group: my-e-commers-tg           |
| ------------------------ | --------------------------------------------------- |
| **Weight**               | 1 (100% of traffic goes to our target group)        |
| **Sticky  Sessions**     | ENABLED (visitors stick to same server)             |
| **Stickiness  Duration** | 1 hour (3600  seconds) — shopping cart stays intact |
| **Cross-Zone  LB**       | Enabled —  traffic balanced across both AZs         |

 

  📸  **SCREENSHOT 12**  *ALB Listener — showing Forward to my-e-commers-tg,  Stickiness enabled, 1 hour duration*  [ ![](Screenshot%202026-03-30%20144418.png)]  

 

## ALB Created — Live DNS Address

Once created, AWS gave us a public DNS address that visitors can use to reach our website:

| **Status**                      | Active (was  Provisioning first, then became Active)  |
| ------------------------------- | ----------------------------------------------------- |
| **DNS Name  (website address)** | my-ecommers-apl-557059701.us-east-1.elb.amazonaws.com |
| **Hosted Zone**                 | Z35SXDOTRQ7X7K                                        |
| **AZ Coverage**                 | us-east-1a +   us-east-1b                             |
| **Created**                     | March 30,  2026 14:44                                 |

 

  📸  **SCREENSHOT 13**  *ALB Created — showing Status, DNS Name, Availability  Zones, ARN details*  [<img src="Screenshot%202026-03-30%20144512.png" style="zoom:67%;" /> ]  

 

  **✅ Load Balancer is LIVE! DNS address created.  Website is reachable from the internet.**  

 

 

| **STEP 9** | **Creating the  Auto Scaling Group — The Automatic Server Manager** |
| ---------- | ------------------------------------------------------------ |
|            |                                                              |

 

  **💡 What is this?**     The Auto Scaling Group (ASG) is the brain that watches your  servers and automatically: • ADDS new  servers when CPU usage goes above 60% (traffic is high) • REMOVES servers  when CPU usage drops below 30% (traffic is low) • Always keeps at least 2  servers running (so website never goes fully down) • Never goes above 6  servers (to control costs) Every new  server it creates is an exact copy of our Golden AMI 'task1' — so it's already  set up with Nginx and our website!  

 

## Step 1 of 7 — Choose Launch Template

We told the ASG to use our Golden AMI (task1) as the blueprint for new servers:

| **ASG Name**            | my-e-commers-asg                 |
| ----------------------- | -------------------------------- |
| **Launch  Template**    | task1 (our Golden AMI blueprint) |
| **Launch  Template ID** | lt-0d6a4aefcd344ea96             |
| **Version**             | Default (1)                      |

 

  📸  **SCREENSHOT 14**  *Create ASG Step 1 — ASG name 'my-e-commers-asg', Launch  template 'task1' selected*  [ ![](Screenshot%202026-03-30%20144555.png)]  

 

## Step 2 of 7 — Choose Instance Launch Options

We chose the VPC and set the instance type that new servers will use:

| **Instance  Type** | t2.micro (1 CPU, 1GB RAM)                          |
| ------------------ | -------------------------------------------------- |
| **VPC**            | vpc-09f7c6b46a291696c (our my-e-Commers-vpc)       |
| **Subnets  Used**  | Private  subnets in both AZs (pvt1 + pvt2)         |
| **AZ  Balancing**  | Automatic —  ASG spreads servers across both zones |

 

  📸  **SCREENSHOT 15**  *Create ASG Step 2 — showing instance type t2.micro, VPC  my-e-Commers-vpc selected*  [ ![](Screenshot%202026-03-30%20145327.png) ]  

 

## Step 3 of 7 — Connect to Load Balancer

We linked the ASG to our Load Balancer. Now whenever ASG creates a new server, it is automatically added to the Load Balancer's list — no manual work needed!

| **Load  Balancing Option** | Attach to an  existing load balancer |
| -------------------------- | ------------------------------------ |
| **Target  Group Selected** | my-e-commers-tg \|   HTTP            |
| **Associated  ALB**        | my-ecommers-apl                      |

 

  📸  **SCREENSHOT 16**  *Create ASG Step 3 — Attach to existing load balancer,  my-e-commers-tg | HTTP selected*  [ ![](Screenshot%202026-03-30%20145339.png) ]  

 

## Final ASG Settings — Group Size

| **Minimum  Capacity**        | 2 servers (always running — website never fully down)        |
| ---------------------------- | ------------------------------------------------------------ |
| **Desired  Capacity**        | 2 servers (normal operating state)                           |
| **Maximum  Capacity**        | 6 servers (never exceeds this — cost control)                |
| **Termination  Policy**      | Oldest Launch  Template (remove oldest servers first  during scale-in) |
| **Default  Cooldown Period** | 300  seconds (wait 5 min before next  scaling action)        |

 

  📸  **SCREENSHOT 24**  *ASG Advanced Config — Termination Policy: Oldest Launch  Template, Cooldown: 300 seconds*  [ ![](Screenshot%202026-03-30%20153149.png) ]  

 

  **✅ Auto Scaling Group active! Running 2  servers across 2 Availability Zones.**  

 

 

| **STEP 10** | **Setting Up  Scaling Policies — The Rules for Auto Scaling** |
| ----------- | ------------------------------------------------------------ |
|             |                                                              |

 

  **💡 What is this?**     We created two scaling POLICIES — these are the actual rules  that tell the ASG when and how to add/remove servers: • SCALE-OUT policy = 'When CPU is too high,  ADD a server' • SCALE-IN policy = 'When CPU is too low, REMOVE a server' Each policy is linked to a CloudWatch ALARM  that monitors the CPU and fires when the threshold is crossed.  

 

## Scale-Out Policy — Add Servers When Busy

| **Policy Name** | scale-out                                          |
| --------------- | -------------------------------------------------- |
| **Policy Type** | Step  Scaling (adds servers in steps)              |
| **Trigger**     | CloudWatch  Alarm: my-e-commers-alarm-60           |
| **Condition**   | CPU  Utilization >= 60% for 1 period of 60 seconds |
| **Action**      | Add EC2  instances to handle the extra load        |
| **Status**      | Enabled ✓                                          |

 

## Scale-In Policy — Remove Servers When Quiet

| **Policy Name** | scaling-in                                          |
| --------------- | --------------------------------------------------- |
| **Policy Type** | Step Scaling                                        |
| **Trigger**     | CloudWatch  Alarm: my-e-commers-alarm-30            |
| **Condition**   | CPU  Utilization <= 30% for 1 period of 300 seconds |
| **Action**      | Remove EC2  instances to save cost                  |
| **Status**      | Enabled ✓                                           |

 

  📸  **SCREENSHOT 22**  *ASG Scaling Tab — showing both scale-out (CPU>=60)  and scaling-in (CPU<=30) policies side by side*  [ ![](Screenshot%202026-03-30%20152122.png) ]  

 

  **✅ Both scaling policies active! System will  automatically expand and shrink based on real traffic.**  

 

 

| **STEP 11** | **Creating  CloudWatch Alarms — The Smoke Detectors** |
| ----------- | ----------------------------------------------------- |
|             |                                                       |

 

  **💡 What is this?**     CloudWatch is AWS's monitoring service — it watches your  servers 24/7 and records metrics like CPU usage, memory, network traffic  etc. A CloudWatch ALARM is like a  smoke detector. When it detects a condition (CPU > 60%), it fires — and  that fire triggers the scaling policy to add more servers. We created TWO alarms: • Alarm 1: Fires  when CPU >= 60% → tells ASG to add servers • Alarm 2: Fires when CPU <=  30% → tells ASG to remove servers  

 

## Alarm 1: Scale-Out Alarm (CPU High)

Created in CloudWatch → Alarms → Create Alarm:

| **Alarm Name**      | my-e-commers-alarm-60                        |
| ------------------- | -------------------------------------------- |
| **Metric  Watched** | CPUUtilization (checks CPU every 5 minutes)  |
| **Threshold  Type** | Static (a fixed number)                      |
| **Condition**       | Greater/Equal  (>=) than 60                  |
| **Meaning**         | FIRE the alarm  when CPU is at 60% or higher |
| **Linked To**       | scale-out  policy in ASG → adds servers      |

 

  📸  **SCREENSHOT 18**  *CloudWatch Create Alarm — Condition: Greater/Equal,  Threshold value: 60*  [ ![](Screenshot%202026-03-30 151241.png) ]  

 

  📸  **SCREENSHOT 19**  *CloudWatch Alarm Name — showing alarm name:  my-e-commers-alarm-60*  [ ![](Screenshot%202026-03-30%20151305.png) ]  

 

  📸  **SCREENSHOT 20**  *CloudWatch Alarms List — showing my-e-commers-alarm-60  successfully created, State: Insufficient data*  [  ![](Screenshot%202026-03-30%20151329.png)  

 

## Alarm 2: Scale-In Alarm (CPU Low)

| **Alarm Name**      | my-e-commers-alarm-30                          |
| ------------------- | ---------------------------------------------- |
| **Metric  Watched** | CPUUtilization                                 |
| **Condition**       | Lower/Equal  (<=) than 30                      |
| **Period**          | 5 minutes                                      |
| **Meaning**         | FIRE the alarm  when CPU drops to 30% or lower |
| **Linked To**       | scaling-in  policy in ASG → removes servers    |

 

  📸  **SCREENSHOT 21**  *CloudWatch Create Alarm — Condition: Lower/Equal,  Threshold value: 30, Period: 5 minutes*  [ ![](Screenshot%202026-03-30%20151455.png) ]  

 

  **✅ Both CloudWatch alarms created! CPU  monitoring is now active — system will scale automatically.**  

 

 

| **STEP ✓** | **Validation —  Proving Everything Works** |
| ---------- | ------------------------------------------ |
|            |                                            |

 

  **💡 What is this?**     Validation means checking that everything we built is actually  working correctly. This is the most important step — it proves our  architecture is doing what it was designed to do.  

 

## ✅ Validation 1: Health Checks PASSED — 2/2 Servers Healthy

After the Auto Scaling Group launched 2 servers and registered them with the Target Group, we checked the health status. Both servers passed the ALB health check — meaning the Load Balancer can successfully route traffic to them.

 

| **Total  Registered Servers** | 2                                        |
| ----------------------------- | ---------------------------------------- |
| **Healthy  Servers**          | 2 ✅   (ALL servers passing health check) |
| **Unhealthy  Servers**        | 0 ✅   (no failures)                      |
| **Anomalous**                 | 0                                        |
| **Load  Balancer Connected**  | my-ecommers-apl ✅                        |

 

  📸  **SCREENSHOT 23**  *Target Group Health — showing 2 Total targets, 2  Healthy (green), 0 Unhealthy — VALIDATION PASSED*  [ ![](Screenshot%202026-03-30%20152712.png) ]  

 

  **✅ VALIDATED: ALB health checks PASS! Both  servers are healthy and receiving traffic.**  

 

## ✅ Validation 2: Scaling Policies Confirmed Active

  📸  **SCREENSHOT 22**  *ASG Scaling Policies — both scale-out and scaling-in  policies showing as Enabled*  [ ![](Screenshot%202026-03-30%20152122.png) ]  




## ✅ Validation 3: Advanced Config Confirmed

  📸  **SCREENSHOT 24**  *ASG Advanced Config — Termination Policy = Oldest  Launch Template, Default Cooldown = 300s*  [ ![](Screenshot%202026-03-30%20153149.png)]  

 

 

# 📸 Screenshot Placement Guide

Use this table to know EXACTLY which screenshot goes where in this document:

 

| **#**  | **Step**               | **What to Show**                                       | **Filename   (approx)** |
| ------ | ---------------------- | ------------------------------------------------------ | ----------------------- |
| **1**  | Step 1 — VPC           | VPC list  showing my-e-Commers-vpc, Status: Available  | 120229.png              |
| **2**  | Step 2 —  Subnets      | 4 subnets  list: pub1, pub2, pvt1, pvt2                | 121102.png              |
| **3**  | Step 3 — Route  Tables | Pub subnet  associations (pub1+pub2 checked)           | 121408.png              |
| **4**  | Step 3 — Route  Tables | Pvt subnet  associations (pvt1+pvt2 checked)           | 121510.png              |
| **5**  | Step 3 —  Routes       | Edit routes:  0.0.0.0/0 → IGW                          | 121603.png              |
| **6**  | Step 3 — VPC  Map      | VPC overview  showing all subnets + routes             | 122234.png              |
| **7**  | Step 4 — EC2  Launch   | EC2 launch  page with User Data script visible         | 124607.png              |
| **8**  | Step 6 —  Golden AMI   | AMI summary:  task1, ami-0a643783..., Available        | 125654.png              |
| **9**  | Step 5 — EBS  Volume   | Storage  config: 8GiB, gp2, Encrypted                  | 130107.png              |
| **10** | Step 7 —  Target Group | my-e-commers-tg  created, HTTP:80 details              | 143316.png              |
| **11** | Step 8 — ALB  Config   | ALB creation:  name, internet-facing, IPv4             | 144403.png              |
| **12** | Step 8 —  Stickiness   | Listener:  forward to TG, stickiness 1hr ON            | 144418.png              |
| **13** | Step 8 — ALB  Live     | ALB created:  status, DNS name, AZs                    | 144512.png              |
| **14** | Step 9 — ASG  Step 1   | ASG creation:  name + task1 template selected          | 144555.png              |
| **15** | Step 9 — ASG  Step 2   | Instance  options: t2.micro, VPC selected              | 145327.png              |
| **16** | Step 9 — ASG  Step 3   | Attach to load  balancer: my-e-commers-tg              | 145339.png              |
| **18** | Step 11 — CW  Alarm    | Create alarm:  CPU >= 60, Greater/Equal selected       | 151241.png              |
| **19** | Step 11 —  Alarm Name  | Alarm name:  my-e-commers-alarm-60                     | 151305.png              |
| **20** | Step 11 —  Alarm List  | Alarms list:  alarm-60 created, Insufficient data      | 151329.png              |
| **21** | Step 11 —  Scale-In    | Create alarm:  CPU <= 30, Lower/Equal selected         | 151455.png              |
| **22** | Step 10 —  Policies    | ASG scaling  tab: scale-out + scaling-in both Enabled  | 152122.png              |
| **23** | Validation —  Health   | Target Group:  2 Total, 2 Healthy (green), 0 Unhealthy | 152712.png              |
| **24** | Step 9 —  Advanced     | ASG Edit:  Oldest Launch Template, Cooldown 300s       | 153149.png              |

 

  **💡 How to Insert Screenshots in Microsoft  Word:**     9.     Click  inside the dashed 📸 SCREENSHOT box in this document  10.   Go to  Insert → Pictures → Insert Picture from This Device  11.   Select  the matching screenshot file from your computer  12.   Resize it  to fit neatly — right-click → Size and Position → set Width to 15 cm  

 

 

# What We Achieved — Summary

  **💡 What is this?**     If you do not know anything about AWS, here is what we built  in simple words: We built an online  shopping website that: • Runs on Amazon's cloud — not your laptop! • Has 2  copies running at the same time in 2 different cities • If one city has a  problem, the other one takes over automatically • During a sale, it can grow  from 2 servers to 6 servers by itself • After the sale, it shrinks back to 2  to save money • Monitors itself — if a server dies, it is replaced automatically  • Shopping cart does not disappear because sticky sessions keep you on the  same server • All data on disk is encrypted — hackers cannot read it even if  they steal the disk  

 

## Completed Components

|      | **Component**       | **What It Does**                                    | **Status**       |
| ---- | ------------------- | --------------------------------------------------- | ---------------- |
| 🌐    | VPC + Subnets       | Private  network with 4 rooms (2 public, 2 private) | **✅ Done**       |
| 🚪    | Internet  Gateway   | Front door to  the internet                         | **✅ Done**       |
| 🗺️    | Route Tables        | Traffic  direction rules                            | **✅ Done**       |
| 🖥️    | EC2 Servers         | Web servers  running Nginx + website                | **✅ Done**       |
| 💾    | EBS Volume          | Encrypted disk  for each server                     | **✅ Done**       |
| 📷    | Golden AMI          | Blueprint for  creating new servers fast            | **✅ Done**       |
| 📋    | Target Group        | List of  servers for Load Balancer                  | **✅ Done**       |
| ⚖️    | Load Balancer       | Divides  visitors across servers, sticky sessions   | **✅ Done**       |
| 📈    | Auto Scaling  Group | Adds/removes  servers based on load                 | **✅ Done**       |
| ⏱️    | Scaling  Policies   | Rules: add at  60% CPU, remove at 30% CPU           | **✅ Done**       |
| 🚨    | CloudWatch  Alarms  | CPU monitors  that trigger scaling                  | **✅ Done**       |
| ❤️    | Health Checks       | 2/2 servers  healthy — confirmed working!           | **✅ VALIDATED**  |
|      | S3 Bucket           | Store static assets                                 | **✅ Done**       |
|      | Backup Retention    | 7-day recovery window                               | **✅ Configured** |
|      | Versioning          | Maintain file versions                              | **✅ Enabled**    |
|      | AWS Backup          | Automate EBS snapshots                              | **✅ Done**       |
|      | Encryption (SSE-S3) | Secure data at rest                                 | **✅ Enabled**    |

 

AWS Infrastructure Documentation | Task 1 – E-Commerce 3-Tier Architecture | March 2026 | Prepared by Veeranjaneyulu