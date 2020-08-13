---
title: Shorten ssh
date: 2016-05-21T17:22:12+01:00
layout: archive
---

Every time we do operation on remote machine need specify hostname , user and `.pem`
Example

```bash
ssh -i your-pem-file.pem ubuntu@abc.compute.amazonaws.com
```

We can create shorten name for above, so that we don't have to remember long machine name, user and pem file location.

Example

```bash
ssh abcAws
#abcAws` is shorten for `-i your-pem-file.pem ubuntu@abc.compute.amazonaws.com`
```

**How ?**

1. Add shorten in `~/.ssh/config` file

```bash
Host abcAws
HostName ubuntu@abc.compute.amazonaws.com
User ubuntu
IdentityFile your-pem-file.pem
```

That's it Enjoy :)
