---
layout: post
title:  "Building the Android kernel for Galaxy Nexus"
date:   2013-05-28 03:01:19
categories: android development
tags: gnexus android
---

The Android/Linux kernel shipped with Android 4.2.2 release on the Galaxy Nexus (and presumably other devices) does not include the cifs.ko module. I like mounting
all my samba shares to my android phone as that gives me access to all my media and documents without having to worry about syncing or running out of disk space on 
my phone.

Fortunately, Google releases full kernel source code for the Nexus devices. This makes it easy to build the kernel ourselves, enabling whatever modules one needs. 
Once the kernel and modules are built, its a simple matter of transferring the .ko files for the modules we need to the phone and "insmod" them into the kernel.

There are quite a few tutorials online demonstrating how to build a custom kernel from scratch. Unfortunately, most of them focus on replacing the entire kernel. 
This not being my intent, it took a bit of trial and error to figure our the right way to build a _specific_ version of the kernel, matching the one shipped with the
phone exactly. This allows insmoding the kernel modules directly from our build.

This here is a detailed transcript of what needs to be do, derived mainly from http://source.android.com/source/building-kernels.html.

Firstly, we note that the code name for the GSM Galaxy Nexus is "maguro". Thus, its binaries are in device/samsung/tuna and kernel source is in kernel/omap.

Lets start by making a new working directory:

{% highlight bash %}

mkdir cifsbuild
cd cifsbuild



Now we can download and install the prebuilt toolchain:

{% highlight bash %}

git clone https://android.googlesource.com/platform/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6
export PATH=$(pwd)/arm-eabi-4.6/bin:$PATH

export ARCH=arm
export SUBARCH=arm
export CROSS_COMPILE=arm-eabi-

{% endhighlight %}


Next, get the kernel binaries. This is needed to obtain the specific commit id of the kernel release:


{% highlight bash %}

git clone https://android.googlesource.com/device/samsung/tuna
cd tuna/
git log --max-count=1 kernel

{% endhighlight %}

This generates output like so (example only):

{% highlight bash %}

commit c262ac6b02f84e25e98b0d5f74bb2f20ab37db1b
Author: Todd Poynor <toddpoynor@google.com>
Date:   Wed Nov 28 12:21:52 2012 -0800

    tuna: prebuilt kernel (security fix)
    
    9f818de mm: Hold a file reference in madvise_remove
    
    Bug 7517474
    
    Change-Id: Id03f6b8e328643925bef7e848978e442e24cef24

{% endhighlight %}

Make a note of the 7-digit commit id, here it is 9f818de.

Go back to the working directory, get the kernel source tree and checkout the correct commit:

{% highlight bash %}

cd ..

git clone https://android.googlesource.com/kernel/omap.git
cd omap

git checkout 9f818de

{% endhighlight %}

This completes the setup. We can now simply configure the kernel as per our needs:

{% highlight bash %}

make tuna_defconfig
cp .config .config_default

# make changes and save
make menuconfig

{% endhighlight %}

For example the changes made were:

{% highlight bash %}

diff .config_default .config

{% endhighlight %}

{% highlight diff %}

 2426c2426,2437
 < # CONFIG_NETWORK_FILESYSTEMS is not set
 ---
 > CONFIG_NETWORK_FILESYSTEMS=y
 > # CONFIG_NFS_FS is not set
 > # CONFIG_NFSD is not set
 > # CONFIG_CEPH_FS is not set
 > CONFIG_CIFS=m
 > # CONFIG_CIFS_STATS is not set
 > # CONFIG_CIFS_WEAK_PW_HASH is not set
 > # CONFIG_CIFS_XATTR is not set
 > # CONFIG_CIFS_DEBUG2 is not set
 > # CONFIG_NCP_FS is not set
 > # CONFIG_CODA_FS is not set
 > # CONFIG_AFS_FS is not set
 2488c2499
 < # CONFIG_NLS_UTF8 is not set
 ---
 > CONFIG_NLS_UTF8=m
 2669c2680
 < # CONFIG_CRYPTO_MD4 is not set
 ---
 > CONFIG_CRYPTO_MD4=m

{% endhighlight %}


Now simply build the kernel, and extract the errant modules:

{% highlight bash %}

make

# copy the modules to another place for transfer to your phone

cp fs/cifs/cifs.ko fs/nls/nls_utf8.ko crypto/md4.ko /tmp/cifs_modules

{% endhighlight %}


