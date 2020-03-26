---
layout: post
title: dovecot doveadm failed: cache message size
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
# doveadm fetch -u user@***DOMAINNAME** hdr mailbox SpamCheck/Spam uid 10
doveadm(user@***DOMAINNAME**): Error: Mailbox SpamCheck/Spam: UID=10: read(/var/spool/mail/virtual/***DOMAINNAME**/public/.Spam/cur/1583888147.M606102P10872.MAILDOMAIN,S=5367,W=5455:2,S) failed: Cached message size larger than expected (5367 > 2476, box=SpamCheck/Spam, UID=10) (read reason=mail stream)
doveadm(user@***DOMAINNAME**): Error: Corrupted record in index cache file /var/spool/mail/virtual/***DOMAINNAME**/public/.Spam/dovecot.index.cache: UID 10: Broken physical size in mailbox SpamCheck/Spam: read(/var/spool/mail/virtual/***DOMAINNAME**/public/.Spam/cur/1583888147.M606102P10872.MAILDOMAIN,S=5367,W=5455:2,S) failed: Cached message size larger than expected (5367 > 2476, box=SpamCheck/Spam, UID=10)
doveadm(user@***DOMAINNAME**): Error: Mailbox SpamCheck/Spam: UID=10: read(/var/spool/mail/virtual/***DOMAINNAME**/public/.Spam/cur/1583888147.M606102P10872.MAILDOMAIN,S=5367,W=5455:2,S) failed: Cached message size larger than expected (5367 > 2476, box=SpamCheck/Spam, UID=10) (read reason=)
doveadm(user@***DOMAINNAME**): Error: fetch(hdr) failed for box=SpamCheck/Spam uid=10: Mailbox SpamCheck/Spam: UID=10: read(/var/spool/mail/virtual/***DOMAINNAME**/public/.Spam/cur/1583888147.M606102P10872.MAILDOMAIN,S=5367,W=5455:2,S) failed: Cached message size larger than expected (5367 > 2476, box=SpamCheck/Spam, UID=10) (read reason=)
</pre>

`dovecot` has a nice feature where its `imap` module as well as all of its components (lmtp,doveadm etc) can read and write compressed email files. You would use this if you are running low on space and want to preserve some until you can upgrade.

Although you can compress old email files too, it is not strictly necessary (and is a complicated process anyway) since dovecot is able to distinguish and read plain emails along with those that are compressed, regardless of the compression algorithm.

{: .box-warning}
With one caveat. And this is what got me. *You have to enable the compression methods that you use for all subsystems that need it. doveadm is NOT excluded from this.*

{: .box-note}
I changed all occurences of my actual domain name to ***DOMAINNAME*** in the following.

Let's step back a bit and start over. I first search the mailbox to get the `guid` of a message. We're looking for this in that particular user's `SpamCheck/Spam` mailbox. (We're using this public shared mailbox for `bayes` operations. I will cover public shared mailboxes with dovecot later.)

{: .box-note}
<pre>
#doveadm search -u user@\***DOMAINNAME\*** mailbox SpamCheck/Spam
d1adce0f1722685e4a010000233c2ca8 10
</pre>

For the rest of this we'll concentrate on this one mail message with the guid discovered above.
Let's try to read its headers.

{: .box-note}
<pre>
# doveadm fetch -u user@***DOMAINNAME** hdr mailbox SpamCheck/Spam uid 10
doveadm(user@***DOMAINNAME**): Error: Mailbox SpamCheck/Spam: UID=10: read(/var/spool/mail/virtual/***DOMAINNAME**/public/.Spam/cur/1583888147.M606102P10872.MAILDOMAIN,S=5367,W=5455:2,S) failed: Cached message size larger than expected (5367 > 2476, box=SpamCheck/Spam, UID=10) (read reason=mail stream)
doveadm(user@***DOMAINNAME**): Error: Corrupted record in index cache file /var/spool/mail/virtual/***DOMAINNAME**/public/.Spam/dovecot.index.cache: UID 10: Broken physical size in mailbox SpamCheck/Spam: read(/var/spool/mail/virtual/***DOMAINNAME**/public/.Spam/cur/1583888147.M606102P10872.MAILDOMAIN,S=5367,W=5455:2,S) failed: Cached message size larger than expected (5367 > 2476, box=SpamCheck/Spam, UID=10)
doveadm(user@***DOMAINNAME**): Error: Mailbox SpamCheck/Spam: UID=10: read(/var/spool/mail/virtual***DOMAINNAME**/public/.Spam/cur/1583888147.M606102P10872.MAILDOMAIN,S=5367,W=5455:2,S) failed: Cached message size larger than expected (5367 > 2476, box=SpamCheck/Spam, UID=10) (read reason=)
doveadm(user@***DOMAINNAME**): Error: fetch(hdr) failed for box=SpamCheck/Spam uid=10: Mailbox SpamCheck/Spam: UID=10: read(/var/spool/mail/virtual/***DOMAINNAME**/public/.Spam/cur/1583888147.M606102P10872.MAILDOMAIN,S=5367,W=5455:2,S) failed: Cached message size larger than expected (5367 > 2476, box=SpamCheck/Spam, UID=10) (read reason=)
</pre>
