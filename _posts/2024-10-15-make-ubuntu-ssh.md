---
layout: post
title: Ubuntu 24.04 sshd settings to work with make.com
comments: True
published: True

---

We use [make.com](https://www.make.com) for a lot of automation tasks across the company, especially in automating HR account work. One common thing I do is use make.com to call [GAM](https://github.com/GAM-team/GAM/wiki) running on a linux server on GCP.

I recently noticed that updating Debian from 11 to 12 broke all these connections as Debian deprecated the old default ssh-rsa key exchange method for accessing the servers with keys. I am all for improving security, but I was unable to get the ssh keys using the ed25519 ciphers that are standard on Debian now. This is because make.com needs the private key in a pem encoded format, and it seems like the command 
{% highlight ini %}
ssh-keygen -p -m PEM -f .ssh/id_ed25519
{% endhighlight %}
just does not create a key that make.com understands.

In the end, the only way to get this working was adding these rows to the normal sshd config file (/etc/ssh/sshd_config):

{% highlight ini %}
# Enable old key encryption from make.com
PubkeyAcceptedKeyTypes=+ssh-rsa
HostKeyAlgorithms +ssh-rsa
{% endhighlight %}