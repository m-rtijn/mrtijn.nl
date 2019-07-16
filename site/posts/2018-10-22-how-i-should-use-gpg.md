---
title: How I (should) use GPG
published_date: "2018-10-22 18:56:47 +0100"
layout: blog_post.liquid
is_draft: false
data:
  published_date: 2018-10-22
  last_updated_date: 2018-10-22
---
<a href="https://en.wikipedia.org/wiki/GNU_Privacy_Guard">GPG</a>,
and the OpenPGP protocol it implements, are both quite unforgiving.
You can find plenty of best practices, usage tips and tutorials on
how to use it, and you'll find plenty of best practices that aren't
enforced by the program, bad defaults or outdated recommendations.
I have no illusion that this one of many blog posts on GPG will
change anything. I however hope that you'll maybe learn a few things
about GPG, which you can combine with abundantly available other
sources.¹

That said, I want to be clear that I won't be writing about integration
of GPG in other programs. I also assume that you already know what
GPG is and what it is used for. If you don't already, there are plenty
of posts and resources out there that will give a better introduction.
For example, the <a href="https://www.dewinter.com/gnupg_howto/english/GPGMiniHowto.html">
"GPG mini HOWTO"</a>. This blog post will solely be about how to setup
the <code>gpg</code> command line program and how to use it. I will
also only use <code>gpg</code> version 2 (version 2.2.4 to be precise).
If you don't already have <code>gpg</code> version 2 or higher on
your personal computer, I strongly suggest updating. On Ubuntu
18.04, version 2 is installed by default. On Ubuntu 16.04, version
2 is available with the <code>gpg2</code> package.

Before executing any commands, I recommend first thoroughly reading
the whole post and maybe to some further research into GPG.

So let's get to it! Before we'll even start to use <code>gpg</code>,
we are going to edit our configuration file. Create <code>~/.gnupg/gpg.conf</code>
and append these lines:
<pre>
# Use the longer key id format
keyid-format 0xlong
# Show the full key fingerprint
with-fingerprint
# Always show validity of user IDs (is default for some versions of gpg)
verify-options show-uid-validity
list-options show-uid-validity

# Set strong default preferences for new keys
default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 CAMELLIA256 AES192 CAMELLIA192 AES CAMELLIA128

# GPG will use the highest ranked preferences that are compatible with
# the recipient's key.
personal-compress-preferences ZLIB BZIP2 ZIP Uncompressed
personal-digest-preferences SHA512 SHA384 SHA256
personal-cipher-preferences AES256 CAMELLIA256 TWOFISH AES192 CAMELLIA192 AES CAMELLIA128
</pre>
The first few lines slightly changes the way <code>gpg</code> displays
information about public keys. Here's a comparison:
<pre>
# The default format
pub   rsa4096 2018-10-21 [SC]
A95EF5275370376BE3B4B37A2DCBA7C834DF7D0F
uid           [ unknown] John Doe &lt;johndoe@example.com&gt;
sub   rsa4096 2018-10-21 [E] [expires: 2020-10-21]
</pre>
<pre>
# The customized format
pub   rsa4096/0x2DCBA7C834DF7D0F 2018-10-21 [SC]
Key fingerprint = A95E F527 5370 376B E3B4  B37A 2DCB A7C8 34DF 7D0F
uid                   [ unknown] John Doe &lt;johndoe@example.com&gt;
sub   rsa4096/0x76998D8F7B22B22E 2018-10-21 [E] [expires: 2020-10-21]
</pre>
It is mostly personal preference, but it makes it easier to distinguish
between subkeys. The second part of the configuration enforces more
secure digest and cipher preferences.

Now we can make our new key pair. Since recent versions of <code>gpg</code>,
we need to explicitly state that we want to have the full featured key
generation dialog:
<pre>$ gpg --full-gen-key
gpg (GnuPG) 2.2.4; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
	(1) RSA and RSA (default)
	(2) DSA and Elgamal
	(3) DSA (sign only)
	(4) RSA (sign only)
</pre>
We will choose the default: an RSA key pair both for signing and
encryption.
<pre>
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
</pre>
Computing power is basically never a bottleneck: always choose the highest
supported keysize. Currently, that's 4096-bits. This should be secure
for a few decades.
<pre>
What keysize do you want? (3072) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
 0 = key does not expire
	&lt;n&gt;  = key expires in n days
	&lt;n&gt;w = key expires in n weeks
	&lt;n&gt;m = key expires in n months
	&lt;n&gt;y = key expires in n years
</pre>
You can always edit the expiration time of your key pairs to postpone
the expiration. It is useful to set an expiration date, so that your
key will be automatically invalidated if you ever lose your private
master key and don't have a revocation certificate to invalidate your
key immediately. We'll go for 2 years.

<pre>
Key is valid for? (0) 2y
Key expires at di 20 okt 2020 15:02:27 CEST
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Kees Test
Email address: keestest@example.com
Comment:
You selected this USER-ID:
    "Kees Test &lt;keestest@example.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
</pre>

If you use multiple email addresses, you can always add more later.
<code>gpg</code> will now ask you to for a password for your secret
key. Make sure that you use a safe password which you don't already
use for something else.²
<pre>
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

gpg: key 0xD7518A5449C9D140 marked as ultimately trusted
gpg: revocation certificate stored as: '/home/testkees/.gnupg/openpgp-revocs.d/4B08B38EA98672661D485A4AD75
public and secret key created and signed.

pub   rsa4096/0xD7518A5449C9D140 2018-10-21 [SC] [expires: 2020-10-20]
      Key fingerprint = 4B08 B38E A986 7266 1D48  5A4A D751 8A54 49C9 D140
uid                   [ultimate] Kees Test &lt;keestest@example.com&gt;
sub   rsa4096/0xFB9E92ADD9565FA8 2018-10-21 [E] [expires: 2020-10-20]
</pre>

We see here that <code>gpg</code> actually generated two key pairs
instead of one. The first key pair is your main key: it is denoted with
<code>[SC]</code>. You can use this key to sign messages or files: <code>[S]</code>,
and to sign other people their keys, create new subkeys and revoke
subkeys or the main key itself: <code>[C]</code>. A notable feature that
is missing is encryption: for this <code>gpg</code> automatically makes
a <i>subkey</i>.

A subkey is essentially a separate key pair, but it is associated with
a master key pair and used for a different purpose. As I already said,
<code>gpg</code> by default uses a subkey for encryption. I would
recommend to also use a subkey for signing, because this will allow you
to generate your master key pair on a secure offline computer and then
only export your subkeys to the computers you daily use. If your regular
computer gets compromised, the attackers still don't have your main key,
so you can safely revoke your subkeys and generate new ones. The ideal
setup is to put your subkeys on a smartcard and your main key on an
offline and secure computer. Do note however, that even though an
encryption subkey has been revoked, it can still be used to decrypt
messages that were encrypted for that specific subkey.

To create a new subkey, we will enter the <code>--edit-key</code> menu:

<pre>
$ gpg --edit-key keestest@example.com
gpg (GnuPG) 2.2.4; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  rsa4096/0xD7518A5449C9D140
     created: 2018-10-21  expires: 2020-10-20  usage: SC
     trust: ultimate      validity: ultimate
ssb  rsa4096/0xFB9E92ADD9565FA8
     created: 2018-10-21  expires: 2020-10-20  usage: E
[ultimate] (1). Kees Test &lt;keestest@example.com&gt;

gpg&gt;
</pre>
You can do a whole lot more from this menu which I won't cover here,
if you want to know more, see the man page or type <code>help</code>.
<pre>
gpg&gt; addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
Your selection? 4
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      &lt;n&gt;  = key expires in n days
      &lt;n&gt;w = key expires in n weeks
      &lt;n&gt;m = key expires in n months
      &lt;n&gt;y = key expires in n years
Key is valid for? (0) 2y
Key expires at di 20 okt 2020 16:12:39 CEST
Is this correct? (y/N) y
Really create? (y/N) y
</pre>
Then <code>gpg</code> will ask you for a password.
<pre>
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

sec  rsa4096/0xD7518A5449C9D140
     created: 2018-10-21  expires: 2020-10-20  usage: SC
     trust: ultimate      validity: ultimate
ssb  rsa4096/0xFB9E92ADD9565FA8
     created: 2018-10-21  expires: 2020-10-20  usage: E
ssb  rsa4096/0xD0E9FD5D3BEE3A31
	 created: 2018-10-21  expires: 2020-10-20  usage: S
[ultimate] (1). Kees Test &lt;keestest@example.com&gt;</pre>
We have now created our key pair, where <code>0xD7518A5449C9D140</code>
is the master key <code>[SC]</code>, <code>0xFB9E92ADD9565FA8</code>
is our encryption key <code>[E]</code>, and <code>0xD0E9FD5D3BEE3A31</code> is our
signing key <code>[S]</code>. <code>gpg</code> will automatically use
the subkey instead of the master key for signing, but if you want to
be sure, you can set the <code>default-key</code> option in your
<code>gpg.conf</code> appropriately.

To communicate securely, your contacts will need to have your public
keys:
<pre>$ gpg --armor --output keestest@example.com.asc --export keestest@example.com</pre>
You can share your public key with anyone: it can only be used to encrypt
data for you to receive and to verify the signatures you made. You may
also upload your public key to a keyserver, but watch out: this is
permanent. You can't delete anything from PGP keyservers.

You can send your public key any way you like to the people you would
like to communicate with. But, you can't immediately trust keys you've
received via any channel. You will have to either verify the key's
fingerprint via a secure channel (for example, in person), or you have
to verify this indirectly using the <a href="https://en.wikipedia.org/wiki/Web_of_trust">
Web of Trust</a>.

#### Commonly used commands

For all actions with <code>gpg</code>: the man page and a shell with
tab autocompletion for command line flags and arguments will help a
great deal.

To import your friends' keys, you can simply do:
<pre>$ gpg --import johndoe@example.org.asc</pre>
To sign and encrypt a file:
<pre>$ gpg --sign --recipient $RECIPIENT_EMAIL --encrypt $FILE</pre>
You might also want to use the <code>--armor</code> flag to output it
in OpenPGP's ASCII container format.
<br>
To backup your secret keys, or to export them to a different computer:
<pre>
# Export all secret keys, including your master key:
$ gpg --armor --output gpg_secret_keys.asc --export-secret-keys
</pre>
<pre>
# Only export the secret subkeys:
$ gpg --armor --output gpg_secret_subkeys.asc --export-secret-subkeys
</pre>

## Some final remarks

The title of this post is <i>How I (should) use GPG</i>. With an emphasis
on <i>should</i>. Creating a good GPG setup can be complex. And worst
of all, you will probably realize the mistakes you've made only when
you are using GPG a lot, after you've made your keys. It's also a rather
time-consuming and annoying process to create a new key pair and make sure
that all the people you communicate with use the new keys. You will also
need to keep the old keys around to decrypt old messages and files you
may still need. So you'll be stuck with your current keys for a while.

GPG and the OpenPGP protocol certainly aren't user-friendly. They are
also not the solution for mainstream private communications. Nonetheless,
I still think that it is useful to have GPG and communicate using it
as much as possible. Every extra encrypted message is one less sent in
plaintext. GPG is also useful for things other than email communication:
you might want to make encrypted backups or you might want to use GPG
to cryptographically sign your Git commits, to make sure that nobody
impersonates you.

*Since this post became way too long, I sadly had to cut some things out.
Most importantly, using a smartcard or a secure offline machine and
using a GPG subkey for SSH authentication. I will try to write about
these topics at another time.*

<b>Notes:</b>
<br>
¹ If you are interested in a more in-depth explanation of GnuPG, see
<a href="https://www.dewinter.com/gnupg_howto/english/GPGMiniHowto.html">
the "mini" HowTo</a> or <a href="https://www.gnupg.org/documentation/manuals/gnupg/">
the GnuPG manual</a>.
<br>
² I prefer to use <a href="http://world.std.com/~reinhold/diceware.html">
Diceware generated passphrases</a> for these kind of things: it helps
you to create easily memorizable and very strong passphrases.
