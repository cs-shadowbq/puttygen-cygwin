# puttygen-cywin
Steps to recreate a `.PPK` -> `.PEM` CLI tool in cygwin for windows automation reasons.

## What's in the Release?

SHA256(puttygen-cygwin.zip)= 5788962752e401c197ec85dc18c33e33edbfe262646e1f2433ae9cb7b7593e92

The release includes two files:

* puttygen.exe - the cli 64-bit version!
* cygwin1.dll - 2.90x branch of Cygwin (https://www.cygwin.com/)

## Origin

|Arch|Tool|Sha256|Location|
|---|---|---|---|
|x86-64|cygwin2-9-0-branch||https://community.chocolatey.org/packages/Cygwin/2.9.0|
|x86-64|putty-0.76.tar.gz|547cd97a8daa87ef71037fab0773bceb54a8abccb2f825a49ef8eba5e045713f|https://the.earth.li/~sgtatham/putty/latest/putty-0.76.tar.gz|

## Why do this?

 `puttygen` is a completely different tool on unix, and homebrew (macOS) than it is on Windows. 

### Example Usage

```
C:\Users\Administrator>.\puttygen.exe ./my.ppk -O private-openssh -o ./pemkey.pem
C:\Users\Administrator>.\puttygen.exe ./my.ppk -O public-openssh -o ./pemkey.pub
```

### Why Cygwin?

Even the cygwin makefile is designed to compile the windows UI version. Putty has many `Makefile` for each of the diferent architectures and platforms.

Cygwin requires the use of the `mgw` environment, but it doesnt map well to standard windows. 

Note: Why not use WSL2? WSL2 does not work in Amazon EC2 or other virtual environments easily off the shelf.

## What does it do?

```C:\Users\Administrator>.\puttygen.exe -h
PuTTYgen: key generator and converter for the PuTTY tools
Release 0.76
Usage: puttygen ( keyfile | -t type [ -b bits ] )
                [ -C comment ] [ -P ] [ -q ]
                [ -o output-keyfile ] [ -O type | -l | -L | -p ]
  -t    specify key type when generating:
           eddsa, ecdsa, rsa, dsa, rsa1   use with -b
           ed25519, ed448                 special cases of eddsa
  -b    specify number of bits when generating key
  -C    change or specify key comment
  -P    change key passphrase
  -q    quiet: do not display progress bar
  -O    specify output type:
           private             output PuTTY private key format
           private-openssh     export OpenSSH private key
           private-openssh-new export OpenSSH private key (force new format)
           private-sshcom      export ssh.com private key
           public              RFC 4716 / ssh.com public key
           public-openssh      OpenSSH public key
           fingerprint         output the key fingerprint
           text                output the key components as 'name=0x####'
  -o    specify output file
  -l    equivalent to `-O fingerprint'
  -L    equivalent to `-O public-openssh'
  -p    equivalent to `-O public'
  --dump   equivalent to `-O text'
  --reencrypt          load a key and save it with fresh encryption
  --old-passphrase file
        specify file containing old key passphrase
  --new-passphrase file
        specify file containing new key passphrase
  --random-device device
        specify device to read entropy from (e.g. /dev/urandom)
  --primes <type>      select prime-generation method:
        probable       conventional probabilistic prime finding
        proven         numbers that have been proven to be prime
        proven-even    also try harder for an even distribution
  --strong-rsa         use "strong" primes as RSA key factors
  --ppk-param <key>=<value>[,<key>=<value>,...]
        specify parameters when writing PuTTY private key file format:
            version       PPK format version (min 2, max 3, default 3)
            kdf           key derivation function (argon2id, argon2i, argon2d)
            memory        Kbyte of memory to use in passphrase hash
                             (default 8192)
            time          approx milliseconds to hash for (default 100)
            passes        number of hash passes to run (alternative to 'time')
            parallelism   number of parallelisable threads in the hash function
                             (default 1)
```

## How do I recreate these files?

* `puttygen.exe`

This files come from putty 0.76 source 

* <https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html>
* <https://the.earth.li/~sgtatham/putty/latest/putty-0.76.tar.gz>

Download the Package and Expand it in `cygwin shell`

```shell
$ cd ~
$ mkdir sandbox && cd sandbox
$ curl -O -J -L https://the.earth.li/~sgtatham/putty/latest/putty-0.76.tar.gz
$ tar zxvf putty-0.76.tar.gz
```

Compile `puttygen` from the unix folder `Makefile.ux`, not the cygwin folder.

```
$ cd ~/sandbox/putty-0.76/unix
$ make -f Makefile.ux puttygen
cc  -O2 -Wall -std=gnu99 -Wvla -g -I.././ -I../charset/ -I../windows/ -I../unix/ -D _FILE_OFFSET_BITS=64  -c ../cmdgen.c
cc -o puttygen cmdgen.o conf.o console.o ecc.o import.o marshal.o \
        memory.o millerrabin.o misc.o mpint.o mpunsafe.o notiming.o \
        pockle.o primecandidate.o smallprimes.o sshaes.o sshargon2.o \
        sshauxcrypt.o sshbcrypt.o sshblake2.o sshblowf.o sshdes.o \
        sshdss.o sshdssg.o sshecc.o sshecdsag.o sshhmac.o sshmd5.o \
        sshprime.o sshprng.o sshpubk.o sshrand.o sshrsa.o sshrsag.o \
        sshsh256.o sshsh512.o sshsha.o sshsha3.o stripctrl.o time.o \
        tree234.o utils.o uxcons.o uxgen.o uxmisc.o uxnogtk.o \
        uxnoise.o uxpoll.o uxstore.o uxutils.o version.o wcwidth.o \
```

Validate the file and copy it to Windows User space

```
$ cp ./puttygen.exe /cygdrive/c/Users/Administrator/
$ ldd /cygdrive/c/Users/Administrator/puttygen.exe
        ntdll.dll => /cygdrive/c/Windows/SYSTEM32/ntdll.dll (0x7ff95d650000)
        KERNEL32.DLL => /cygdrive/c/Windows/System32/KERNEL32.DLL (0x7ff95d430000)
        KERNELBASE.dll => /cygdrive/c/Windows/System32/KERNELBASE.dll (0x7ff959840000)
        cygwin1.dll => /usr/bin/cygwin1.dll (0x180040000)
```

Copy in Cygwin core dll runtime

```
$ cd /cygdrive/c/Users/Administrator/
$ mkdir puttygen-cygwin
$ cp /usr/bin/cygwin1.dll .
```

Package it

```shell

$ mv cygwin1.dll puttygen.exe puttygen-cygwin/
$ zip -r puttygen-cygwin.zip puttygen-cygwin
  adding: puttygen-cygwin/ (stored 0%)
  adding: puttygen-cygwin/cygwin1.dll (deflated 64%)
  adding: puttygen-cygwin/puttygen.exe (deflated 70%)

$ openssl sha256 puttygen-cygwin.zip
SHA256(puttygen-cygwin.zip)= 5788962752e401c197ec85dc18c33e33edbfe262646e1f2433ae9cb7b7593e92
```


## Appendix: Installing Cygwin 2.90x Branch

Install Cygwin 2.90x Branch via Chocolatey. 

```
C:\Users\Administrator\choco install cygwin -y --version=2.9.0
C:\Users\Administrator\choco install cygwin-devel --source=cygwin
C:\Users\Administrator\choco install gcc-core --source=cygwin
C:\Users\Administrator\choco install make --source=cygwin
C:\Users\Administrator\choco install zip --source=cygwin
```

ReRun installer Selecting packages

* cygwin-devel ~> 3.2.0-1
* gcc-core     ~> 10.2.0-1
* make         ~> 4.3-1
* zip          ~> 3.0-12

```
$ cygcheck -c
Cygwin Package Information
Package                  Version            Status
_autorebase              001007-1           OK
alternatives             1.3.30c-10         OK
base-cygwin              3.8-1              OK
base-files               4.3-3              OK
bash                     4.4.12-3           OK
binutils                 2.37-2             OK
bzip2                    1.0.8-1            OK
ca-certificates          2.50-4             OK
coreutils                8.26-2             OK
crypto-policies          20190218-1         OK
cygutils                 1.4.16-7           OK
cygwin                   3.2.0-1            OK
cygwin-devel             3.2.0-1            OK
dash                     0.5.11.5-1         OK
diffutils                3.8-1              OK
editrights               1.03-1             OK
file                     5.39-1             OK
findutils                4.8.0-1            OK
gawk                     5.1.0-1            OK
gcc-core                 10.2.0-1           OK
getent                   2.18.90-4          OK
grep                     3.7-2              OK
groff                    1.22.4-1           OK
gzip                     1.11-1             OK
hostname                 3.13-1             OK
info                     6.8-2              OK
ipc-utils                1.0-2              OK
less                     590-1              OK
libargp                  20110921-3         OK
libatomic1               11.2.0-1           OK
libattr1                 2.4.48-2           OK
libblkid1                2.33.1-2           OK
libbz2_1                 1.0.8-1            OK
libcrypt2                4.4.20-1           OK
libfdisk1                2.33.1-2           OK
libffi6                  3.2.1-2            OK
libgc1                   8.0.6-1            OK
libgcc1                  11.2.0-1           OK
libgdbm6                 1.18.1-1           OK
libgmp10                 6.2.1-2            OK
libgomp1                 11.2.0-1           OK
libguile2.2_1            2.2.7-1            OK
libiconv2                1.16-2             OK
libintl8                 0.21-1             OK
libisl22                 0.22.1-2           OK
libisl23                 0.24-2             OK
libltdl7                 2.4.6-7            OK
liblz4_1                 1.7.5-1            OK
liblzma5                 5.2.4-1            OK
libmpc3                  1.2.1-2            OK
libmpfr6                 4.1.0-2            OK
libncursesw10            6.1-1.20190727     OK
libp11-kit0              0.23.20-1          OK
libpcre1                 8.45-1             OK
libpcre2_8_0             10.38-1            OK
libpipeline1             1.5.3-1            OK
libpopt-common           1.18-1             OK
libpopt0                 1.18-1             OK
libquadmath0             11.2.0-1           OK
libreadline7             8.1-2              OK
libsigsegv2              2.10-2             OK
libsmartcols1            2.33.1-2           OK
libssl1.1                1.1.1f-1           OK
libstdc++6               11.2.0-1           OK
libtasn1_6               4.14-1             OK
libunistring2            0.9.10-1           OK
libuuid1                 2.33.1-2           OK
libzstd1                 1.5.0-1            OK
login                    1.13-1             OK
make                     4.3-1              OK
man-db                   2.9.4-1            OK
mintty                   3.5.1-1            OK
ncurses                  6.1-1.20190727     OK
openssl                  1.1.1f-1           OK
p11-kit                  0.23.20-1          OK
p11-kit-trust            0.23.20-1          OK
rebase                   4.5.0-1            OK
run                      1.3.4-2            OK
sed                      4.8-1              OK
tar                      1.34-1             OK
terminfo                 6.1-1.20190727     OK
terminfo-extra           6.1-1.20190727     OK
tzcode                   2021d-1            OK
tzdata                   2021d-1            OK
util-linux               2.33.1-2           OK
vim-minimal              8.2.0486-1         OK
w32api-headers           9.0.0-1            OK
w32api-runtime           9.0.0-1            OK
which                    2.20-2             OK
windows-default-manifest 6.4-1              OK
xz                       5.2.4-1            OK
zlib0                    1.2.11-1           OK
zstd                     1.5.0-1            OK
zip                      3.0-12             OK
```
