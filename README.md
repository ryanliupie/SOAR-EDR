# SOAR-EDR

### What is SOC? 

A SOC (Security Operations Center) team serves as a centralized location for monitoring and defending against threats. Some functions include monitoring the environment using a SIEM, isolating/shutting down infected endpoints and removing malware. They often need a constant flow of threat intelligence to ensure they have constant flow of data such as IP addresses, domains, hashes, and other indicators. 

SOCs face many challenges such as alert fatigue (false positives) and security tools being deployed without integration in an organization. As well as, when SOCs handle security incidents, there are <b>no written step-by-step instructions</b> for how to investigate alerts. This becomes a problem when new analysts come and try to solve problems but do not know where to begin. 
<

### What is SOAR? 

SOAR stands for Security Orchestration Automation and Response which is a tool that unifies all the security tools used in a SOC (Security Operations Center). 

1. <b>Security:</b> This refers to everything that helps protect a system, network, user, and data from attacks. For instance, it can be logs from a SIEM, firewalls, IDS/IPS. As well are Threat Intelligence, and alerts such as brute-force attempts. 

2. <b>Orchestration:</b> Normally a SOC analyst would have to manually switch between tools such as a SIEM, IAM, ticketing systems and more. This function helps connect tools, pass data between them, and execute a playbook. 

3. <b>Automation:</b> Once all the tools are coordinated through a playbook (predefined list of actions) seen in Orchestration, that playbook, can be automated. This saves a ton of time for a SOC analyst as they do not have to manually click as much anymore. 

4. <b>Response:</b> This is the remediation & containment portion of the playbook, which may occur at any given time where it can also be automated. 


### What is LimaCharlie? 

It is a an Endpoint Detection and Response (EDR) solution that helps customers have real-time visibility by streaming telemetry data. It also uses a YAML (Yet Another Markup Language) which is a human-readable serialization language used for configuration files and applications where data is stored or transmitted. This allows security teams to make highly sophisticated detections and the ability to track detection login. 


### What is Tines? 

This is a SOAR tool that lets IT and Security teams build automation workflows also called "Stories" with no code/low code drag and drop actions. It integrates with many tools, and in this case, it will integrate with the EDR, LimaCharlie. 


### Architecture

<img src="images/playbook.jpg" width="600px" alt="playbook-overview">
<hr>

## How to set up LimaCharlie

Before anything, please go to <a href="https://limacharlie.io/"> LimaCharlie's website</a> and sign up either through a username and password or via an Identity Provider (IP), such as Google. 

### Endpoint Agent Installation 

Once you have signed up, Head over to <a href="https://docs.limacharlie.io/docs/endpoint-agent-installation"> Lima Charlie Endpoint Agent Documentation</a> and the follow the provided steps to install the endpoint agent. This agent is important to produce and transmit telemetry on the endpoint (the machine itself). 

<img src="images/sensor.png" width="800px" alt="sensor-list">

Go to `Sensors` and you should be able to see your endpoint installed. 
<hr>

### Simulate Attack 

on your Windows computer, go to `Settings`, `Privacy & Security`, then to `Windows Security`. If you have a different antivirus software installed such as <b>Avast</b>, click on `Open App`, then disable `File Shield`. Disable any setting that monitors malicious files. 

If not, click on `Virus & threat protection` --> `Manage Settings`. You want to turn off `real-time protection`. You can follow this video --> <a href="https://www.youtube.com/watch?v=TjqzYG_01do"> Disable real-time protection</a> to help navigate this step. 

![alt text](image.png)

Now head over to <a href="https://github.com/AlessandroZ/LaZagne/releases/tag/v2.4.7"> LaZagne</a> and download the <b>LaZagne.exe</b> file. This is a program that lets you retrieve lost passwords on your local computer. ❌ Please do not use this to perform intended malicious actions ❌, only controlled attacks for learning purposes.  

![alt text](image-1.png)

Open <b>Powershell</b>, and head over the directory that holds the file. You can use the `cd` command to do so. You want to run this file by entering `.\LaZagne.exe`. The file will present <b>! BIG BANG !</b> and below it, you will see all the passwords stored locally either in plaintext or as a hash. 
<hr>

### Check LimaCharlie (EDR)

The agent on the endpoint will pick this up in `TimeLine` hopefully. 

![alt text](image-2.png)

Perfect! the <b>endpoint agent</b> was able to pick up this process running on the local computer. If we click on it, we can see all the <b>event details</b>. These details will be important for making the detection rule. 
<hr>

### Develop Detection 

In your organization, click on `Automation`, then `D&R Rules`, hit `ADD RULE` at the top right. There are two sections labeled <b>Detect</b> and <b>Response</b>. This is where we are going to write our detection rules. 

![alt text](image-3.png)

<b>Detect:</b> In this section, copy the following text: 

        ```
    events:
    - NEW_PROCESS
    - EXISTING_PROCESS
    op: and
    rules:
    - op: is windows
    - op: or
    rules:
      - case sensitive: false
        op: ends with
        path: event/FILE_PATH
        value: LaZagne.exe
      - case sensitive: false
        op: contains
        path: event/COMMAND_LINE
        value: LaZagne
        ```

<b>Respond:</b> In this section, copy the following text: 

    ```
    - action: report
    metadata:
    author: rl-DFIR
    description: SOAR-EDR Tool (Detects Lazagne.exe execution)
    falsepositives:
      - Damn it MANN
    level: high
    tags:
      - attack.credential_access
    name: rl- DFIR - Lazagne (SOAR-EDR)
    ```
<hr>

Once the information is filled, name the detection `LaZagne.exe Detection`, then click `Create`. We now have a fully functioning detection. In practice, before deploying detections in a production environment, it is important to test it in a testing environment. This is important as it will help identify if any changes need to be made to the detection logic. 

In this case, go back to `Sensors`, your sensor, `Timeline`, find the LaZagne.exe event, click on it, then click on `COPY EVENT`. After copied, go back to `D&R Rules`, scroll down and you will see `TARGET EVENT`. 

![alt text](image-4.png)

Paste the copied event there, then hit `TEST EVENT`. If the detection logic is valid, it should display Match at the bottom. 

![alt text](image-5.png)
<hr>

### Simulate Attack Again 

Once the detection is made, simulate the attack again and see if LimaCharlie (EDR) will pick it up. 
 
![](image-6.png)

Success! An event showed up in `Detections` and it corresponds to the detection rule that was made. 
<hr>

## Tines and LimaCharlie (Integration)

Go to <a href="https://slack.com/"> Slack.com</a> to setup a new account, and then go ahead and create a `new workspace`. It can be named anything, as long as you know that it is specific to this project. Create a new channel called `alerts`. 

![alt text](image-7.png)

Go to <a href= "https://www.tines.com/">Tines.com</a> and set up a new account. This is where the playbook/story will be created. 

Once LimaCharlie produces a detection, Tines will send a message that contains the provided details to the alerts channel. 

![alt text](image-9.png)

Now in your story, drag `WebHook` to the right. This will allows Tines to receive data instantly when events occur in another system. In this case, it will receive detections from LimaCharlie (EDR). Now copy the `webhook URL`. 

![alt text](image-8.png)

Go back to LimaCharlie, and in your organization, click on `Outputs`, then `ADD OUTPUT`. 

There will be 4 steps that will be required. First, click on `Detections`, `Tines`. Name it `SOAR-EDR`, then the `DESTINATION HOST` will be the <b>webhook URL</b>. 

![alt text](image-10.png)

Now, click `SAVE OUTPUT`, lastly, click `ALL DONE`. 

![](image-11.png)

To double check if they are connected, simulate the attack once again, then go to `Events` on the Webhook. If an event occurred, they are connected!

### Linking Tines and Slack

![alt text](image-12.png)

Go to Slack, click on `More`, `Apps`, in the text bar, search up Tines, and then click `Install`. You will be taken to a website that will help you install the <b>credential</b> needed to link both platforms. Once done, we can move onto the next step. 

![alt text](image-13.png)

In tines, click on `Templates`, search up `Slack`, then click on `Second a message`. 

![alt text](image-14.png)

Go to Slack, choose the `alerts` channel that we created, hit `view channel details`, then copy the `Channel ID`. 

![alt text](image-16.png)

Click on Slack, go to `Build`, and paste the channel ID. After, click on run to make sure that they are linked. 

![](image-15.png)

In the channel, you should see a message from Tines appear! 