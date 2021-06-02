# gnupg-config

## About

Pretty Good Privacy (**PGP**)，是一套用于讯息加密、验证的应用程序，由 [Phil Zimmermann](https://en.wikipedia.org/wiki/Phil%5FZimmermann) 于1991年发布，由一系列散列、数据压缩、对称密钥加密以及公钥加密的算法组合而成。GNU Privacy Guard (**GPG**)，是一个用于加密、签名通信内容以及管理非对称密钥的自由软件，遵循IETF订定的 [OpenPGP技术标准](https://tools.ietf.org/html/rfc4880) 设计，并与PGP保持兼容。

GPG的基于现代密码学，主要是对非对称加密的应用。

- **对称加密** ：又称私钥加密，这类算法在加密与解密时使用 **相同的** 的密钥，通信双方在通信之前需要协商一个密钥。对称加密简单、高效，加密强度随密钥长度的增加而增加，常见加密算法 `DES` 、 `ChaCha20` 、 `AES` 等
- **非对称加密** ：又称公开密钥加密，这类算法采用公钥加密私钥解密，公钥可以随意发布，私钥必须由用户严格保管，通信双方在通信时使用对方的公钥加密自己的信息。非对称加密的数学基础是 **超大整数的因数分解** 、 **整数有限域离散对数** 、 **椭圆曲线离散对数** 等问题的复杂性。数字签名也是基于非对称加密实现，简单地说即将文件散列后使用私钥加密生成签名，验证时散列文件并与公钥解密签名的值做对比进行验证，数字签名可以验证文件完整性，也有防止伪造的作用。常见的加密算法有 `DSA` 、 `RSA` 、 `ECDSA` 等

> The public key is given out to the world; the private key must be kept a secret. Anyone possessing the public key can encrypt a message so that it can only be read by someone possessing the private key. It's also possible to use a private key to sign a file, not encrypt it. If a private key is used to sign a file, then anyone who has the public key can check that the file was signed by that key. Anyone who doesn't have the private key can't forge such a signature.

To illustrate what each key does, and to see an example of why one would both encrypt and sign, consider Snowden (S) and Greenwald (G):

- Each of them creates their own key pair, and they publish their respective public keys.
- Assume (for now) that the public keys are trustworthy.
- Say S wants to send a message to G. To do that:
  - S looks for G's public key.
  - S encrypts the message using G's public key.
  - S signs the encrypted message with his own private key.
- G receives an encrypted signed message. The untrusted email header says it is from S.
  - G looks for S's public key.
  - G verifies the signature using S's public key. Now, G is convinced message is from S.
  - G decrypts message using his own private key.

The whole scheme relies on the assumption that public keys are trustworthy. Otherwise, for instance, the spook K might publish a key with G's name and intercept G's email, so that when S sends the message, K decrypts it instead of G. This is where key certificates come into play.

> [What is the difference between Key, Certificate and Signing in GPG?](https://security.stackexchange.com/a/133393)

- `密码`: 用来保护私钥的.这样子, 即使你的 `~/.gnupg` 目录被泄漏了, 还有一层密码保护
- `主密钥` : 一般是你调用 `gpg --full-gen-key` 命令生成的那个密钥.
- `子密钥` : `subkey`, 跟主密钥几乎没有差别, 除了它是隶属于 `主密钥` 的. 注意, 子密钥是可以单独用来加解密的. 只是它要隶属于主密钥来管理.
- 密钥的功能
  - `[S]` : `签名（Sign）`允许私钥持有者对文件或消息签名，以允许公钥持有者验证文件完整性和文件来源。
  - `[E]` : `加密（Encrypt）`允许使用此密钥用于加密文件或消息。
  - `[A]` : `身份验证（Authenticate）`允许使用此密钥用于验证身份，比如ssh登陆之类的。
  - `[C]` : `认证（Certify）`允许私钥持有者使用该密钥对其他密钥签名，用于认证其他的密钥。只允许`主密钥`持有此标记，且`主密钥`不允许移除此标记。这个密钥的数字, 就是你的身份.

**总结一下**：

信息加密中，他人用公钥来加密，自己用私钥来解密；

数字签名中，自己用私钥来签名(加密)，他人用公钥来验证(解密)；

无论信息加密还是数字签名都是：公钥公开，私钥保密。

## Installation

### Mac OSX

```sh
brew install "gnupg"
brew install "pinentry-mac"
```

## 文件的结构

```plain
${GNUPGHOME}
├── S.gpg-agent          # agent-socket
├── S.gpg-agent.browser  # agent-browser-socket
├── S.gpg-agent.extra    # agent-extra-socket
├── S.gpg-agent.ssh      # agent-ssh-socket
├── S.dirmngr            # dirmngr-socket
├── S.scdaemon
│
├── gpg.conf  # gpg启动配置文件 （需备份）
├── gpg-agent.conf
├── dirmngr.conf
├── scdaemon.conf
│
├── openpgp-revocs.d  # 存储所有预生成的吊销证书的文件夹 （需备份）
│   └── D993DA3B74DF27BA45622637D899FD6DE9C11BCA.rev  # 一个文件名对应了一个相应密钥的OpenPGP指纹，表明其可用于吊销相应的密钥
│
├── private-keys-v1.d  # 存储所有密钥的私钥部分的文件夹，每个私钥以单文件的形式分别存放在该文件夹下 （需备份）
│   ├── 4496F1FD131F010998D2B20B05626D8DCFFC6209.key  # 一个文件名即一个密钥夹
│   └── 785FF8E130273C6BDFD1F18A51A27EE66D1A3545.key  # 密钥夹将密钥的私钥部分与公钥部分对应了起来
│
├── pubring.kbx.lock  # The lock file for `pubring.kbx'.
├── pubring.kbx  # 在 GPG 2.1 或以上的版本中存储密钥环的公钥部分及相关信息的数据库文件，所有公钥都存储在这个文件中，与 gpgsm 共享（需备份）
├── pubring.kbx~
│
├── random_seed  # A file used to preserve the state of the internal random pool.
├── sshcontrol
│
├── tofu.db
├── trustdb.gpg.lock  # The lock file for the trust database.
├── trustdb.gpg  # 可信任数据库 （无需备份）. 推荐备份 ownertrust values (see: [option --export-ownertrust]).
└── trustlist.txt
```

`gnupg` 目录的默认 权限 是 `700`，其中文件的权限是 `600`. 仅目录的所有者有权读写，访问这些文件。这是基于安全考虑，请不要变更。如果不使用这样的安全权限设置，会收到不安全文件的警告。

## 配置

### 环境变量

`GNUPGHOME`
`PINENTRY_USER_DATA`
`SSH_AUTH_SOCK`

```sh
GPG_TTY=${TTY}
# GPG_TTY=$(tty)
SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)
```

## Usage

### 试运行

```sh
gpg --dry-run --expert --full-generate-key
```

### 生成密钥

我们的只有三个步骤：

- 第一步，生成足够强度的主密钥，仅用于签发子密钥；
- 第二步，根据功能不同，添加独立的子密钥；
- 第三步，离线备份主密钥的私钥和吊销证书，把公钥发布到密钥服务器。

#### 生成主密钥对

一个密钥对由一个公钥和一个私钥组成，公钥可以公开发行，而不会影响密钥的保密性;私钥应当小心保管，避免丢失或泄露。

在命令行中执行下面这条命令，创建一个新的密钥对

```sh
gpg --expert --full-generate-key
```

使用 `--generate-key` / `--gen-key` 参数可以创建一个使用默认值的密钥对，会略去自定义密钥参数的步骤。如果想使用完整的步骤创建密钥对，使用 `--full-generate-key` / `--full-gen-key` 参数，可以自定义密钥的参数，具有所有的可选项。在此仅介绍 `--full-generate-key` 参数。

其中 `--gen-key` 是 `--generate-key` 参数的简写， `--full-gen-key` 是 `--full-generate-key` 参数的简写。

> 在 GPG 2.1.17 或以上的版本中，可以使用 `--full-generate-key`，否则使用 `--default-new-key-algo rsa4096 --gen-key`。
>
> [生成新 GPG 密钥](https://docs.github.com/cn/github/authenticating-to-github/generating-a-new-gpg-key)

加上 `--expert` 开启 Expert Mode（专家模式），专家模式允许你自己选择 **不同的加密算法** 与 **不同的密钥种类** 。

> 在 GPG 2.1 或以上的版本中，可以使用较新的 ECDSA+ECDH 算法，用更短的密钥长度换取和 RSA 相近的安全性。在 `--gen-key` 前加上 `--expert` 即可看到选项。要注意支持这种算法的 OpenPGP 程序暂时较少。
>
> [Elliptic Curve Cryptography (ECC)](https://wiki.gnupg.org/ECC)

选择加密算法与密钥种类

```plain
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
Your selection? 11（此处选择7/8/11均可，这里以ECC密钥为例）
```

第一段是版权声明，然后让用户自己选择希望的加密算法与密钥种类。

第一个问题是询问加密算法与密钥对的种类，默认选择第一个选项的 RSA and RSA (直接回车会默认选择1)，会生成两对采用RSA算法且拥有加密、签名、验证功能的密钥。GPG支持RSA、DSA、ECC等加密算法，其中RSA兼容性最好，而ECC被认为能够通过更短的位数提供更高的安全性。不过，目前而言，无论哪种算法在现在都可以基本上是不可能被破解的。

DSA and RSA are algorithm.
DSA和RSA是算法名字。

- Choice 1 means creating two key pairs, both use RSA algorithm, one for signing, one for encrypting.
  选项1表示制作两对公私钥，都使用RSA算法，一对公私钥用于签名，一对公私钥用于加密。
- Choice 2 means creating two key pairs, one uses DSA algorithm, for signing, one uses Elgamal algorithm, for encrypting.
  选项2表示制作两对公私钥，一对公私钥用DSA算法，用于签名，一对公私钥用Elgamal算法，用于加密。
- Choice 3 means creating one key pair, use DSA algorithm, for both signing and encrypting.
  选项3表示制作一对公私钥，使用DSA算法，仅用于签名。
- Choice 4 means creating one key pair, use RSA algorithm, for both signing and encrypting.
  选项4表示制作一对公私钥，使用RSA算法，仅用于签名。

RSA 是目前公认具有足够强度并且兼容性不错的算法。基于安全性和运算速度的考虑我们选择更「新潮」的 椭圆曲线加密算法（ECC），此处我们选择 (11) ECC (自定义用途)。

更改密钥功能标记

这个问题仅会在选择"自定义用途「set your own capabilities」"的情况下才会显示，这意味着我们要自行选择此密钥能进行的操作，且不同的密钥算法所支持的功能也各不相同。比如说ECC算法就不支持加密功能。

此步骤能够允许使用者修改密钥使用范围(修改密钥使用范围标记)。

输入功能对应的字母，能够开启或者关闭相应的功能。不过一次只能输入一个字母。

Just follow the hints, do choices, at last it will output:
根据提示填，最后会得到输出：

```plain
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
```

我们只需要主密钥有 “认证（Certify）” 功能，故 「<kbd>s</kbd>」 「<kbd>Enter</kbd>」 「<kbd>q</kbd>」 「<kbd>Enter</kbd>」 取消主密钥的 “签名（Sign）”功能，只留下“认证（Certify）”功能，然后选择“完成（Finished）”。最后就可以得到一对只能尽进行“认证（Certify）”操作的密钥。

之后会提示选择用于签发密钥的椭圆曲线。其中美国国家标准与技术研究院（NIST）系列椭圆曲线、Brainpool系列椭圆曲线、secp256k1都存在不同的安全风险，不予考虑。尤其是NIST与NSA说不清道不明的关系，可能是故意留下的弱化实现。

25519椭圆曲线是最快的椭圆曲线之一，而且没有专利壁垒，是公有领域的产品，在2013年NSA的Dual_EC_DRBG后门爆出之后备受关注。目前，25519曲线作为P-256的成功替代者，被众多应用广泛使用，支持良好。

```plain
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
```

选择一种椭圆曲线算法，这里我们选择 Curve 25519。

~~其实这里还有一些隐藏算法没有列出，可通过如下命令查看~~

```sh
$ gpg --list-config --with-colons curve
cfg:curve:cv25519;ed25519;nistp256;nistp384;nistp521;brainpoolP256r1;brainpoolP384r1;brainpoolP512r1;secp256k1
```

上面缺少的(2)对应的就是ed25519 ，是一种签名算法，你选择 （1）时， 主密钥用来签发和签名 用的就是ed25519，自动生成的用来加密的子密钥 算法 是cv25519 。

Expiration

```plain
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 3y（设置密钥过期时间，没有特殊需求用默认的不过期即可）
Key expires at Sun May  5 10:36:16 2024 CST
```

选择主密钥时效，根据个人喜好选择即可，建议 [3 年左右](https://www.keylength.com/en/4/)（可随时延长）。

```plain
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Issenn
Email address: issennknight@gmail.com
Comment: GnuPG Master Key
You selected this USER-ID:
    "Issenn (GnuPG Master Key) <issennknight@gmail.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
```

填写名称、邮箱等信息。在弹出的窗口中输入保护私钥的口令（建议强口令，以后都将用到这个口令），稍等片刻即可生成完毕。

```plain
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key 0x90654051BA902E1B marked as ultimately trusted
gpg: revocation certificate stored as '/Users/issenn/.config/gnupg/openpgp-revocs.d/23D08DE0580658B88D87F84E90654051BA902E1B.rev'
public and secret key created and signed.

pub   ed25519/0x90654051BA902E1B 2021-05-08 [C] [expires: 2024-05-07]
      Key fingerprint = 23D0 8DE0 5806 58B8 8D87  F84E 9065 4051 BA90 2E1B
uid                              Issenn (GnuPG Master Key) <issennknight@gmail.com>

```

RSA

Key-length

```plain
Possible actions for a RSA key: Sign Certify Encrypt Authenticate
Current allowed actions: Sign Certify Encrypt （默认启用签名、认证和加密功能）

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? s（禁用签名功能）

Possible actions for a RSA key: Sign Certify Encrypt Authenticate
Current allowed actions: Certify Encrypt（现在仅启用认证和加密功能）

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? e（禁用加密功能）

Possible actions for a RSA key: Sign Certify Encrypt Authenticate
Current allowed actions: Certify（现在仅启用认证功能）

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? q（结束功能设置）
```

我们只需要主密钥有 “认证（Certify）” 功能，故 「<kbd>s</kbd>」 「<kbd>Enter</kbd>」 「<kbd>e</kbd>」 「<kbd>Enter</kbd>」 「<kbd>q</kbd>」 「<kbd>Enter</kbd>」 分别取消主密钥的 “加密（Encrypt）”与“签名（Sign）”功能，只留下“认证（Certify）”功能，然后选择“完成（Finished）”。最后就可以得到一对只能尽进行“认证（Certify）”操作的密钥。

### 密钥管理

#### 列出公钥

```sh
gpg --list-keys --with-keygrip

# Or

gpg -k --with-keygrip
```

- pub其后的是该密钥的公钥特征，包括了密钥的参数（加密算法是rsa，长度为4096，生成于2020-03-19，用途是Signing和Certificating，永不过期）以及密钥的ID。
- uid其后的是生成密钥时所输入的个人信息。
- sub其后的则是该密钥的子密钥特征，格式和公钥部分大致相同（E表示用途是Encrypting）。

#### 列出私钥

```sh
gpg --list-secret-keys

# Or

gpg -K
```

下面那两个 `ssb` 就是 subkey. 第一次创建时, 默认也会创建一个 subkey.

```sh
# 使用此命令列出我的key id，顾名思义LONG 这种形式的id 比一般的id要长
gpg --list-secret-keys --keyid-format LONG
```

### 编辑密钥

#### 编辑模式

```sh
gpg --edit-key <MASTERKEYID>
gpg --expert --edit-key <MASTERKEYID>
```

#### 添加子密钥对

```sh
gpg --edit-key <MASTERKEYID>
gpg> addkey
```

#### 信任密钥

```sh
gpg --edit-key <MASTERKEYID>
gpg>trust

Please decide how far you trust this user to correctly verify other users' keys
(by looking at passports, checking fingerprints from different sources, etc.)

  1 = I don't know or won't say
  2 = I do NOT trust
  3 = I trust marginally
  4 = I trust fully
  5 = I trust ultimately
  m = back to the main menu

Your decision? 5

gpg>save
```

#### 信任 subkey

```sh
gpg --edit-key <MASTERKEYID>
选定哪个 key. N 是序号. 如果正确选中的话, 某个 key 会出现 `*` 符号. 默认是选中所有.
gpg>key N
gpg>trust
gpg>save
```

#### 吊销 subkey

```sh
gpg --edit-key <MASTERKEYID>
选定哪个 key. N 是序号. 如果正确选中的话, 某个 key 会出现 `*` 符号. 默认是选中所有.
gpg>key N
gpg>revkey
gpg>save
```

#### 删除 subkey

```sh
gpg --edit-key <MASTERKEYID>
选定哪个 key. N 是序号. 如果正确选中的话, 某个 key 会出现 `*` 符号. 默认是选中所有.
gpg>key N
gpg>delkey
gpg>save
```

### 备份/导出/导入密钥、上传公钥服务器

- `.pgp` - this extension is for a file that has been encrypted using PGP (if you encrypt filename.txt, the new file created will be named filename.txt.pgp)
- `.gpg` - ~~GNU Privacy Guard public keyring file, binary format. See examples from [4.2 Configuration files](https://www.gnupg.org/documentation/manuals/gnupg/GPG-Configuration.html).~~ this extension is for a file that has been encrypted using GPG. Just consider it the same as .pgp
- `.asc` - this extension is for a public PGP key file saved using the American Standard Code for Information Interchange character-encoding scheme, abbreviated ASCII (when you import or export a public PGP key the file name will be keyname.asc). ASCII-armored signature with or without wrapped document, plain text format. Usually used in [clearsigned documents](https://www.gnupg.org/gph/en/manual/x135.html). Usually it's attached unmodified original doc and its signature. In the usage of detached signatures, you can generate signature only without original doc via `--detach-sig`.
- `.key` - this can be the same as a .pgp, .gpg or .asc file (since they are the same, filename.key can be renamed filename.asc)
- `.sig` - GPG signed document file, binary format. (if you sign filename.txt, a second file, filename.txt.sig, will be created)

> [What are the meaningful differences between .gpg, .sig., & .asc?](https://stackoverflow.com/questions/58929260/what-are-the-meaningful-differences-between-gpg-sig-asc)

`--armor` / `-a` 将二进制的公钥/私钥导出为ASSCII码

#### 备份

```sh
--export-options <parameters>
    backup
    export-backup
        Export for use as a backup. The exported data includes all data which is needed to restore the key or keys later with GnuPG. The format is basically the OpenPGP format but enhanced with GnuPG specific data. All other contradicting options are overridden.

--import-options parameters
    restore
    import-restore
        Import in key restore mode. This imports all data which is usually skipped during import; including all GnuPG specific data. All other contradicting options are overridden.
```

`keep-ownertrust`

> [GPG Input and Output (Using the GNU Privacy Guard)](https://www.gnupg.org/documentation/manuals/gnupg/GPG-Input-and-Output.html)  

```sh
gpg --output backupkeys-<MASTERKEYID>.sec.asc --armor --export-options export-backup --export-secret-keys <MASTERKEYID>
gpg --output backupkeys-<MASTERKEYID>.sec.key --export-options export-backup --export-secret-keys <MASTERKEYID>
# gpg --output backupkeys-<MASTERKEYID>.sec.asc --armor --export-options export-minimal --export-secret-keys <MASTERKEYID>

gpg --import-options import-restore --import backupkeys-<MASTERKEYID>.sec.asc
```

#### 导出主密钥公钥

```sh
gpg --output master-public-key-<MASTERKEYID>.pub.asc --armor --export <MASTERKEYID>
```

```sh
# Prints the GPG key ID, in ASCII armor format
gpg --armor --export <MASTERKEYID> | pbcopy
```

如果指定的密钥ID是主密钥的ID(以pub开头的行所指定的密钥)，那么将不会导出除主密钥公钥外任何公钥。

#### 导出特定子密钥公钥

就目前所知的情况，无论使用哪种导出方式，主密钥公钥 pub 都会包含在导出结果内。虽然说，一般情况下没有单独导出特定子密钥公钥的必要，不过姑且在这里记录一下导出方法。就目前已知的信息，任何情况下，使用gpg命令导出密钥，主密钥公钥都会被包含在导出文件中。因此，此处的“导出子密钥公钥”是指导出不包括其他子密钥公钥的公钥文件

这一特殊导出类型，就目前所知，仅支持密钥ID或密钥指纹指定。

```sh
# 导出特定的子密钥私钥. 注意后面的 ! 号.
gpg --output public-subkey-<KEYID>.sub.asc --armor --export <KEYID>!
```

#### 导出主密钥私钥

```sh
gpg --output master-secret-key-<MASTERKEYID>.sec.asc --armor --export-secret-keys <MASTERKEYID>
```

导出时需要输入密钥密码验证权限。

#### 导出子密钥私钥

```sh
# 这会导出所有的 subkey 的子密钥私钥
gpg --output secret-all-subkey-<MASTERKEYID>.ssb.asc --armor --export-secret-subkeys <MASTERKEYID>
```

与 `--export-secret-key` 选项的区别是， `--export-secret-subkey` 如果拒绝输入密码，将会导出公钥(而不是不导出任何信息)。

另外，需要注意的是， `--export-secret-key` 不允许单独导出子密钥，即使指定密钥ID时使用”!" ，也会在导出结果中包含主私钥。

#### 导出特定子密钥私钥

```sh
# 导出特定的子密钥私钥. 注意后面的 ! 号.
gpg --output secret-subkey-<KEYID>.ssb.asc --armor --export-secret-subkeys <KEYID>!
```

将会导出特定子密钥的私钥，而主密钥私钥和其他子密钥在导出时就此被剥离。

如果指定导出的密钥ID是主密钥私钥的密钥ID(以sec开头的行所指定的密钥)，则除主密钥外的任何信息都将被剥离，比如子密钥私钥以及子密钥公钥。

**注意， `--export-secret-subkey` 是GPG的拓展选项。通过该选项所导出的密钥文件不一定能被其他OpenPGP标准实现上导入。**

#### 导入密钥

```sh
gpg --import <KEYID>.[pub/sec].asc
gpg --keyserver hkp://subkeys.pgp.net --search-keys [用户ID]
```

#### 导出信任

```sh
gpg --check-trustdb
gpg --export-ownertrust > ownertrust-gpg.txt
```

#### 导入信任

```sh
rm "${GNUPGHOME:-${XDG_CONFIG_HOME:-${HOME}/.config}/gnupg}/trustdb.gpg"
gpg --import-ownertrust ownertrust-gpg.txt
```

#### 上传公钥

上传公钥到公钥服务器

```sh
gpg --send-keys [用户ID] --keyserver hkp://subkeys.pgp.net
```

生成用于公布的公钥指纹（用于他人校验）

`--fingerprint` `--with-fingerprint` `--with-subkey-fingerprint` `--with-subkey-fingerprints`

```sh
gpg --fingerprint <用户ID>
```

### 吊销密钥

`--generate-revocation name` / `--gen-revoke name`

生成一张撤销证书，用于密钥作废，请求外部公钥服务器撤销你的公钥

```sh
gpg --generate-revocation <KEYID>

# Or

gpg --gen-revoke <KEYID>
```

```sh
# 二进制证书 revocation-cert-<KEYID>.rev.cert； 但会提示 “已强行使用 ASCII 封装过的输出”
gpg --output revocation-cert-<KEYID>.rev.cert --gen-revoke <KEYID>

# -a (--armor)输出文件为 revocation-cert-<KEYID>.rev.asc 文本文件. armor参数可以将其转换为ASCII码显示。
gpg --output revocation-cert-<KEYID>.rev.asc --armor --gen-revoke <KEYID>
```

将吊销证书导入即可吊销本地

吊销证书默认存储位置 `${GNUPGHOME:-${XDG_CONFIG_HOME:-${HOME}/.config}/gnupg}/openpgp-revocs.d/`

### 删除密钥

#### 删除公钥

```sh
gpg --delete-keys [key-id]
```

#### 删除私钥

```sh
gpg --delete-secret-keys [key-id]
```

**如果要删除属于自己创建的公钥，首先需要删除对应的私钥。**

#### 同时删除私钥与公钥

```sh
--delete-secret-and-public-keys
```

### 加密和解密

加密

```sh
gpg --output demo.en.txt --recipient [收件人ID/密钥标识ID] --encrypt demo.txt
```

批量加密 encrypt:

```sh
ls | gpg --multifile --encrypt
# or
ls | gpg --encrypt-files -r <recipient>
```

解密

```sh
gpg --output demo.de.txt --decrypt demo.en.txt
```

批量解密 decrypt:

```sh
ls | gpg --multifile --decrypt
#or
ls | gpg --decrypt-files
```

### 签名

对文件签名

```sh
# 采用二进制储存.gpg文件
gpg --sign demo.txt

# 生成ASCII码的.asc签名文件
gpg --clearsign demo.txt

# 生成单独的.sig签名文件
gpg --detach-sign demo.txt

# 采用ASCII码形式
gpg --armor --detach-sign demo.txt
```

签名+加密

```sh
gpg --local-user [发信者ID] --recipient [接收者ID] --armor --sign --encrypt demo.txt
```

验证签名

```sh
gpg --verify demo.txt.asc demo.txt
```

验证口令

```sh
gpg --passwd <KEYID>
```

```plain
Constant           Character      Explanation
─────────────────────────────────────────────────────
PUBKEY_USAGE_SIG      S       key is good for signing
PUBKEY_USAGE_CERT     C       key is good for certifying other signatures
PUBKEY_USAGE_ENC      E       key is good for encryption
PUBKEY_USAGE_AUTH     A       key is good for authentication
```

```plain
主密钥显示：SC（“sign”&“certify”，代表可以签名和认证其它密钥）

第一个副密钥显示：E（“encrypt”，加密）

第二个副密钥显示：S（“sign”，签名
```

停止gpg-agent

```sh
gpgconf --kill gpg-agent
```

重新加载gpg-agent

```sh
gpg-connect-agent reloadagent /bye
```

```sh
curl https://github.com/web-flow.gpg | gpg --import
gpg --sign-key <GithubKEYID>
git log --show-signature
```

让随机数字发生器获得足够的熵数

```sh
dd if=/dev/random of=/dev/null bs=1 count=2000000
```

- `/dev/random`
- `/dev/urandom`
- `/dev/null` produces **no output**.
- `/dev/zero` produces **a continuous stream of NULL (zero value) bytes**.

名词解释

```plain
sec   rsa4096/0xDD1FD749790FE416 2020-03-19 [SC] [expires: 2021-03-19]
      Key fingerprint = F249 0E1C 0D82 859C 4C17  67D3 DD1F D749 790F E416
uid                   [ultimate] Issenn (Master Key) <issennknight@gmail.com>
ssb   rsa4096/0x60BA4EDB9F7E70B4 2020-03-19 [E]
```

```plain
sec   rsa4096/0xDD1FD749790FE416 2020-03-19 [SC] [expires: 2021-03-19]
```

- `sec` => `SECret key` - We have the secret (private) part of this primary key in our keyring.
- `rsa4096` - This is a 4096-bit RSA key.
- `0xDD1FD749790FE416` - This the “KeyID,” the tail end of the key’s “fingerprint.”
- `2020-03-19` - The creation date of this key.
- `[expires: 2021-03-19]` - This key should no longer be used. It expired 2021-03-19.

```plain
uid                   [ultimate] Issenn (Master Key) <issennknight@gmail.com>
```

- `uid` - What follows is a uid for this key (a key may have zero or more uids).
- `Issenn` - The “Real Name”
- `(Master Key)` - The “Comment”
- `<issennknight@gmail.com>` - The e-mail address

```plain
ssb   rsa4096/0x60BA4EDB9F7E70B4 2020-03-19 [E]
```

- `ssb` => `Secret SuBkey` - This is a subkey. We have the secret part in our keyring.
- `rsa4096` - This subkey is another 4096-bit RSA key.
- `0x60BA4EDB9F7E70B4` - The “KeyID” for this subkey, again derived from the key’s fingerprint.
- `2020-03-19` - The creation date of this subkey.

```plain
pub   rsa4096/0xDD1FD749790FE416 2020-03-19 [SC]
      Key fingerprint = F249 0E1C 0D82 859C 4C17  67D3 DD1F D749 790F E416
uid                   [ultimate] Issenn (Master Key) <issennknight@gmail.com>
sub   rsa4096/0x60BA4EDB9F7E70B4 2020-03-19 [E]
```

- `pub` => `PUBlic key`
- `sub` => `Public SUBkey`

```sh
chown -R $(whoami) "${GNUPGHOME:-${XDG_CONFIG_HOME:-${HOME}/.config}/gnupg}"
find "${GNUPGHOME:-${XDG_CONFIG_HOME:-${HOME}/.config}/gnupg}" -type f -exec chmod 600 {} \;
find "${GNUPGHOME:-${XDG_CONFIG_HOME:-${HOME}/.config}/gnupg}" -type d -exec chmod 700 {} \;
```

> [GPG 的正确使用姿势](https://mogeko.me/2019/068/)
>
> [4.1.3 How to manage your keys](https://www.gnupg.org/documentation/manuals/gnupg/OpenPGP-Key-Management.html)
>
> [GnuPG WIKI](https://wiki.archlinux.org/index.php/GnuPG)

<!-- Further Reading -->
