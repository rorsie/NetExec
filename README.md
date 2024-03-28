# NetExec Enumeration and Exploitation

<h2>Description</h2>
Using NetExec and Bloodhound, I will go through a simulated network of VMs that consisted of various different types of servers (file server, domain controller, and SQL server) to enumerate and exploit any vulnerabilities so that I may obtain access or an account with elevated privleges. Lab created by John Hammond (https://jh.live/nypt)
<br />


<h2>Languages and Utilities Used</h2>

- <b>NetExec</b> 
- <b>Kali Linux</b>
- <b>Bloodhound</b>
- <b>John the Ripper</b>
- <b>Impacket</b>
- <b>Virtual machines</b>
- <b>Python</b>

<h2>Project Walkthrough:</h2>

<p>
<h3 align="center">Initial Setup</h3> <br/>
After the VMs have booted up, I only needed to add the IP addresses to the /etc/hosts file on the Kali VM to be able to connect directly.
<br />
<br />
<h3 align="center">SMB Share Enumeration/h3> <br/>
NetExec makes enumerating shares on this simulated network easy. First I try a NULL session by supplying no username or password. Once that doesn't show anything useful, I try the guest/anonymous session next. This time we at least find the shares and see that we have read permissions on a couple
<img src="authenticateSMB">
<img src="SMBguest">
<br />
<br />
 <h3 align="center">Finding the Right User</h3>
Next, I perform another NULL session but with the "--users" parameter. Luckily this works out perfectly and I find a very interesting user...
<img src="enumerateUSERSnull">
 <br />
Now I'll check if these credentials actually work over SMB. Looks like they do!
<img src="SMBauthSUCCESSreal">
 <br />
 <br />
<h3 align="center"> Bringing the Bloodhound Out</h3>
It's time to really map out this domain and see if there are any Kerberoastable users. For this, I will first export the necessary data from the domain controller in a format fit for Bloodhound/
<img src="bloodhoundCOLLECTION">
<br />
Then I upload all the JSON files to Bloodhound (after logging in of course).
<img src="UPLOADbloodhound">
<br />
Now at the Explore tab, I can search for clients with some very sharp searching tools. For now, I will use the "Cypher" tab to input a custom search to find any users susceptible to Kerberoasting. Our target has now shown up.
<img src="KERBEROASTABLE">
<br />
<br />
<h3 align="center">The Roast Begins</h3>
Time to use NetExec to dump the hash of the user we found. You probably know what is coming next.
<img src="johnHASH">
 <br />
 The trusty John the Ripper comes in handy once again to crack this hash quickly. Now we have access to that account and the SQL server!
<img src="johntheRIPPER">
<img src="kerberosAUTH">
<br />
<br />
 <h3 align="center">Ever-reaching Tenderils</h3>
Impacket has come to my aid to help us remotely log in to the SQL server with the newly acquried credential set.
 <img src="impacketSQLlogin">
<br />
Now I can run a quick 'enum_db' to see all the databases. The 'users' database looks interesting, so I will see if I can search for anything inside there first. What I find is even more intriguing.
 <img src="SQLdbEnum">
<br />
 A quick query will give me all I need to know to continue expanding into the network. We'll have to see where these work.
<img src="SQLaccountCREDENTIALS">
<br />
<br />
<h3 align="center">Final Rungs of the Ladder</h3>
Since I have Bloodhound still open, I can do a quick search for this new user to see if he comes with any interesting bells or whistles. I would say being part of the "FSADMINS" group is a pretty shiny bell.
<img src="bloodhoundBENJAMIN">
<br />
Now we plug the credentials in and aim it at the file server. He got Pwn3d! This means Benjamin is a local admin on the file server!
<img src="benjiPWN3d">
This access gives us more options. I used NetExec to dump the SAM hashes, which could come in use later, and to check other logged on users. A new target reveals itself, one from the domain controller nonetheless.
<img src="SAMDUMPloggedon">
 <br />
 Since I still have local admin access (thanks Benjamin), I can use NetExec to run commands as 'johna'.
 <img src="whoamiJOHNA">
 <br />
 <br />
<h3 align="center"> Wait, That Means...</h3>
Yes! Code execution as a domain admin! Now we can really heat this up. (Especially because Windows Defender is disabled in this environment) <br/>
It's time to start on the reverse shell. I create this with 'msfvenom'. Just a standard, stageless, Meterpreter shell will do. Since this is a Meterpreter shell, I must start up multi/handler.
<img src="multihandler">
To get our payload to the target, I will simply set up a Python web server ("python3 -m http.server") and execute some commands to have it be downloaded by our target.
<img src="pythonserver">
<img src="payload1">
<br />
Now we officially have access to the network as a full domain admin.
<img src="payload3">
<img src="iminJOHN">
<br />
<br />
<h3 align="center">Final Thoughts</h3>
This project really caught my eye initially due to the visualization of all the attacks at the end. Maybe its just me, but it helps connect evreything together when you can plot it out on a map that has more relevance to the real world than just seeing IP addresses and coordinates. This also let me get some much needed experience with Azure and Sentinel, two very prominent and powerful tools. It also introduced me into some powershell scripts which also look very handy. Logs are not a new thing, but seeing how they are created, organized, and manipulated in Azure was new, so that was also beneficial. Overall, I think this was a great project that helped introduce me to Azure, Sentinel, Powershell, Azure logs, and incident detection. It also only took a 2-3 hours. Thanks for reading.

 
</p>

