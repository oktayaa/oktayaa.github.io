---
layout: post
title: 'dovecot/doveadm "failed:  cache message size"'
categories:
  - geeking out
tags:
  - linux
  - dovecot
  - doveadm
published: true
---

>*This one stumped me for a while and I only realized what was wrong after getting ready to submit a bug report to `dovecot`. I am putting it here since it's not a common case and it might help some people. I found no help online when it happened to me.*

Here's what the error message I encountered while using doveadm to read a message looked like.

{: .box-note}
<pre>
# doveadm fetch -u user@DOMAINNAME hdr mailbox SpamCheck/Spam uid 10
doveadm(user@DOMAINNAME): Error: Mailbox SpamCheck/Spam: UID=10: read(/var/spool/mail/virtual/DOMAINNAME/public/.Spam/cur/1583888147.M606102P10872.MAILDOMAIN,S=5367,W=5455:2,S) failed: Cached message size larger than expected (5367 > 2476, box=SpamCheck/Spam, UID=10) (read reason=mail stream)
doveadm(user@DOMAINNAME): Error: Corrupted record in index cache file /var/spool/mail/virtual/DOMAINNAME/public/.Spam/dovecot.index.cache: UID 10: Broken physical size in mailbox SpamCheck/Spam: [...]
</pre>

`dovecot` has a nice feature where its `imap` module as well as all of its components (lmtp,doveadm etc) can read and write compressed email files. You would use this if you are running low on space and want to preserve some until you can upgrade.
Although you can compress old email files too, it is not strictly necessary (and is a complicated process anyway) since dovecot is able to distinguish and read plain emails along with those that are compressed, regardless of the compression algorithm.

{: .box-warning}
With one caveat. And this is what got me. *You have to enable the compression methods that you use for all subsystems that need it. doveadm is NOT excluded from this.*

{: .box-note}
I changed all occurences of my actual domain name to `DOMAINNAME` in the following.

Let's step back a bit and start over. I first search the mailbox to get the `uid` of a message. We're looking for this in that particular user's `SpamCheck/Spam` mailbox. (We're using this public shared mailbox for `bayes` operations. I will cover public shared mailboxes with dovecot later.)

{: .box-note}
<pre>
#doveadm search -u user@DOMAINNAME mailbox SpamCheck/Spam
d1adce0f1722685e4a010000233c2ca8 10
</pre>

For the rest of this we'll concentrate on this one mail message with the `uid` discovered above.
Let's try to read its headers.

```
<pre>
# doveadm fetch -u user@DOMAINNAME hdr mailbox SpamCheck/Spam uid 10
</pre>
```
which results in a couple of error messages that repeat a few times.


<pre>
doveadm(user@DOMAINNAME): Error: Mailbox SpamCheck/Spam: UID=10: read(/var/spool/mail/virtual/DOMAINNAME/public/.Spam/cur/1583888147.M606102P10872.MAILDOMAIN,S=5367,W=5455:2,S) failed: Cached message size larger than expected (5367 > 2476, box=SpamCheck/Spam, UID=10) (read reason=mail stream)
</pre>

What's significant here is that doveadm is looking at the message size first and comparing it to what it has in the cache file for the same message. Size is denoated by the `S=` and is `5367` in this case. However dovecot claims that the size should have been `2476` according to its cache.

This particular message was compressed with `bz2` via `zlib` by dovecot when it was delivered (I used first `lda`, later `lmtp` for local delivery rather than the mechanism provided by my MTA `postfix` in order to have a single auth database provided by dovecot and backed by LDAP).

The second error message this generates is as follows.


<pre>
doveadm(user@DOMAINNAME): Error: Corrupted record in index cache file /var/spool/mail/virtual/DOMAINNAME/public/.Spam/dovecot.index.cache: UID 10: Broken physical size in mailbox SpamCheck/Spam:
</pre>

Because the size of file does not match the size for it in the cache, dovecot concludes that the message must be corrupt and it doesn't try to read it.
But is the file actually corrupt? Let's decompress it manually. While we're at it we'll actually check it's size too.


<pre>
# bzcat 1583888147.M606102P10872.MAILDOMAIN,S=5367,W=5455:2,S > /tmp/email.txt
</pre>

<pre>
# ls -alF /tmp/email.txt
-rw-r--r-- 1 root root 5367 Mar 11 02:08 /tmp/email.txt
</pre>

As you can see the file decompresses without an error and if you look at the file size, it is the same as the `S=` field in the filename.

When I actually inspected this file with `cat` I saw that it was a regular email header. It wasn't incorrect, or corrupted. I won't paste that whole thing because there's nothing interesting in it. Just a boring email header with From's and Reply To's and everything.

The real kicker is, this particular email message opened and displayed fine when using an `IMAP` client.

At one point I did have the inkling that compression support was missing somewhere but a cursory glance revealed that I had a bunch of zlib settings enabled in the config file. Besides I could actually read the emails so everything should have been configured correctly right?

{: .box-warning}
Wrong!! My zlib settings where inside particular imap, lmpt protocol blocks. These only apply to said protocols' subsystems. In order for doveadm to have support for various plugins, `zlib` in my case, those have to be declared in the global section. i.e outside of protocol blocks.

I think this was pretty dumb of me. However I haven't found any documentation about this particular error message where the headers were NOT actually corrupted.

It would be nice if doveadm documentation mentioned something at least in passing.

Well. Now you (I) know.
