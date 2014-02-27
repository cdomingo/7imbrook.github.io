---
layout: post
title: "How simple a bug can be."
tagline: ""
description: "Apple recently released a security patch that raised some concerns. What was it and what does this mean for you."
category: articles
tags: [bug, apple, ssl, security]
image:
  feature: cover_secuity.jpg
  credit: Michael Timbrook | Apple Inc
  creditlink: http://timbrook.im
---
```cpp
goto fail;
```
I'm sure you've all heard by now that Apple released a security update to iOS 7 (7.0.6) addressing an issue with SSL certificate validation in the Security framework. If you haven’t updated your software, do so now. OS X Mavericks got it’s update today (Feb 25). This vulnerability puts all information sent from your machine at risk to man in the middle attacks. The security information can be found [here](http://support.apple.com/kb/HT6147).

> What's Wrong?

The bug discovered affects the verification process of public keys used to encrypt information. The final step is skipped at can not return its error if a key is invalid. If you are unfamiliar with how public and private key-pairs work, here’s a quick run down: Your private key is used to decrypt data encrypted by its paired public key. Simple, but how do you know that the public key you are using isn’t some random key that someone in the middle has the private key for? This is why we have Certificate Authorities, or CA’s. CA’s are trusted “people” that sign public keys. Signing a key identifies it as an authentic, unmodified key that the private key is only owned by the public key distributor. Anyone can become a CA, but becoming trusted by a wide number of companies is difficult. This creates a trust that the data we are encrypting can only be read by the ones we intend.

> Ok, but is my data still being encrypted?

The short answer, yes. The truth behind that answer is that it’s encrypted, but you don’t know who can decrypt it. So what happened? Like I mentioned earlier the last step in the key verification process is being skipped. Here’s the code responsible for the bug. There’s an extra goto that skips the last step of the process:

```cpp
...
 if ((err = SSLFreeBuffer(&hashCtx)) != 0)
        goto fail;

    if ((err = ReadyHash(&SSLHashSHA1, &hashCtx)) != 0)
        goto fail;
    if ((err = SSLHashSHA1.update(&hashCtx, &clientRandom)) != 0)
        goto fail;
    if ((err = SSLHashSHA1.update(&hashCtx, &serverRandom)) != 0)
        goto fail;
    if ((err = SSLHashSHA1.update(&hashCtx, &signedParams)) != 0)
        goto fail;
        goto fail;
    if ((err = SSLHashSHA1.final(&hashCtx, &hashOut)) != 0)
        goto fail;
...
fail:
    SSLFreeBuffer(&signedHashes);
    SSLFreeBuffer(&hashCtx);
    return err;
```
The full source can be found over [here](http://opensource.apple.com/source/Security/Security-55471/libsecurity_ssl/lib/sslKeyExchange.c).

First impressions of this code might be “Are those gotos?”, and yes they are. I’ve always strayed away from the use of gotos in code, mainly because they can be unsafe. That’s why when people ask me about gotos, I go straight to no. Or maybe I’ll check a condition first and see if this is really a good way of doing this. Truth is, this is an efficient way of handling this. It propagates the error code up through the return. It’s common practice in C to have functions return their error codes and almost every call is checked to make sure it didn’t error. The simple case looks like this:

```cpp
int err = 0;
if ((err = myFunction(params)) != 0) {
	return err;
}
```

Ok so if the function returns an error we return that error, simple. Now look at this case:

```cpp
MyStruct *obj = malloc(sizeof(MyStruct));
int err = 0;
if ((err = opperationOne(obj) != 0) {
	free(obj);
	return err;
}
if ((err = opperationTwo(obj) != 0) {
	free(obj);
	return err;
}
...
```

We still want to propagate the right error, but now we need to do some memory cleanup before we do so. This example is simple but you can see when this expands that it would be cumbersome to keep track of all the clean up. Thats where this goto pattern for errors comes in:

```cpp
int function(...) {
    int err = 0;
    if ((err = _operation1(&dataBlock) != 0) goto error;
    if ((err = _operation2(&dataBlock) != 0) goto error;
    if ((err = _operation3(&dataBlock) != 0) goto error;
    ...
    if ((err = _operationN(&dataBlock) != 0) goto error;
    
error:
    freeBlock(dataBlock);
    return error; 
}
```
If execution flows perfectly through the function then the return is 0 and the necessary memory cleanup has been done. I prefer to do the inline way because it’s more readable and less prone to putting a goto in the wrong place. What happened with the security framework was probably a copy paste error, and the extra goto got added. The unfortunate part about this is that it wasn’t caught. XCode’s static analyser can usually catch deadcode, but with the default warnings it won't catch this. What’s worse than that is there a simple regression test would have caught that, and if it was in place, this all could have been avoided.

> What have we learned from this?

The moral of the story kids is that testing is important, because simple mistakes like duplicate lines don’t always throw compiler warnings or fail builds. They can alter behavior in a way that leaves you open down the road.
