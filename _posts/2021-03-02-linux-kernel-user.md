---
layout: post
title: Brad Spengler on API to move data between user and kernel space
date: 2021-03-02
permalink: /:title/
description: "Notes on Linux copy_*_user()"
tags: [short]
published: true
share: true
---

Bradley Spengler on "Why do we use copy_to_user()/copy_from_user in kernel code?":


{:.boxit}
Under certain conditions, you could get away with not using such an API -- Windows for instance lacks (lacked?) a defined interface for this (given its historical x86 focus). For that reason, they're not able to enable modern security functionality like SMAP. Linux has long supported architectures like SPARC with separate user/kernel address spaces which requires special code to access userland; a simple memcpy or direct memory dereference won't suffice.
Memcpy or directly memory dereference also wouldn't be possible by virtue of the fact that the Linux kernel isn't allowed to access paged out memory except under code covered by an exception table entry (which the userland-accessing assembly of copy*user does have).
One other advantage to copy_from_user in particular is that it greatly reduces the possibility of "double-fetch" bugs being introduced. In the Windows case where there is/was a lack of an API for this, userland accesses are performed by direct dereference under an exception block (to handle any page faults that occur). The compiler however is free there to fetch certain values from userland multiple times, non-obvious at the source code level. (continued)
When a value is obtained from userland, sanity checked, but then later obtained again from userland, an attacker could have modified that value from another thread, likely resulting in a security vulnerability. copy_from_user ensures that the data is all copied in one fell swoop, so that the kernel is only operating on a guaranteed in-kernel copy of the userland data.
Many years ago, there were bugs in the upstream kernel and in out-of-tree drivers where copy*user APIs were not used (often, mistaking a userland pointer for a kernel one and assuming it could be dereferenced directly). At the time, there was no SMAP on x86, so these issues went unnoticed until the introduction of PaX's UDEREF functionality in 2006. This exposed the problems so that they were eventually fixed long before the introduction of SMAP 8 years later where direct access to paged-in userland memory would cause a fault.
Since userland generally provides the pointer to its own data, it's also a common operation that the kernel needs to ensure the data actually resides in userland, otherwise userland could trick the kernel via copy_from_user into copying its own data and creating an information leak, or via copy_to_user into corrupting its own data.  The access_ok() call within copy*user ensures that the provided userland pointer is appropriate for the given use.  Without copy_to_user/copy_from_user, you would need to perform these checks separately, resulting in error-prone code (in the Windows world, you'd have ProbeForRead calls splattered all over).
Post-Spectre, the kernel also adds a speculation barrier prior to accessing userland.  

Additional reading:
- [http://www.osronline.com/article.cfm%5earticle=514.htm](http://www.osronline.com/article.cfm%5earticle=514.htm)
- [https://grsecurity.net/~spender/uderef.txt](https://grsecurity.net/~spender/uderef.txt)
- [https://en.wikipedia.org/wiki/Supervisor_Mode_Access_Prevention](https://en.wikipedia.org/wiki/Supervisor_Mode_Access_Prevention)
- [https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/arch/x86/include/asm/uaccess.h?id=9ad9f45b3b91162b33abfe175ae75ab65718dbf5#n476](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/arch/x86/include/asm/uaccess.h?id=9ad9f45b3b91162b33abfe175ae75ab65718dbf5#n476)
- [https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/arch/x86/include/asm/uaccess.h?id=9ad9f45b3b91162b33abfe175ae75ab65718dbf5#n401](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/arch/x86/include/asm/uaccess.h?id=9ad9f45b3b91162b33abfe175ae75ab65718dbf5#n401)

