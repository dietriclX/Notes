# Quick Test on Algorithm supported by Mobile APPs

I wanted to know, if there are limitations on KeePass apps. Something that KeePass2 is capable of. I was focusing on mobile apps freely available, which might fit my needs.

Unfortunately my prefered app has a general problem with Argon2id :-(

BTW
*Why on earth is the key file in an XML format?*


## Test Results

| Test Nr. | Application | Version | Mobile OS | AES/Rijndael & AES-KDF | ChaCha & AES-KDF | AES/Rijndael & Argon2d | ChaCha & Argon2d | AES/Rijndael & Argon2id | ChaCha & Argon2id | 
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | 
| 1 | AuthPass | 1.9.11 | Android 10 | Ok | Ok | Ok | Ok | Failed | Failed |
| 2 | AuthPass | 1.9.10 | iOS 15 | Ok | Ok | Ok | Ok | Failed | Failed |
| 3 | KeePass2Android | 1.09e-r7 | Android 10 | Ok | Ok | Ok | Ok | Ok | Ok |
| 4 | KeePassium | v1.46.140 | iOS 15 | Ok | Ok | Ok | Ok | Ok | Ok |


## Test Scenario

1. start KeePass compatible client app
2. open local file ("Database?.kdbx")
3. select key file ("2.0_256bits.keyx")
4. enter password ("qqq")
5. unlock/open database
6. lock&close database

repeat steps 2. - 6. for all databases


## Test Preparation

The following databases were created using "KeePass Password Safe" Version 2.47 (64-bit) on Linux.

For each database, ...

- Master Password: qqq
- Database file compressed with GZip
- Key File "2.0_256bits.keyx" (created with the same application, as Version 2.0 with 256bits)


### The important properties ... similarities/differences

#### Password File "DatabaseA.kdbx"
- Database file encryption algorithm: AES/Rijndael (256-bit key, FIPS 197)
- Key derivation function: AES-KDF
	- Interations: 60000

#### Password File "DatabaseB.kdbx"
- Database file encryption algorithm: ChaCha (256-bit key, RFC 7539)
- Key derivation function: AES-KDF
	- Interations: 60000

#### Password File "DatabaseC.kdbx"
- Database file encryption algorithm: ChaCha AES/Rijndael (256-bit key, FIPS 197)
- Key derivation function: Argon2d
	- Interations: 2
	- Memory: 64 MB
	- Parallelism: 2

#### Password File "DatabaseD.kdbx"

- Database file encryption algorithm: ChaCha AES/Rijndael (256-bit key, FIPS 197)
- Key derivation function: Argon2d
	- Interations: 2
	- Memory: 64 MB
	- Parallelism: 2

#### Password File "DatabaseE.kdbx"
- Database file encryption algorithm: ChaCha AES/Rijndael (256-bit key, FIPS 197)
- Key derivation function: Argon2id
	- Interations: 2
	- Memory: 64 MB
	- Parallelism: 2

#### Password File "DatabaseF.kdbx"
- Database file encryption algorithm: ChaCha AES/Rijndael (256-bit key, FIPS 197)
- Key derivation function: Argon2id
	- Interations: 2
	- Memory: 64 MB
	- Parallelism: 2



## Addendum

Email attachment "app.log.txt" of failed test AuthPass on iOS

```code
:
<date-/time-stamp> FINER format_utils - Initialized with locale de
<date-/time-stamp> FINER keyfile - Decoded base64 of keyfile.
<date-/time-stamp> FINER kdbx_bloc - {type: FileSourceLocal, uuid: 1c589f15-82b0-4a0f-b5f6-204a1b36c069, databaseName: null, displayPath: file:///<path>/Test/DatabaseE.kdbx, filePickerIdentifier: {"identifier":"<a very long string>","persistable":"true","uri":"file:///<path>/Test/DatabaseE.kdbx","fileName":"DatabaseE.kdbx"}, local: internal} got from FileContentSource.memoryCache
<date-/time-stamp> FINER kdbx_bloc - reading kdbx file ...
<date-/time-stamp> FINER kdbx.header - Reading version: 4.0
<date-/time-stamp> FINEST kdbx.header - Reading header HeaderFields.CipherID (2) (size: 16)}
<date-/time-stamp> FINEST kdbx.header - Reading header HeaderFields.CompressionFlags (3) (size: 4)}
<date-/time-stamp> FINEST kdbx.header - Reading header HeaderFields.MasterSeed (4) (size: 32)}
<date-/time-stamp> FINEST kdbx.header - Reading header HeaderFields.KdfParameters (11) (size: 139)}
<date-/time-stamp> FINEST kdbx.header - Reading header HeaderFields.EncryptionIV (7) (size: 16)}
<date-/time-stamp> FINEST kdbx.header - EndOfHeader HeaderFields.EndOfHeader
<date-/time-stamp> FINEST kdbx.format - Header hash matches.
<date-/time-stamp> FINEST kdbx_var_dictionary - Reading VarDictionary 1.0
<date-/time-stamp> WARNING kdbx_bloc - Error while reading kdbx file.
### KdbxCorruptedFileException: KdbxCorruptedFileException{message: Invalid KDF UUID <24 characters>}
#0      KeyEncrypterKdf.kdfTypeFor.<anonymous closure> (package:kdbx/src/crypto/key_encrypter_kdf.dart:86)
#1      KeyEncrypterKdf.kdfTypeFor (package:kdbx/src/crypto/key_encrypter_kdf.dart:87)
#2      KeyEncrypterKdf.encrypt (package:kdbx/src/crypto/key_encrypter_kdf.dart:93)
#3      KdbxFormat._computeKeysV4 (package:kdbx/src/kdbx_format.dart:704)
#4      KdbxFormat._loadV4 (package:kdbx/src/kdbx_format.dart:561)
#5      KdbxFormat.read (package:kdbx/src/kdbx_format.dart:467)
#6      KdbxBloc.readKdbxFile (package:authpass/bloc/kdbx_bloc.dart:569)
#7      KdbxBloc._openFileContent (package:authpass/bloc/kdbx_bloc.dart:368)
#8      KdbxBloc.openFile (package:authpass/bloc/kdbx_bloc.dart:334)
<asynchronous suspension>
#9      _CredentialsScreenState._tryUnlock (package:authpass/ui/screens/select_file_screen.dart:1159)
<asynchronous suspension>
#10     _CredentialsScreenState.build.<anonymous closure> (package:authpass/ui/screens/select_file_screen.dart:1120)
<asynchronous suspension>

<date-/time-stamp> FINE authpass.select_file_screen - Unable to open kdbx file. 
### KdbxCorruptedFileException: KdbxCorruptedFileException{message: }
#0      KdbxBloc._openFileContent (package:authpass/bloc/kdbx_bloc.dart:377)
<asynchronous suspension>
#1      KdbxBloc.openFile (package:authpass/bloc/kdbx_bloc.dart:334)
<asynchronous suspension>
#2      _CredentialsScreenState._tryUnlock (package:authpass/ui/screens/select_file_screen.dart:1159)
<asynchronous suspension>
#3      _CredentialsScreenState.build.<anonymous closure> (package:authpass/ui/screens/select_file_screen.dart:1120)
<asynchronous suspension>
```
