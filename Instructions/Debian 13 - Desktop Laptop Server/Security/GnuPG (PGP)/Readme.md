# Introduction

OpenPGP is basically specifying the standard, which - in result - is implemented by various software. One of the software is GnuGP with its command line tool gpg.

GnuPG: Using the GNU Privacy Guard

Arch Linux: GnuPG

See OpenPGP Best Practices

## gpg vs. gpg2

New versions of GnuGP propagate gpg2, and on some distributions/plattforms  an older version could exists in parallel.

Example of Debian 12 with GnuPG 2.2.40

```console
ls -la /usr/bin/gpg*
-rwxr-xr-x 1 root root 1108440 26. Mär 2023  /usr/bin/gpg
lrwxrwxrwx 1 root root       3 26. Mär 2023  /usr/bin/gpg2 -> gpg
```

Example of Windows 11 & cygwin with GnuPG 2.2.35

```console
$ ls -l /usr/bin/gpg*
... /usr/bin/gpg-agent.exe
... /usr/bin/gpg-connect-agent.exe
... /usr/bin/gpg-wks-server.exe
... /usr/bin/gpg2.exe
... /usr/bin/gpgconf.exe
... /usr/bin/gpgparsemail.exe
... /usr/bin/gpgscm.exe
... /usr/bin/gpgsm.exe
... /usr/bin/gpgsplit.exe
... /usr/bin/gpgtar.exe
... /usr/bin/gpgv2.exe
```

## Capabilities

The capabilities of your platform can be identify, by asking GnuPG itself.

On Debian 12

```console
gpg --version
gpg (GnuPG) 2.2.40
libgcrypt 1.10.1
Copyright (C) 2022 g10 Code GmbH
License GNU GPL-3.0-or-later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: /root/.gnupg
Supported algorithms:
Pubkey: RSA, ELG, DSA, ECDH, ECDSA, EDDSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
           CAMELLIA128, CAMELLIA192, CAMELLIA256
Hash: SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB, BZIP2
```

On Windows 11 with Cygwin

```console
gpg2 --version
gpg (GnuPG) 2.2.35-unknown
libgcrypt 1.10.2-unknown
Copyright (C) 2022 g10 Code GmbH
License GNU GPL-3.0-or-later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: /home/<user-id>/.gnupg
Supported algorithms:
Pubkey: RSA, ELG, DSA, ECDH, ECDSA, EDDSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH, CAMELLIA128, CAMELLIA192, CAMELLIA256
Hash: SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB, BZIP2
```

### Algorithm IDs

At certain points, you might be given an id of the algorithm instead of the acronym. The following might help.

Unknown algorithm 22

|       ID | Algorithm |
| -------: | :--- |
|        1 | RSA (Encrypt or Sign) \[HAC\] |
|        2 | RSA Encrypt-Only \[HAC\] |
|        3 | RSA Sign-Only \[HAC\] |
|       16 | Elgamal (Encrypt-Only) \[ELGAMAL\] \[HAC\] |
|       17 | DSA (Digital Signature Algorithm) \[FIPS186\] \[HAC\] |
|       18 | ECDH public key algorithm |
|       19 | ECDSA public key algorithm \[FIPS186\] |
|       20 | Reserved (formerly Elgamal Encrypt or Sign) |
|       21 | Reserved for Diffie-Hellman |
|          | (X9.42, as defined for IETF-S/MIME) |
|       22 | EdDSA \[I-D.irtf-cfrg-eddsa\] |
| 100--110 | Private/Experimental algorithm |

Latest information can be picked up from the source code. See github: gnupg / common / openpgpdefs.h

```c
:

typedef enum
  {
    SIGSUBPKT_TEST_CRITICAL = -3,
    SIGSUBPKT_LIST_UNHASHED = -2,
    SIGSUBPKT_LIST_HASHED   = -1,
    SIGSUBPKT_NONE	    =  0,
    SIGSUBPKT_SIG_CREATED   =  2, /* Signature creation time. */
    SIGSUBPKT_SIG_EXPIRE    =  3, /* Signature expiration time. */
    SIGSUBPKT_EXPORTABLE    =  4, /* Exportable. */
    SIGSUBPKT_TRUST	    =  5, /* Trust signature. */
    SIGSUBPKT_REGEXP	    =  6, /* Regular expression. */
    SIGSUBPKT_REVOCABLE     =  7, /* Revocable. */
    SIGSUBPKT_KEY_EXPIRE    =  9, /* Key expiration time. */
    SIGSUBPKT_ARR	    = 10, /* Additional recipient request. */
    SIGSUBPKT_PREF_SYM	    = 11, /* Preferred symmetric algorithms. */
    SIGSUBPKT_REV_KEY	    = 12, /* Revocation key. */
    SIGSUBPKT_ISSUER	    = 16, /* Issuer key ID. */
    SIGSUBPKT_NOTATION	    = 20, /* Notation data. */
    SIGSUBPKT_PREF_HASH     = 21, /* Preferred hash algorithms. */
    SIGSUBPKT_PREF_COMPR    = 22, /* Preferred compression algorithms. */
    SIGSUBPKT_KS_FLAGS	    = 23, /* Key server preferences. */
    SIGSUBPKT_PREF_KS	    = 24, /* Preferred keyserver. */
    SIGSUBPKT_PRIMARY_UID   = 25, /* Primary user id. */
    SIGSUBPKT_POLICY	    = 26, /* Policy URL. */
    SIGSUBPKT_KEY_FLAGS     = 27, /* Key flags. */
    SIGSUBPKT_SIGNERS_UID   = 28, /* Signer's user id. */
    SIGSUBPKT_REVOC_REASON  = 29, /* Reason for revocation. */
    SIGSUBPKT_FEATURES      = 30, /* Feature flags. */

    SIGSUBPKT_SIGNATURE     = 32, /* Embedded signature. */
    SIGSUBPKT_ISSUER_FPR    = 33, /* Issuer fingerprint. */
    SIGSUBPKT_PREF_AEAD     = 34, /* Preferred AEAD algorithms. */

    SIGSUBPKT_ATTST_SIGS    = 37, /* Attested Certifications.  */
    SIGSUBPKT_KEY_BLOCK     = 38, /* Entire key used.          */

    SIGSUBPKT_FLAG_CRITICAL = 128
  }
sigsubpkttype_t;


typedef enum
  {
    CIPHER_ALGO_NONE	    =  0,
    CIPHER_ALGO_IDEA	    =  1,
    CIPHER_ALGO_3DES	    =  2,
    CIPHER_ALGO_CAST5	    =  3,
    CIPHER_ALGO_BLOWFISH    =  4, /* 128 bit */
    /* 5 & 6 are reserved */
    CIPHER_ALGO_AES         =  7,
    CIPHER_ALGO_AES192      =  8,
    CIPHER_ALGO_AES256      =  9,
    CIPHER_ALGO_TWOFISH	    = 10, /* 256 bit */
    CIPHER_ALGO_CAMELLIA128 = 11,
    CIPHER_ALGO_CAMELLIA192 = 12,
    CIPHER_ALGO_CAMELLIA256 = 13,
    CIPHER_ALGO_PRIVATE10   = 110
  }
cipher_algo_t;


/* Note that we encode the AEAD algo in a 3 bit field at some places.  */
typedef enum
  {
    AEAD_ALGO_NONE	    =  0,
    AEAD_ALGO_EAX	    =  1,
    AEAD_ALGO_OCB	    =  2
  }
aead_algo_t;


typedef enum
  {
    PUBKEY_ALGO_RSA         =  1,
    PUBKEY_ALGO_RSA_E       =  2, /* RSA encrypt only (legacy). */
    PUBKEY_ALGO_RSA_S       =  3, /* RSA sign only (legacy).    */
    PUBKEY_ALGO_ELGAMAL_E   = 16, /* Elgamal encrypt only.      */
    PUBKEY_ALGO_DSA         = 17,
    PUBKEY_ALGO_ECDH        = 18, /* RFC-6637  */
    PUBKEY_ALGO_ECDSA       = 19, /* RFC-6637  */
    PUBKEY_ALGO_ELGAMAL     = 20, /* Elgamal encrypt+sign (legacy).  */
    /*                        21     reserved by OpenPGP.            */
    PUBKEY_ALGO_EDDSA       = 22, /* EdDSA.                          */
    PUBKEY_ALGO_KY768_25519 = 29, /* Kyber768 + X25519               */
    PUBKEY_ALGO_KY1024_448  = 30, /* Kyber1024 + X448                */
    PUBKEY_ALGO_DIL3_25519  = 35, /* Dilithium3 + Ed25519            */
    PUBKEY_ALGO_DIL5_448    = 36, /* Dilithium5 + Ed448              */
    PUBKEY_ALGO_SPHINX_SHA2 = 41, /* SPHINX+-simple-SHA2             */
    PUBKEY_ALGO_PRIVATE10   = 110
  }
pubkey_algo_t;


typedef enum
  {
    DIGEST_ALGO_MD5         =  1,
    DIGEST_ALGO_SHA1        =  2,
    DIGEST_ALGO_RMD160      =  3,
    /* 4, 5, 6, and 7 are reserved. */
    DIGEST_ALGO_SHA256      =  8,
    DIGEST_ALGO_SHA384      =  9,
    DIGEST_ALGO_SHA512      = 10,
    DIGEST_ALGO_SHA224      = 11,
    DIGEST_ALGO_PRIVATE10   = 110
  }
digest_algo_t;


typedef enum
  {
    COMPRESS_ALGO_NONE      =  0,
    COMPRESS_ALGO_ZIP       =  1,
    COMPRESS_ALGO_ZLIB      =  2,
    COMPRESS_ALGO_BZIP2     =  3,
    COMPRESS_ALGO_PRIVATE10 = 110
  }
compress_algo_t;

:
```

## Interoperability

More important than the capabilities is the interoperability. Means, all what you generate using GnuPG has to be able to be used on other systems than yours alone.

Read the following part - taken from the man page - and head over to the next section "Compliance".

"... Only override this safe default if you really know what you are doing. If you absolutely must override the safe default, or if the preferences on a given key are invalid for some reason, you are far better off using the --pgp6, --pgp7, or --pgp8 options. These options are safe as they do not force any particular algorithms in violation of OpenPGP, but rather reduce the available algorithms to a "PGP-safe" list."

From the GnuPG: gpg man page

## Compliance

You can predefine the bahviour of gpg, following the rules of a given standard.

List of available standards.

```console
gpg --compliance help
gpg: valid values for option '--compliance':
gpg:   gnupg
gpg:   openpgp
gpg:   rfc4880bis
gpg:   rfc4880
gpg:   rfc2440
gpg:   pgp6
gpg:   pgp7
gpg:   pgp8
gpg:   de-vs
```

Where at the moment of writing ...

```console
gpg --compliance rfc4880bis
gpg: WARNING: using experimental features from RFC4880bis!
:
```

| compliance | parameter ¹          | comment |
| ---------: | :------------------- | :-------|
|      gnupg | \--gnupg             | default; focus on compatibility |
|    openpgp | \--openpgp --rfc4880 |  |
| rfc4880bis |                      | "... using experimental features ..." |
|    rfc4880 | \--openpgp --rfc4880 |  |
|    rfc2440 | \--rfc2440           | obsolete?; RFC 4880 out since 2007 |
|       pgp6 | \--pgp6 --pgp7       | obsolete |
|       pgp7 | \--pgp6 --pgp7       | "... as ... compliant as possible ..." !? |
|       pgp8 | \--pgp8              | "... as ... compliant as possible ..." !? |
|      de-vs |                      | quick search > no result > learned nothing :-( |

¹ of course, the parameter --compliance <compliance name> is valid as well and thankfully for the standard "de-vs"

Where at the moment of writing and a conservative view ... that reduces the list down to ...

gnupg vs. rfc4880

"as fancy/new as possible" vs. "Mr. Standard"

## Hands on

Okay, enough theory ... let's find out, what the "Compliance" means, with regards to Email encryption and signing.

Do not pay to much attention to all the parameters. The first one is the important one, as it is used to specify the compliance to follow.

```console
# create key, based on gnupg standard
gpg --compliance gnupg --batch --passphrase xyz --quick-generate-key C-gnupg default encrypt,sign 0
:
# create subkey, based on gnupg standard
gpg --with-colons --list-secret-keys C-gnupg | grep ^fpr | awk -F : '{ print $10 }' | xargs -I{} gpg --compliance gnupg --batch --passphrase xyz --quick-generate-key {} default encrypt,sign 1y
:
# create key, based on rfc4880 standard
gpg --compliance rfc4880 --batch --passphrase xyz --quick-generate-key C-rfc4880 default encrypt,sign 0
:
# create subkey, based on rfc4880 standard
gpg --with-colons --list-secret-keys C-rfc4880 | grep ^fpr | awk -F : '{ print $10 }' | xargs -I{} gpg --compliance rfc4880 --batch --passphrase xyz --quick-generate-key {} default encrypt,sign 1y
:
# list all newly created keys (secret or public, it doens't matter)
gpg --list-secret-keys
/home/.../.gnupg/pubring.kbx
---------------------------------
sec   rsa3072 2023-11-06 [SCE]
      B2516820F9A9C609198A5F18563592292C891971
uid           [ultimate] C-gnupg

sec   rsa3072 2023-11-06 [SCE] [expires: 2024-11-05]
      F289298C2B37D15CBF31C2E79D57FE140F2747AB
uid           [ultimate] B2516820F9A9C609198A5F18563592292C891971

sec   rsa3072 2023-11-06 [SCE]
      9709406B93D5045839FE852C15B72B3E3A6337DF
uid           [ultimate] C-rfc4880

sec   rsa3072 2023-11-06 [SCE] [expires: 2024-11-05]
      E1312E65DF6A1408FDCC2DA58324A104ECB6268D
uid           [ultimate] 9709406B93D5045839FE852C15B72B3E3A6337DF
```

```
$ gpg --export --armor > qqq.asc
$ gpg --list-packets qqq.asc
```

Based on (at least partically)
Add second sub-key to unattended GPG keyUsing output of awk to run command

## Home Directory

When using the gnupg as a regular user, the tool used by default the directory \~/.gnupg for storing all relevant information (keys, subkeys, ...).

After the start of command gpg -v, simply press CTRL+D.

```console
gpg -v  
gpg: enabled compatibility flags:
gpg: directory '/root/.gnupg' created
gpg: WARNING: no command supplied.  Trying to guess what you mean ...
gpg: Go ahead and type your message ...
gpg: processing message failed: Unknown system error
```

The default directory can be changed, using the parameter --homedir < directory >.

If executed as root ...

```console
cd ~
mkdir qqq
gpg --homedir ~/qqq -v
gpg: WARNING: unsafe permissions on homedir '/root/qqq'
gpg: enabled compatibility flags:
gpg: keybox '/root/qqq/pubring.kbx' created
gpg: WARNING: no command supplied.  Trying to guess what you mean ...
gpg: Go ahead and type your message ...
gpg: processing message failed: Unknown system error
```

## Configuration

See GnuPG: 4.3 Configuration files

Use an "gpg.conf" file like the following, to ensure that you will be at least able to issue subkeys (by having "personal-digest-preferences ... SHA256 ... cert-digest-algo SHA256 ...").

```code
# when outputting certificates, view user IDs distinctly from keys:
fixed-list-mode
# long keyids are more collision-resistant than short keyids
keyid-format 0xlong
# choose the strongest digest:
personal-digest-preferences SHA512 SHA384 SHA256 SHA224
# preferences for new keys:
default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 BZIP2 ZLIB ZIP Uncompressed
# use gpg-agent
use-agent
# show which User IDs gpg thinks are bound to keys in the keyring:
verify-options show-uid-validity
list-options show-uid-validity
# for OpenPGP certification, use a strong digest:
cert-digest-algo SHA256
```

From the GNUPG: Best Practices

# Key Creation (four separate keys)

To separate the action, on creating your GPG keys for one or more email, create new directory and navigate into the same.

```console
mkdir --mode=700 PGP.example.org
cd PGP.example.org
```

## Generate PGP certify key

- ⬆ start the creation of the own base key, using command: `gpg --homedir=. --full-gen-key --expert`
    - ✔️✏️ enter `11` to select `(11) ECC (set your own capabilities)` on question `Please select what kind of key you want:`
        - ⬆✔️✏️ enter `S` to change the `Current allowed actions:` from `Sign Certify` to `Certify`
        - ✔️🔙 enter `Q` to end the selection of action
    - ✔️✏️ enter `1` to select `(1) Curve 25519` on question `Please select which elliptic curve you want:`
    - ✔️✏️ enter `5y` for the certify key to last five years
    - ✔️✏️ enter `Y` on question `Is this correct? (y/N)` to confirm the printed expiry date is correct
    - ✔️✏️ enter the real name, when prompted for `Real name:`
    - ✔️✏️ enter the email address, when prompted for `Email address:`
    - ✔️✏️ press directly ENTER or enter a comment, on prompt `Comment:`
    - ✔️✏️ enter `O` ("o" for Okay), on question `Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit?` to confirm the printed information is correct
    - ✔️✏️ enter the Passphrase for this key
    - ✔️🔙 re-enter the Passphrase for this key

¹ *the Passphrase is crucial, so do not loose it ...*

## Generate Sub-Key for Sign

- ⬆ start the creation of the sub-key, using command: `gpg --homedir=. --edit-key --expert qqq@example.org`
    - ⬆✔️✏️ enter command `addkey` after the `gpg>` prompt
        - ⬆✔️✏️ enter `11` to select `(11) ECC (set your own capabilities)` on question `Please select what kind of key you want:`
            - 🔙 enter `Q` to keep `Current allowed actions: Sign`
        - ✔️✏️ enter `1` to select `(1) Curve 25519` on question `Please select which elliptic curve you want:`
        - ✔️✏️ enter `1y` for the sign key to last one year
        - ✔️✏️ enter `Y` on question `Is this correct? (y/N)` to confirm the printed expiry date is correct
        - ✔️✏️ enter `Y` on question `Really create? (y/N)` to confirm for the last time
        - 🔙 enter the Passphrase for this key
    - ✔️🔙 enter command `save` after the `gpg>` prompt

## Generate Sub-Key for Encryption

- ⬆ start the creation of the sub-key, using command: `gpg --homedir=. --edit-key --expert qqq@example.org`
    - ⬆✔️✏️ enter command `addkey` after the `gpg>` prompt
        - ✔️✏️ enter  `12` to select `(12) ECC (encrypt only)` on question `Please select what kind of key you want:`
        - ✔️✏️ enter `1` to select `(1) Curve 25519` on question `Please select which elliptic curve you want:`
        - ✔️✏️ enter `1y` for the encryption key to last one year
        - ✔️✏️ enter `Y` on question `Is this correct? (y/N)` to confirm the printed expiry date is correct
        - ✔️✏️ enter `Y` on question `Really create? (y/N)` to confirm for the last time
        - 🔙 enter the Passphrase for this key
    - ✔️🔙 enter command `save` after the `gpg>` prompt

## Generate Sub-Key for Authenticate

- ⬆ start the creation of the sub-key, using command: `gpg --homedir=. --edit-key --expert qqq@example.org`
    - ⬆✔️✏️ enter command `addkey` after the `gpg>` prompt
        - ⬆✔️✏️ enter `11` to select `(11) ECC (set your own capabilities)` on question `Please select what kind of key you want:`
            - ✔️✏️ enter `S` to remove `Sign` from `Current allowed actions:`
            - ✔️✏️ enter `A` to add `Authenticate` to `Current allowed actions:`
            - 🔙 enter `Q` to end the selection of action
        - ✔️✏️ enter `1` to select `(1) Curve 25519` on question `Please select which elliptic curve you want:`
        - ✔️✏️ enter `1y` for the key to last one year
        - ✔️✏️ enter `Y` on question `Is this correct? (y/N)` to confirm the printed expiry date is correct
        - ✔️✏️ enter `Y` on question `Really create? (y/N)` to confirm for the last time
        - 🔙 enter the Passphrase for this certificate
    - ✔️🔙 enter command `save` after the `gpg>` prompt

## Export everything. 

```console
gpg --homedir=. --export-secret-keys    --armor qqq@example.org > PRIVATE_COMPLETE_qqq@example.org.asc
gpg --homedir=. --export-secret-subkeys --armor qqq@example.org >  PRIVATE_SUBKEYS_qqq@example.org.asc
gpg --homedir=. --export                --armor qqq@example.org >           PUBLIC_qqq@example.org.asc
```

- `...COMPLETE...asc` is a general backup of all keys
- `...SUBKEYS...asc` is to be imported into the Email-Application and later to be used to sign and decrypted emails
- `...PUBLIC...asc` is to be shared with others to encrypt emails

Based on [Kagel Blog: PGP Schlüssel generieren wie ein Experte](https://www.kagel.ch/artikel/pgp-schluessel-generieren-wie-ein-experte/)