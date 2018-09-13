---
layout: post
title:  "Automated DNS Record mapping for cloud instances"
date:   2013-06-02
categories: cloud
tags: cloud dns vps
---

# instance-dns

Cloud VPSs are everywhere. Amazon EC2, Google CE, DigitalOcean, Upcloud, and the hundreds listed on serverbear and lowendbox.
Commonly, when the instance is launched, it is assigned an IP address from the providers pool. This IP address is usually 
obtainable from the control pabel of the provider, and sometimes (but not always) from the providers API. The trouble is, 
most providers do not offer a way to automatically map the cloud instance name to its IP through a custom DNS record. EC2 does create 
a DNS A record under the ec2.aws subdomain, but it also does not offer a way to setup a record like instance_name.custom_domain.com. 

This is why I wrote the simple simple `instance-dns` script. You can get it from [github.com/abgoyal/instance-dns](//github.com/abgoyal/instance-dns).


## How does it work?

Very simply, instance-dns provides a boot-time script (run thru cron with the `@reboot` timespec) that calls the API of a supported DNS provider 
(currently, only [Zonomi](//www.zonomi.com) is supported, but support for Amazon Route 53, Zerigo, PointDNS etc, is forthcoming).

When the instance boots, the `HOSTNAME` environment is set by the virtualization software based on the instance name configured by user. `instance-dns` 
maps the hostname as a subdomain of the configured custom domain (see the project readme for details of the configuration options).

Note that this is not dynamic dns and thus simply installing ddclient will not work. Specifically:

- Dynamic DNS providers rarely (never?) provide API-based means of creating a _new_ host record. They are 
  mostly concerned with remapping the A/AAAA record of a pre-existing, manually created host record.
- The IP of a cloud instance is not dynamic. It does not change while the instance stays up. Running a deamon like ddclient is wasteful
  and unnecessary. Some dynamic dns providers even consider API calls where the IP doesn't change as abusive.

I spent a lot of time researching the various DNS providers out there. My criteria were:

- Must have at least a minimal free tier, and the free tier should include API access.
- API must be capable of creating new host records

Zonomi was the simplest, cleanest ones from the ones I found. Zerigo and PointDNS also look promising. There are others - Route 53, Cloudflare DNS, Namecheap, etc
which do have the API capabilities, but do not provide any free tiers at all, or free tiers lack API access.

## Using it

Just follow the installtion instructions on the project page, update your custom domain in the config file, and update your API key.

I personally use it with my digitalocean account, and used it with upcloud previously. 
