# Migrating GPG keys as subkeys into new master key

## How to do

导入旧的主密钥对

```sh
gpg --import-options import-restore --import backupkeys-<MASTERKEYID>.sec.asc
# Or
gpg --import master-secret-key-<MASTERKEYID>.sec.asc
```

这是旧的主密钥对

```sh
$ gpg --with-keygrip --list-public-keys

/Users/issenn/.config/gnupg/pubring.kbx
---------------------------------------
sec   rsa4096/0xDD1FD749790FE416 2020-03-19 [SC]
      Key fingerprint = F249 0E1C 0D82 859C 4C17  67D3 DD1F D749 790F E416
      Keygrip = 7C0D9C264C65C676405C7BB9F139C51A27ECB1E1
uid                   [ unknown] Issenn (Master Key) <issennknight@gmail.com>
ssb   rsa4096/0x60BA4EDB9F7E70B4 2020-03-19 [E]
      Keygrip = D8EE6BCF9B03C94FF0F72C90D6460A480D97BB99
ssb   rsa4096/0xDF0B5C535207EF1E 2020-03-19 [E]
      Keygrip = 0A9D01D726678C5259C34F25C66531CE8DD71561
```

```sh
$ gpg --with-keygrip --list-secret-keys

/Users/issenn/.config/gnupg/pubring.kbx
---------------------------------------
pub   rsa4096/0xDD1FD749790FE416 2020-03-19 [SC]
      Key fingerprint = F249 0E1C 0D82 859C 4C17  67D3 DD1F D749 790F E416
      Keygrip = 7C0D9C264C65C676405C7BB9F139C51A27ECB1E1
uid                   [ unknown] Issenn (Master Key) <issennknight@gmail.com>
sub   rsa4096/0x60BA4EDB9F7E70B4 2020-03-19 [E]
      Keygrip = D8EE6BCF9B03C94FF0F72C90D6460A480D97BB99
sub   rsa4096/0xDF0B5C535207EF1E 2020-03-19 [E]
      Keygrip = 0A9D01D726678C5259C34F25C66531CE8DD71561
```

备份旧的主密钥对

```sh
gpg --armor --output backupkeys-base-DD1FD749790FE416.sec.asc --export-options export-backup --export-secret-keys DD1FD749790FE416
```

检查存储所有密钥的私钥部分的文件夹

```sh
ls -lt ${GNUPGHOME:-${XDG_CONFIG_HOME:-${HOME}/.config}/gnupg}/private-keys-v1.d/
```

生成一个新的主密钥对

```sh
$ gpg --expert --full-generate-key

gpg (GnuPG) 2.3.1; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC (sign and encrypt) *default*
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (13) Existing key
  (14) Existing key from card
Your selection? 11

Possible actions for this ECC key: Sign Certify Authenticate
Current allowed actions: Sign Certify

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? s

Possible actions for this ECC key: Sign Certify Authenticate
Current allowed actions: Certify

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? q
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (2) Curve 448
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (6) Brainpool P-256
   (7) Brainpool P-384
   (8) Brainpool P-512
   (9) secp256k1
Your selection? 1
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 3y
Key expires at Tue May  7 09:53:22 2024 CST
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Issenn
Email address: issennknight@gmail.com
Comment: GnuPG Master Key
You selected this USER-ID:
    "Issenn (GnuPG Master Key) <issennknight@gmail.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key 0xADF83E605405B910 marked as ultimately trusted
gpg: revocation certificate stored as '/Users/issenn/.config/gnupg/openpgp-revocs.d/E4B667CD6988085BC854F388ADF83E605405B910.rev'
public and secret key created and signed.

pub   ed25519/0xADF83E605405B910 2021-05-08 [C] [expires: 2024-05-07]
      Key fingerprint = E4B6 67CD 6988 085B C854  F388 ADF8 3E60 5405 B910
uid                              Issenn (GnuPG Master Key) <issennknight@gmail.com>
```

设置口令为 ``me^?C]cR{k&LD\iJK:y4_j^9i.Aj)|5gBk`@F?FP``

这是新的主密钥对

```sh
$ gpg --with-keygrip --list-public-keys

/Users/issenn/.config/gnupg/pubring.kbx
---------------------------------------
pub   ed25519/0xADF83E605405B910 2021-05-08 [C] [expires: 2024-05-07]
      Key fingerprint = E4B6 67CD 6988 085B C854  F388 ADF8 3E60 5405 B910
      Keygrip = 2DF388A26ED4CF0C710228041BCFE093EDCB0CE7
uid                   [ultimate] Issenn (GnuPG Master Key) <issennknight@gmail.com>
```

```sh
$ gpg --with-keygrip --list-secret-keys

/Users/issenn/.config/gnupg/pubring.kbx
---------------------------------------
sec   ed25519/0xADF83E605405B910 2021-05-08 [C] [expires: 2024-05-07]
      Key fingerprint = E4B6 67CD 6988 085B C854  F388 ADF8 3E60 5405 B910
      Keygrip = 2DF388A26ED4CF0C710228041BCFE093EDCB0CE7
uid                   [ultimate] Issenn (GnuPG Master Key) <issennknight@gmail.com>
```

备份新的主密钥对

```sh
gpg --armor --output backupkeys-base-ADF83E605405B910.sec.asc --export-options export-backup --export-secret-keys ADF83E605405B910
```

查看待迁移的密钥的组成数据包

```sh
$ gpg --armor --export-options export-minimal --export-secret-subkeys 0xDF0B5C535207EF1E! | gpg --list-packets

# off=0 ctb=95 tag=5 hlen=3 plen=533
:secret key packet:
    version 4, algo 1, created 1584613416, expires 0
    pkey[0]: [4096 bits]
    pkey[1]: [17 bits]
    gnu-dummy S2K, algo: 0, simple checksum, hash: 0
    protect IV:
    keyid: DD1FD749790FE416
# off=536 ctb=b4 tag=13 hlen=2 plen=44
:user ID packet: "Issenn (Master Key) <issennknight@gmail.com>"
# off=582 ctb=89 tag=2 hlen=3 plen=590
:signature packet: algo 1, keyid DD1FD749790FE416
    version 4, created 1584613416, md5len 0, sigclass 0x13
    digest algo 10, begin of digest f0 a3
    hashed subpkt 33 len 21 (issuer fpr v4 F2490E1C0D82859C4C1767D3DD1FD749790FE416)
    hashed subpkt 2 len 4 (sig created 2020-03-19)
    hashed subpkt 27 len 1 (key flags: 03)
    hashed subpkt 11 len 4 (pref-sym-algos: 9 8 7 3)
    hashed subpkt 21 len 4 (pref-hash-algos: 10 9 8 11)
    hashed subpkt 22 len 4 (pref-zip-algos: 2 3 1 0)
    hashed subpkt 30 len 1 (features: 01)
    hashed subpkt 23 len 1 (keyserver preferences: 80)
    subpkt 16 len 8 (issuer key ID DD1FD749790FE416)
    data: [4095 bits]
# off=1175 ctb=9d tag=7 hlen=3 plen=1862
:secret sub key packet:
    version 4, algo 1, created 1584613532, expires 0
    pkey[0]: [4096 bits]
    pkey[1]: [17 bits]
    iter+salt S2K, algo: 7, SHA1 protection, hash: 2, salt: A6B5F39748A87C32
    protect count: 32505856 (239)
    protect IV:  f1 b8 b3 1f ab fa 15 2a 5a 30 b0 ee 21 cb 99 c5
    skey[2]: [v4 protected]
    keyid: DF0B5C535207EF1E
# off=3040 ctb=89 tag=2 hlen=3 plen=566
:signature packet: algo 1, keyid DD1FD749790FE416
    version 4, created 1584613532, md5len 0, sigclass 0x18
    digest algo 10, begin of digest 68 df
    hashed subpkt 33 len 21 (issuer fpr v4 F2490E1C0D82859C4C1767D3DD1FD749790FE416)
    hashed subpkt 2 len 4 (sig created 2020-03-19)
    hashed subpkt 27 len 1 (key flags: 0C)
    subpkt 16 len 8 (issuer key ID DD1FD749790FE416)
    data: [4094 bits]
```

可以查到子密钥对 `DF0B5C535207EF1E` 的创建时间戳是 `created` `1584613532`。

直接迁移旧密钥会导致密钥指纹产生变化。GPG使用ECDH椭圆曲线密钥协商算法包含密钥的ID，因此，直接迁移后的密钥不能用于解密旧的对象。指纹变化是由时间戳导致的，为了避免这种情况，我们使用 `--faked-system-time="<timestamp>!"` 与 `--ignore-time-conflict`。

> `--faked-system-time` 兼容时间格式 `$(date -d "2020-1-1" +%Y%m%dT%H%M%S)`。

编辑新的主密钥对

迁移子密钥对``
Keygrip = 0A9D01D726678C5259C34F25C66531CE8DD71561

```sh
$ gpg --expert --faked-system-time 1584613532! --ignore-time-conflict --edit-key ADF83E605405B910

gpg (GnuPG) 2.3.1; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

gpg: WARNING: running with faked system time: 2020-03-19 10:25:32
gpg: key 0xADF83E605405B910 was created 414 days in the future (time warp or clock problem)
Secret key is available.

sec  ed25519/0xADF83E605405B910
     created: 2021-05-08  expires: 2024-05-07  usage: C
     trust: ultimate      validity: ultimate
[ultimate] (1). Issenn (GnuPG Master Key) <issennknight@gmail.com>

gpg> addkey
gpg: key has been created 35827385 seconds in future (time warp or clock problem)
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
  (14) Existing key from card
Your selection? 13 （此处可使用别名keygrip）
Enter the keygrip: 0A9D01D726678C5259C34F25C66531CE8DD71561 （ DF0B5C535207EF1E 的 keygrip ）

Possible actions for this RSA key: Sign Encrypt Authenticate
Current allowed actions: Sign Encrypt

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? s

Possible actions for this RSA key: Sign Encrypt Authenticate
Current allowed actions: Encrypt

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? q
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
Key does not expire at all
Is this correct? (y/N) y
Really create? (y/N) y
gpg: key 0xADF83E605405B910 was created 35827385 seconds in the future (time warp or clock problem)
gpg: key 0xADF83E605405B910 was created 414 days in the future (time warp or clock problem)
gpg: public key 0xADF83E605405B910 is 414 days newer than the signature
gpg: key 0xADF83E605405B910 was created 414 days in the future (time warp or clock problem)

sec  ed25519/0xADF83E605405B910
     created: 2021-05-08  expires: 2024-05-07  usage: C
     trust: ultimate      validity: ultimate
ssb  rsa4096/0xDF0B5C535207EF1E
     created: 2020-03-19  expires: never       usage: E
[ultimate] (1). Issenn (GnuPG Master Key) <issennknight@gmail.com>

gpg> save
```

这是迁移后的密钥环

```sh
$ gpg --with-keygrip --list-secret-keys

/Users/issenn/.config/gnupg/pubring.kbx
---------------------------------------
gpg: public key 0xADF83E605405B910 is 414 days newer than the signature
sec   ed25519/0xADF83E605405B910 2021-05-08 [C] [expires: 2024-05-07]
      Key fingerprint = E4B6 67CD 6988 085B C854  F388 ADF8 3E60 5405 B910
      Keygrip = 2DF388A26ED4CF0C710228041BCFE093EDCB0CE7
uid                   [ultimate] Issenn (GnuPG Master Key) <issennknight@gmail.com>
ssb   rsa4096/0xDF0B5C535207EF1E 2020-03-19 []
      Keygrip = 0A9D01D726678C5259C34F25C66531CE8DD71561
```

因为子密钥对的签名还是旧的，所以这个密钥无法使用，且提示警告。

备份迁移后的新主密钥对

```sh
gpg --armor --output backupkeys-migrate-ADF83E605405B910.sec.asc --export-options export-backup --export-secret-keys ADF83E605405B910
```

导出信任

```sh
gpg --check-trustdb
gpg --export-ownertrust > ownertrust-gpg.txt
```

删除旧的主密钥对

```sh
$ gpg --delete-secret-and-public-keys DD1FD749790FE416

gpg (GnuPG) 2.3.1; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


sec  rsa4096/0xDD1FD749790FE416 2020-03-19 Issenn (Master Key) <issennknight@gmail.com>

Delete this key from the keyring? (y/N) y
This is a secret key! - really delete? (y/N) y

pub  rsa4096/0xDD1FD749790FE416 2020-03-19 Issenn (Master Key) <issennknight@gmail.com>

Delete this key from the keyring? (y/N) y
```

删除新的主密钥对

```sh
$ gpg --delete-secret-and-public-keys ADF83E605405B910

gpg (GnuPG) 2.3.1; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


sec  ed25519/0xADF83E605405B910 2021-05-08 Issenn (GnuPG Master Key) <issennknight@gmail.com>

Delete this key from the keyring? (y/N) y
This is a secret key! - really delete? (y/N) y

pub  ed25519/0xADF83E605405B910 2021-05-08 Issenn (GnuPG Master Key) <issennknight@gmail.com>

Delete this key from the keyring? (y/N) y
```

检查存储所有密钥的私钥部分的文件夹

```sh
ls -lt ${GNUPGHOME:-${XDG_CONFIG_HOME:-${HOME}/.config}/gnupg}/private-keys-v1.d/
```

导入迁移后的新主密钥对

```sh
gpg --import-options import-restore --import backupkeys-migrate-ADF83E605405B910.sec.asc
```

分别输入 旧主密钥对的口令 ``${Q4&s%Qj!*&/m{J(X!-E9>`k[pjK{_4p^&8+hk\`` 与 新主密钥对的口令 ``me^?C]cR{k&LD\iJK:y4_j^9i.Aj)|5gBk`@F?FP``

检查存储所有密钥的私钥部分的文件夹

```sh
ls -lt ${GNUPGHOME:-${XDG_CONFIG_HOME:-${HOME}/.config}/gnupg}/private-keys-v1.d/
```

导入信任

```sh
rm "${GNUPGHOME:-${XDG_CONFIG_HOME:-${HOME}/.config}/gnupg}/trustdb.gpg"
gpg --import-ownertrust ownertrust-gpg.txt
```

更新子密钥对的签名

```sh
$ gpg --expert --ignore-time-conflict --edit-key ADF83E605405B910

gpg (GnuPG) 2.3.1; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

gpg: public key 0xADF83E605405B910 is 414 days newer than the signature
Secret key is available.

sec  ed25519/0xADF83E605405B910
     created: 2021-05-08  expires: 2024-05-07  usage: C
     trust: ultimate      validity: ultimate
ssb  rsa4096/0xDF0B5C535207EF1E
     created: 2020-03-19  expires: never       usage: E
[ultimate] (1). Issenn (GnuPG Master Key) <issennknight@gmail.com>

gpg> key 1

sec  ed25519/0xADF83E605405B910
     created: 2021-05-08  expires: 2024-05-07  usage: C
     trust: ultimate      validity: ultimate
ssb* rsa4096/0xDF0B5C535207EF1E
     created: 2020-03-19  expires: never       usage: E
[ultimate] (1). Issenn (GnuPG Master Key) <issennknight@gmail.com>

gpg> expire
Changing expiration time for a subkey.
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 3y
Key expires at Tue May  7 12:10:26 2024 CST
Is this correct? (y/N) y

sec  ed25519/0xADF83E605405B910
     created: 2021-05-08  expires: 2024-05-07  usage: C
     trust: ultimate      validity: ultimate
ssb* rsa4096/0xDF0B5C535207EF1E
     created: 2020-03-19  expires: 2024-05-07  usage: E
[ultimate] (1). Issenn (GnuPG Master Key) <issennknight@gmail.com>

gpg> save
```

这是更新签名后的密钥环

```sh
$ gpg --with-keygrip --list-secret-keys

/Users/issenn/.config/gnupg/pubring.kbx
---------------------------------------
sec   ed25519/0xADF83E605405B910 2021-05-08 [C] [expires: 2024-05-07]
      Key fingerprint = E4B6 67CD 6988 085B C854  F388 ADF8 3E60 5405 B910
      Keygrip = 2DF388A26ED4CF0C710228041BCFE093EDCB0CE7
uid                   [ultimate] Issenn (GnuPG Master Key) <issennknight@gmail.com>
ssb   rsa4096/0xDF0B5C535207EF1E 2020-03-19 [E] [expires: 2024-05-07]
      Keygrip = 0A9D01D726678C5259C34F25C66531CE8DD71561
```

更新子密钥的过期时间

```sh
$ gpg --expert --edit-key ADF83E605405B910

gpg (GnuPG) 2.3.1; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  ed25519/0xADF83E605405B910
     created: 2021-05-08  expires: 2024-05-07  usage: C
     trust: ultimate      validity: ultimate
ssb  rsa4096/0xDF0B5C535207EF1E
     created: 2020-03-19  expires: 2024-05-07  usage: E
[ultimate] (1). Issenn (GnuPG Master Key) <issennknight@gmail.com>

gpg> key 1

sec  ed25519/0xADF83E605405B910
     created: 2021-05-08  expires: 2024-05-07  usage: C
     trust: ultimate      validity: ultimate
ssb* rsa4096/0xDF0B5C535207EF1E
     created: 2020-03-19  expires: 2024-05-07  usage: E
[ultimate] (1). Issenn (GnuPG Master Key) <issennknight@gmail.com>

gpg> expire
Changing expiration time for a subkey.
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 3y
Key expires at Tue May  7 12:14:07 2024 CST
Is this correct? (y/N) y

sec  ed25519/0xADF83E605405B910
     created: 2021-05-08  expires: 2024-05-07  usage: C
     trust: ultimate      validity: ultimate
ssb* rsa4096/0xDF0B5C535207EF1E
     created: 2020-03-19  expires: 2024-05-07  usage: E
[ultimate] (1). Issenn (GnuPG Master Key) <issennknight@gmail.com>

gpg> save
```

这是更新过期时间后的密钥环

```sh
$ gpg --with-keygrip --list-secret-keys

/Users/issenn/.config/gnupg/pubring.kbx
---------------------------------------
sec   ed25519/0xADF83E605405B910 2021-05-08 [C] [expires: 2024-05-07]
      Key fingerprint = E4B6 67CD 6988 085B C854  F388 ADF8 3E60 5405 B910
      Keygrip = 2DF388A26ED4CF0C710228041BCFE093EDCB0CE7
uid                   [ultimate] Issenn (GnuPG Master Key) <issennknight@gmail.com>
ssb   rsa4096/0xDF0B5C535207EF1E 2020-03-19 [E] [expires: 2024-05-07]
      Keygrip = 0A9D01D726678C5259C34F25C66531CE8DD71561
```

将子密钥对的口令更改为与新主密钥对一致

```sh
$ gpg --expert --edit-key ADF83E605405B910

gpg (GnuPG) 2.3.1; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  ed25519/0xADF83E605405B910
     created: 2021-05-08  expires: 2024-05-07  usage: C
     trust: ultimate      validity: ultimate
ssb  rsa4096/0xDF0B5C535207EF1E
     created: 2020-03-19  expires: 2024-05-07  usage: E
[ultimate] (1). Issenn (GnuPG Master Key) <issennknight@gmail.com>

gpg> key 1

sec  ed25519/0xADF83E605405B910
     created: 2021-05-08  expires: 2024-05-07  usage: C
     trust: ultimate      validity: ultimate
ssb* rsa4096/0xDF0B5C535207EF1E
     created: 2020-03-19  expires: 2024-05-07  usage: E
[ultimate] (1). Issenn (GnuPG Master Key) <issennknight@gmail.com>

gpg> passwd

gpg> save
Key not changed so no update needed.
```

分别输入 新主密钥对的口令 ``me^?C]cR{k&LD\iJK:y4_j^9i.Aj)|5gBk`@F?FP`` 与 旧主密钥对的口令 ``${Q4&s%Qj!*&/m{J(X!-E9>`k[pjK{_4p^&8+hk\``

最终得到完整的新密钥环

```plain
/Users/issenn/.config/gnupg/pubring.kbx
---------------------------------------
pub   ed25519/0xADF83E605405B910 2021-05-08 [C] [expires: 2024-05-07]
      Key fingerprint = E4B6 67CD 6988 085B C854  F388 ADF8 3E60 5405 B910
uid                   [ultimate] Issenn (GnuPG Master Key) <issennknight@gmail.com>
sub   rsa4096/0xDF0B5C535207EF1E 2020-03-19 [E] [expires: 2024-05-07]
```

---

其它尝试

```sh
gpg --export-secret-keys DD1FD749790FE416 | gpgsplit -vp DD1FD749790FE416_

cat 000001-005.secret_key 000002-013.user_id 000003-002.sig 000004-007.secret_subkey 000005-002.sig > concatenated.sec.key
```

> [Migrating GPG master keys as subkeys to new master key](https://security.stackexchange.com/questions/32935/migrating-gpg-master-keys-as-subkeys-to-new-master-key)  
> [Google Search](https://www.google.com/search?q=gpg+merge+master+key)  
