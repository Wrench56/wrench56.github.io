+++
title = "Rant: Why does POSIX not provide any Crypto Utils"
date = 2026-05-31
+++

## Background

Readers of my code and/or my blog will probably know that I am trying to always follow POSIX specification, especially when writing Shellscripts, to be as compatible with other systems as possible. This is due to the fact that I am using multiple platforms, particularly many Linux distributions and FreeBSD, along with some embedded systems. It is rather convenient when my automations run on each and every platform I run on. Today, I was/am writing a tool for storing and distributing patches, [PatchVault](https://github.com/Wrench56/patchvault-template), when I encountered the issue of distributing [FreeBSD-like distinfos](https://github.com/freebsd/freebsd-ports/blob/main/editors/2bsd-vi/distinfo).

## The problem

The POSIX standard does not provide any ways of creating cryptographically safe hashes. The only utility which comes close to helping one distribute distinfos is [cksum(1)](https://pubs.opengroup.org/onlinepubs/9799919799/utilities/cksum.html#top). Please note, that there are no flags provided in the specification at all!

Now, some *BSD or Linux users reading have probably used one of the members of the [shasum(1) utility family](https://man.freebsd.org/cgi/man.cgi?query=shasum&sektion=1&manpath=freebsd-release-ports). Conveniently, it provides the option to generate a file containing the file paths and their hashes. Amazing tool! You can quickly generate SHA256 hashes for a tree and then just check against it using the `-c/--check` flag, providing similar functionality to [mtree(8)](https://man.freebsd.org/cgi/man.cgi?mtree(8)).

Let's take a look at a few (and by far not all) popular systems and see if they provide a unified checksum/hashsum checkfile interface!

### FreeBSD

A similar (albeit incompatible) format is used for distinfo files on the FreeBSD ports tree. Namely, [sha256](https://man.freebsd.org/cgi/man.cgi?query=sha256). It generates an output like (for `sha256 <file>`) `SHA256 (.sh_history) = 3071da2af8b82b4aeb9a5a40c497ceb5ef608f35034cf72987fad994c4ee2727`. Technically, Linux does not have any tool (by default) that produces the same output. Using [shasum(1)](https://perldoc.perl.org/shasum) you can get a similar output file, but most definitely not the same format. Linux `shasum(1)` does not note the used algorithm.

### Linux

Now, do you still remember `cksum(1)`? Well, technically, `cksum -a sha256 <file>` on Linux does produce the 1:1 output from above. Great! Two systems, two tools, one output. Neither of the above are a POSIX compatible way of implementing what I want of course. But I am not a purist. If systems unanimously changed a tool to do something beyond POSIX, I am happy to use that. So let's see what other systems do:

### QNX

Of course, as a POSIX certified OS, QNX does have [cksum(1)](https://www.qnx.com/developers/docs/6.5.0SP1.update/com.qnx.doc.neutrino_utilities/c/cksum.html). It even provides some extensions, like the `-v` flag... No way to check against an output file, so that is a no-go. Ergo, QNX does not provide a way to check against a file of sha256 sums by default.

### AIX

Well, let's see what AIX did. Of course, we have [cksum(1)](https://www.ibm.com/docs/en/aix/7.3.0?topic=c-cksum-command) again. Well, no SHA256 extension here either. That's a bummer. No check either.

### z/OS

"Funnily" enough, z/OS provides a different [cksum(1)](https://www.ibm.com/docs/en/zos/3.2.0?topic=descriptions-cksum-calculate-display-checksums-byte-counts) implementation compared to AIX. Still, no check, on SHA256.

## Solutions?

Well, it is about 2AM here, so it is about time I wrap this up and finish what I wanted in my initial project, lol! Well, turns out, we have two separate ways of solving this problem (none of which I like much).

### 1st: OpenSSL

Albeit slightly differently formatted (whitespaces do not matter when checking, so beyond visual differences there isn't any), OpenSSL does give you the same format as FreeBSD's distinfo files! You have to do something along the lines of `openssl dgst -sha256 <file>` to see it! Nice! The above systems **should** support OpenSSL so this is a success! (Well, technically, I do not believe OpenSSL would be recommended on z/OS due to their in-house cryptography solutions.)

Of course, for a simple patch applying script, why do I have to have a third-party (and rather heavy) utility/library?

### 2nd: Good old POSIX...

Well, you can always count on [cmp(1)](https://pubs.opengroup.org/onlinepubs/009604499/utilities/cmp.html)! Just simply generate a CRC32 file using `cksum(1)` and compare a locally created checksum file against the remotely fetched one. Of course I would recommend to [sort(1)](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/sort.html) it beforehand! (Just to avoid strangeness in file ordering)

## So what?

Well, The question is rather "Why?". Why does POSIX not give a way of generating SHA256 (or any other cryptographically safe hashes) out of the box? What is holding them back from making `shasum(1)` standard?

Clearly, they are reluctant to "standardize" cryptography. This is evident from [crypt(3)](https://pubs.opengroup.org/onlinepubs/9799919799/functions/crypt.html#top).

> The algorithm is implementation-defined.

Also, they don't quite force this on you, as you have the `ENOSYS` escape hatch:

> [ENOSYS]
>   The functionality is not supported on this implementation.

So what is the reason behind this?

Well, the Arch Linux manpages for [crypt(3)](https://man.archlinux.org/man/crypt.3.en#PORTABILITY_NOTES) states:

> Due to historical restrictions on the export of cryptographic software from the USA, crypt is an optional POSIX component.

Aha! Politics. Should have guessed it :D
Unfortunate, that because of this, we do not have a unified cryptographic interface, something rather important in my opinion.

Until IEEE POSIX decides to clean up this archaic state, I am stuck with modifying the `cksum(1)` output into something like:

```
CRC32 (<file>) <hash>
```

so that it goes well with SHA256:

```
SHA256 (<file>) <hash>
CRC32  (<file>) <hash>
```

This ensures that systems that don't have the SHA256 distinfo-looking output AND OpenSSL can just use `cksum(1)` and not just fail. Of course, it doesn't solve the problem. At least there is some error detection. Meh...
