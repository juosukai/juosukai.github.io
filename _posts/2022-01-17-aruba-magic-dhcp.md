---
layout: post
title: Aruba "magic" VLAN and DHCP
comments: True
published: True

---

We are running a fairly complicated wireless environment to support the honestly silly amount of phones we see at the office every day. My predecessors had invested quite a bit in Aruba Access Points connected to Aruba Central, and I have had no reason to change vendor.

For our use (extremely high density environment, often 150+ devices per AP) the AP-555 models have been extremely useful, as we can get 2 separate 5Ghz radios serving clients, effectively doubling the capacity of each AP.

When setting up a new site last year I decided to cut some corners and instead of setting up proper VLANs in and office/guest/production networks, I just set up one office VLAN, had all the APs sit in that VLAN and the office SSID was also connected to the same. For the production network I  used the "Instant AP assigned" and "Internal VLAN" options when setting up the SSID. I did not bother with the guest network at all as the site started off with just a handful of people, they could as well sit on the office network (we are completely in the cloud with our internal services, access to our office network does not giv eyou access to any DCs or other important services). This felt pretty safe as the devices on these networks just need the external internet connection, no services from our internal network.

![image](/public/aruba_vlan.png)

This option is actually pretty handy for many situations: it creates a new VLAN (always 3333) between the APs, tunnels traffic between the APs and one AP (the conductor) acts as the DHCP server for all the devices. This means that you can build a walled off VLAN without needing to build a single trunk or VLAN on the switch/fw side. 

Things actually worked pretty darn well for a year. But as we grew from 10 people to 300, we had to move peoples personal devices from the office network to a guest network. So we create a guest network, point that to use the same VLAN setup as the production network, and everything works. I made sure that we will not run out of leases by using a sille /21 subnet for the production and guest network (2048 addresses).

Early this year we hit another problem: the number of DHCP timeouts was growing very quickly, especially with the evening shift. I checked the AP handing out the addresses and since there were only ~1000 addresses leased out: (run the following command over ssh CLI, copypaste the resulting into a code editor, see number of last row)

{% highlight ini %}
accesspoint-04# sh dhcp-allocation

---------------------/etc/dnsmasq.conf--------------------
listen-address=127.0.0.1
addn-hosts=/etc/ld_eth_hosts
addn-hosts=/etc/ld_ppp_hosts
dhcp-src=172.16.8.1
dhcp-leasefile=/tmp/dnsmasq.leases
dhcp-authoritative
aruba-raw-socket-mode
#magic-vlan
{
	vlan-id=3333
	dhcp-range=192.168.8.3,192.168.15.254,255.255.248.0,30m
	dhcp-option=1,255.255.252.0
	dhcp-option=3,172.168.8.1
	dhcp-option=6,1.1.1.1
}

---------------------/tmp/dnsmasq.leases------------------
1642440491 62:ec:95:8c:26:1f 192.168.9.249 3333 * xx:xx:xx:xx:xx:01
1642440491 7e:77:96:a7:a1:69 192.168.10.17 3333 * xx:xx:xx:xx:xx:02
1642440481 e6:61:e6:ac:91:75 192.168.10.128 3333 * xx:xx:xx:xx:xx:03
1642440437 fe:57:c9:98:db:a6 192.168.11.226 3333 * xx:xx:xx:xx:xx:04
1642440439 e2:87:6c:46:63:aa 192.168.10.29 3333 * xx:xx:xx:xx:xx:05
1642440429 12:03:7c:0c:82:9e 192.168.10.20 3333 * xx:xx:xx:xx:xx:06

{% endhighlight %}

After spending several weeks looking into possible ethernet level issues, AP level issues etc, I finally found the culprit:

{% highlight bash %}

Jan 17 17:05:21   dnsmasq-dhcp[28779]: DHCPREQUEST(br0) 192.168.11.158 xx:xx:xx:xx:xx:xx
Jan 17 17:05:21   dnsmasq-dhcp[28779]: DHCPNAK(br0) 192.168.11.158 xx:xx:xx:xx:xx:xx no leases left
Jan 17 17:05:21   dnsmasq-dhcp[28779]: Vlan id: 3333

{% endhighlight %}

Hmm. We cannot be running out of leases since the subnet has 2048 addresses but that is what the error says. And sure enough, a quick google search brought me to the following [comment](https://lists.thekelleys.org.uk/pipermail/dnsmasq-discuss/2015q3/009871.html):


{% highlight ini %}

Dnsmasq has a global limit on the number of leases in use at any time,
and it's that limit you're hitting, not running out of addresses.


The default is 1000 leases, you can increase it by putting

dhcp-lease-max=2000

,or whatever, in the config file.

{% endhighlight %}

So we were hitting the limits of the dnsmasq service Aruba is using to create the "magic" VLAN. Turns out the only real fix to our situation is building proper VLANs and handing off DHCP duties to a proper server (or firewall, as is more likely). 