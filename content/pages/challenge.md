---
title: "The Challenge"
date: 2017-12-10T19:02:39-05:00
type: page
tags: 
categories:
draft: false
---

A few years ago, I stumbled across a post by IConrad on Reddit.  If you've been around [r/sysadmin](https://reddit.com/r/sysadmin) long enough, you'll know exactly which post I'm talking about:  "How do I learn to be a Linux sysadmin?"

I thought, "hey, I'm a Linux sysadmin!" and then realized that I haven't yet had exposure to certain technologies. So I'm challenging myself go gain that exposure, through as many configurations as possible.

## The short version

While I've touched on and worked with many of these services in my day-to-day work, I want to review, document, and share my knowledge. There are some topics I'm strong at--and some I only know by name.

Here's the goal:

- [ ] Hypervisors (KVM, VMWare, HyperV)
- [ ] DHCP
- [ ] DNS
- [ ] LDAP/Active Directory
- [ ] Provisioning (Foreman, SCCM, Vagrant, Docker)
- [ ] System and application monitoring (Grafana, Nagios, Sensu, Prometheus, SCOM)
- [ ] Log aggregation (Graylog, ELK, SCOM)
- [ ] Database servers (MySQL, Postgres, MongoDB), standalone and clustered
- [ ] Web servers (nginx, IIS), standalone and clustered
- [ ] Additional web services (reverse proxying, caching with Varnish)
- [ ] Data storage (NFS, HDFS, LUNs)
- [ ] Data analysis stack (Hadoop)
- [ ] Containerized computing (Docker, Kubernetes)
- [ ] Application deployment
- [ ] Backups
- [ ] Documentation

I want to build out a test lab that I can use as a guinea pig for new content, so the hope is to publish it all in Git as a series of configuration-managed, infastructure-as-code, testing-included playbooks in Ansible, Salt, Puppet, etc., so that interested parties can experiment along with me.

I'm pretty OS-agnostic, so it's likely this infrastructure will be done in both Windows and Linux where possible.

## The longer version

The general gist is as follows:

```markdown
This is what I tell people to do, who ask me "how do I learn to be a Linux sysadmin?".
1) Set up a KVM hypervisor.
2) Inside of that KVM hypervisor, install a Spacewalk server. Use CentOS 6 as the distro for all work below. 
(For bonus points, set up errata importation on the CentOS channels, so you can properly see security update 
advisory information.)
3) Create a VM to provide named and dhcpd service to your entire environment. Set up the dhcp daemon to use 
the Spacewalk server as the pxeboot machine (thus allowing you to use Cobbler to do unattended OS installs). 
Make sure that every forward zone you create has a reverse zone associated with it. Use something like 
"internal.virtnet" (but not ".local") as your internal DNS zone.
4) Use that Spacewalk server to automatically (without touching it) install a new pair of OS instances, with 
which you will then create a Master/Master pair of LDAP servers. Make sure they register with the Spacewalk 
server. Do not allow anonymous bind, do not use unencrypted LDAP.
5) Reconfigure all 3 servers to use LDAP authentication.
6) Create two new VMs, again unattendedly, which will then be Postgresql VMs. Use pgpool-II to set up 
master/master replication between them. Export the database from your Spacewalk server and import it into 
the new pgsql cluster. Reconfigure your Spacewalk instance to run off of that server.
7) Set up a Puppet Master. Plug it into the Spacewalk server for identifying the inventory it will need to
 work with. (Cheat and use ansible for deployment purposes, again plugging into the Spacewalk server.)
8) Deploy another VM. Install iscsitgt and nfs-kernel-server on it. Export a LUN and an NFS share.
9) Deploy another VM. Install bakula on it, using the postgresql cluster to store its database. Register each 
machine on it, storing to flatfile. Store the bakula VM's image on the iscsi LUN, and every other machine on 
the NFS share.
10) Deploy two more VMs. These will have httpd (Apache2) on them. Leave essentially default for now.
11) Deploy two more VMs. These will have tomcat on them. Use JBoss Cache to replicate the session caches 
between them. Use the httpd servers as the frontends for this. The application you will run is JBoss Wiki.
12) You guessed right, deploy another VM. This will do iptables-based NAT/round-robin loadbalancing between 
the two httpd servers.
13) Deploy another VM. On this VM, install postfix. Set it up to use a gmail account to allow you to have 
it send emails, and receive messages only from your internal network.
14) Deploy another VM. On this VM, set up a Nagios server. Have it use snmp to monitor the communication 
state of every relevant service involved above. This means doing a "is the right port open" check, and a 
"I got the right kind of response" check and "We still have filesystem space free" check.
15) Deploy another VM. On this VM, set up a syslog daemon to listen to every other server's input. 
Reconfigure each other server to send their logging output to various files on the syslog server. 
(For extra credit, set up logstash or kibana or greylog to parse those logs.)
16) Document every last step you did in getting to this point in your brand new Wiki.
17) Now go back and create Puppet Manifests to ensure that every last one of these machines is authenticating 
to the LDAP servers, registered to the Spacewalk server, and backed up by the bakula server.
18) Now go back, reference your documents, and set up a Puppet Razor profile that hooks into each of these 
things to allow you to recreate, from scratch, each individual server.
19) Destroy every secondary machine you've created and use the above profile to recreate them, joining them 
to the clusters as needed.
20) Bonus exercise: create three more VMs. A CentOS 5, 6, and 7 machine. On each of these machines, set 
them up to allow you to create custom RPMs and import them into the Spacewalk server instance. Ensure 
your Puppet configurations work for all three and produce like-for-like behaviors.
Do these things and you will be fully exposed to every aspect of Linux Enterprise systems administration. 
Do them well and you will have the technical expertise required to seek "Senior" roles. If you go 
whole-hog crash-course full-time it with no other means of income, I would expect it would take 
between 3 and 6 months to go from "I think I'm good with computers" to achieving all of these -- assuming 
you're not afraid of IRC and google (and have neither friends nor family ...).
```
(Copied from ["How did you get your start?](https://www.reddit.com/r/linuxadmin/comments/2s924h/how_did_you_get_your_start/cnnw1ma/?st=jb1eh9d2&sh=c1c310ee))

There have been a few useful updates over the years, most recently by [Mogwire](https://www.reddit.com/r/linuxadmin/comments/67yufq/updated_iconrads_this_is_what_i_tell_people_to_do/dgutjfv/?st=jb1fs86f&sh=d2cb7c83):

```markdown
I think people need to remember his response:
> This is what I tell people to do, who ask me "how do I learn to be a Linux sysadmin?".
And his response was perfect. If you want to update it then it would only make a few changes / additions.
1. Start with a CentOS 7 server as the base for all servers.
2. Replace Spacewalk with The Foreman; Spacewalk is the upstream of Satellite 5 and that is going EOL.
3. Since Foreman has Puppet built into it you no longer need to install a Puppet Master but you can and reconfigure 
Foreman to use that.
4. You can also skip the Puppet Razor server and use Foreman do to Bare Metal and VM provisioning.
5. Configure BareOS instead of Bacula and setup all VMs to use Relax and Recover (REAR) for backups/recovery.
6. As for Docker, you can setup a container to run a GitLab instance and a Postgres instance. Once configured to 
work properly define them as a service in a Compose File and utilize docker compose.
7. Install and configure Zabbix Server to monitor all servers. Configure Zabbix for Auto Discovery of all Linux 
Servers. Add Zabbix-Agents as a Product in Foreman. Configure all Servers to have Zabbix Repo. Write Ansible 
Playbook / Puppet Manifest to make sure Zabbix Agent is installed on all Servers and zabbix_agentd.conf is configured 
to point to Zabbix Server and configured to work with Auto Discovery.
8. Setup syslog server and write Puppet Manifest / Ansible Playbook to point all servers to syslog server.; Bonus 
would be to setup a Logstash server.
9. Install Grafana and configure Zabbix as a source to monitor server performance and alerts.
You can also include Ansible with basic Playbooks to do updates, software installs, etc. Utilize the Gitlab instance 
to store Puppet Manifests, Ansible Playbooks, code snippets, etc.
```

So, here's the challenge. In part, this site will be about the quest to fulfill some of this setup. Not all, most likely. Probably also using different components. But I'd like to go through and document and experiment with these options as much as possible, through different technologies, platforms, and methodologies.

This will be a way to explore infrastructure as code, testing new technologies, documenting things I've learned and things I intend to learn, and sharing it all. Along the way, we'll touch on Windows servers, Linux servers, book reviews, open source technologies, closed source technologies, and why DNS is the cause of all problems.




