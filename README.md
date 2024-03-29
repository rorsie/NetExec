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
<h3 align="center">SMB Share Enumeration</h3> 
<br/>
NetExec makes enumerating shares on this simulated network easy. First I try a NULL session by supplying no username or password. Once that doesn't show anything useful, I try the guest/anonymous session next. This time we at least find the shares and see that we have read permissions on a couple
<img src="https://i.ibb.co/M9VtbqY/authenticate-SMBshares-ATTEMPT.png" alt="Authenticating SMB shares" border="0">
<img src="https://i.ibb.co/42DR8ff/SMBguest.png" alt="SMB Guest" border="0">
<br />
<br />
 <h3 align="center">Finding the Right User</h3>
 <br />
Next, I perform another NULL session but with the "--users" parameter. Luckily this works out perfectly and I find a very interesting user...
<img src="https://i.ibb.co/Ypcshvf/enumerate-USERSnull.png" alt="Enumerate users with NULL" border="0">
 <br />
Now I'll check if these credentials actually work over SMB. Looks like they do!
<img src="https://i.ibb.co/vmN6cqR/SMBauth-SUCCESSreal.png" alt="SMB authentication success!" border="0">
 <br />
 <br />
<h3 align="center"> Bringing the Bloodhound Out</h3>
<br />
It's time to really map out this domain and see if there are any Kerberoastable users. For this, I will first export the necessary data from the domain controller in a format fit for Bloodhound/
<img src="https://i.ibb.co/599rDNr/blooudhound-COLLECTION.png" alt="Using NetExec to dump the data for Bloodhound" border="0">
<br />
Then I upload all the JSON files to Bloodhound (after logging in of course).
<img src="https://i.ibb.co/SxwKNSg/UPLOADbloodhound-COLLECTION.png" alt="Uploading JSON files to Bloodhound" border="0">
<br />
Now at the Explore tab, I can search for clients with some very sharp searching tools. For now, I will use the "Cypher" tab to input a custom search to find any users susceptible to Kerberoasting. Our target has now shown up.
<img src="https://i.ibb.co/9nmcYG7/KERBEROASTABLEusers.png" alt="Found some Kerberoastable users with a custom search query" border="0">
<br />
<br />
<h3 align="center">The Roast Begins</h3>
<br />
Time to use NetExec to dump the hash of the user we found. You probably know what is coming next.
<img src="https://i.ibb.co/Lv1zc8W/johnHASH.png" alt="Dumping the hash of the user" border="0">
 <br />
 The trusty John the Ripper comes in handy once again to crack this hash quickly. Now we have access to that account and the SQL server!
<img src="https://i.ibb.co/gdk8rGk/johnthe-RIPPEr.png" alt="Cracking with John the Ripper..." border="0">
<img src="https://i.ibb.co/8gFGq04/kerberoast-AUTHsql.png" alt="Authentication success for the SQL server" border="0">
<br />
<br />
<h3 align="center">Ever-reaching Tenderils</h3>
<br />
Impacket has come to my aid to help us remotely log in to the SQL server with the newly acquried credential set.
<img src="https://i.ibb.co/chtnzGj/impacket-SQLlogin.png" alt="Using Impacket to send over the credentials" border="0">
<br />
Now I can run a quick 'enum_db' to see all the databases. The 'users' database looks interesting, so I will see if I can search for anything inside there first. What I find is even more intriguing...
<img src="https://i.ibb.co/B4t4N3L/SQLdb-ENUM.png" alt="Using a command to enumerate the DBs" border="0">
<br />
 A quick query will give me all I need to know to continue expanding into the network. We'll have to see where these work.
<img src="https://i.ibb.co/vxPbQ7z/SQLaccount-CREDENTIALS.png" alt="Selecting ALL from the DB gives us some new credentials" border="0">
<br />
<br />
<h3 align="center">Final Rungs of the Ladder</h3>
<br />
Since I have Bloodhound still open, I can do a quick search for this new user to see if he comes with any interesting bells or whistles. I would say being part of the "FSADMINS" group is a pretty shiny bell.
<img src="https://i.ibb.co/1bNpDPL/bloodhound-BENJAMIN.png" alt="Using Bloodhound to see what Benjamin is connected to in the network" border="0">
<br />
Now we plug the credentials in and aim it at the file server. He got Pwn3d! This means Benjamin is a local admin on the file server!
<img src="https://i.ibb.co/9hMVD3T/benji-PWN3-D.png" alt="Pwning Benjamin by accessing his local admin account" border="0">
<br />
This access gives us more options. I used NetExec to dump the SAM hashes, which could come in use later, and to check other logged on users. A new target reveals itself, one from the domain controller nonetheless.
<img src="https://i.ibb.co/tpm1NJw/admincheck-SAMDUMPloggeduser.png" alt="Checking admin rights, dumping SAM hashes, and checking for other logged on users" border="0">
 <br />
 Since I still have local admin access (thanks Benjamin), I can use NetExec to run commands as 'johna'.
 <img src="https://i.ibb.co/jVHNJk5/whoami-ASJOHN.png" alt="Running 'whoami' lets us know we have accessed John A's account" border="0">
 <br />
 <br />
<h3 align="center"> Wait, That Means...</h3>
<br />
Yes! Code execution as a domain admin! Now we can really heat this up. (Especially because Windows Defender is disabled in this environment) <br/>
It's time to start on the reverse shell. I create this with 'msfvenom'. Just a standard, stageless, Meterpreter shell will do. Since this is a Meterpreter shell, I must start up multi/handler.
<img src="https://i.ibb.co/w4jxNXp/multihandler.png" alt="Setting up multi/handler" border="0">
<br />
To get our payload to the target, I will simply set up a Python web server ("python3 -m http.server") and execute some commands to have it be downloaded by our target.
<img src="https://i.ibb.co/P4BfwNF/pythonserver.png" alt="Setting up our python server for the payload" border="0">
<img src="https://i.ibb.co/80T3qr1/retrieve-PAYLOAD1.png" alt="Retrieving the payload..." border="0">
<br />
Now we officially have access to the network as a full domain admin.
<img src="https://i.ibb.co/PFsMPNB/retrievepayload3.png" alt="Setting up the meterpreter" border="0">
<img src="https://i.ibb.co/FBrhJ2Q/imin-JOHNA.png" alt="WE ARE JOHN" border="0">
<br />
<br />
<h3 align="center">Final Thoughts</h3>
Coming soon...
 
</p>

