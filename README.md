# Ransomware

[![Build Status](https://travis-ci.org/mauri870/ransomware.svg?branch=master)](https://travis-ci.org/mauri870/ransomware)

> Note: This project is purely academic, use at your own risk. I do not encourage in any way the use of this software illegally or to attack targets without their previous authorization

**The intent here is to disseminate and teach more about security in the actual world. Remember, security is always a double-edged sword**

### What is Ransomware?
Ransomware is a type of malware that prevents or limits users from accessing their system, either by locking the system's screen or by locking the users' files unless a ransom is paid. More modern ransomware families, collectively categorized as crypto-ransomware, encrypt certain file types on infected systems and forces users to pay the ransom through certain online payment methods to get a decrypt key.

### Project Summary
This project aims to build an almost functional crypto-ransomware for educational purposes, written in Go. Basically, it will encrypt your files in background using AES-256-CTR, a strong encryption algorithm, using RSA-2048 to secure the key exchange with server. Yeah, a Cryptolocker like malware.

It is composed of two main parts, the server and the malware itself.

The server is responsible for store the Id and the respective encryption key, received from the malware binary during execution.

The malware encrypt with your RSA-2048 public key a payload containing the id/enckey generated on runtime, sending then to the server, where it is properly decrypted with the respective RSA private key, and then persisted for future usage.

### Project tasks

- [x] Run in Background (or not)
- [x] Encrypt files using AES-256-CTR(Counter Mode) with random IV for each file
- [x] Without virus signature (at the moment)
- [x] Use RSA-2048 to secure the comunication with server
- [x] Stream encryption to avoid load an entire file into memory
- [ ] Link the tor library statically (It's already working on linux, any help with the cross compilation to windows will be appreciated) see [issue 3](https://github.com/mauri870/ransomware/issues/3)
- [x] Docker image for compilation

### Building the binaries

> DON'T RUN ransomware.exe IN YOUR PERSONAL MACHINE, EXECUTE ONLY IN A TEST ENVIRONMENT!

#### Docker

```
go get -v github.com/mauri870/ransomware
cd $GOPATH/src/github.com/mauri870/ransomware
# You can compile the server for windows using env GOOS=windows make instead of make
sh build-docker.sh make
```

Done! The binaries live on the bin folder

#### Local

You need Go at least 1.7

```
go get -v github.com/mauri870/ransomware
go get -v github.com/akavel/rsrc
cd $GOPATH/src/github.com/mauri870/ransomware
```

Build the project require a lot of steps, like the RSA key generation, build three binaries, embed manifest files, so, let's leave `make` do your job
```
make
```
If you like build the server for windows from a unix machine, run `env GOOS=windows make`

The malware will run in background. You can see what is going on by simply remove the `-ldflags="-H windowsgui"` from the ransomware section on Makefile before build

> DON'T RUN ransomware.exe IN YOUR PERSONAL MACHINE, EXECUTE ONLY IN A TEST ENVIRONMENT!

By default, the server will listen on `localhost:8080`. The client will use this host as the default url too.

You can put the server on any domain and start it. Simply overwrite the `SERVER_URL` constant on `client/main.go` before build and the malware will try to connect with this url instead

After build, a binary called `ransomware.exe`, `server`/`server.exe` and `unlocker.exe` will be generated on the bin folder. The execution of `ransomware.exe` and `unlocker.exe` (even if it is compiled for linux/darwin) is locked to windows machines only.

## Usage and How it Works
Feel free to edit the parameters across the files for testing.
Put the binaries on a correct windows test environment and start the server.
It will wait for the malware contact and persist the id/encryption keys

When double click on `ransomware.exe` binary it will run on background, walking interesting directories and encrypting all files that match the interesting file extensions using AES-256-CTR and a random IV, recreating then with encrypted content and a custom extension(.encrypted by default) and create a READ_TO_DECRYPT.html file on desktop

In theory, for decrypt your files you need send an amount of BTC to the attacker's wallet, followed by a contact sending your ID(located on the file created on desktop). If your payment was confirmed, the attacker possibly will return your encryption key and the `unlocker.exe` and you can use then to recover your files. This exchange can be accomplished in several ways and is not been implemented yet.

Let's suppose you get your encryption key back, you can retrieve it pointing to the following url:

```
curl http://localhost:8080/api/keys/:id
```
Where `:id` is your identification stored on the file on desktop. After, run the `unlocker.exe` by double click and follow the instructions.

And that's it, got your files back :smile:

## Server endpoints

The server has only two endpoints

`POST api/keys/add` - Used by the malware to persist new keys. Some verifications are made, like the verification of the RSA autenticity. Returns 204 (empty content) in case of success or a json error.

`GET api/keys/:id` - Id is a 32 characters parameter, representing an Id already persisted. Returns a json containing the encryption key or a json error

## The end

As you can see, building a functional ransomware, with some of the best existing algorithms is not dificult, anyone with programming and security skills can build that.
