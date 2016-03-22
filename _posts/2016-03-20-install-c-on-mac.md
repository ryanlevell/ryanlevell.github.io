---
layout: post
title:  "Installing C On A Mac"
date:   2016-03-20 23:03:03 -0500
categories: hacking c mac keychain homebrew gdb codesign gcc
---

### Intro
I am doing the steps below in order to follow along in the book [Hacking The Art Of Exploitation](http://www.amazon.com/Hacking-The-Art-Exploitation-Edition/dp/1593271441).

### Install needed software via HomeBrew
{% highlight bash linenos %}
~ $: brew install gcc
~ $: brew install gobjdump

# access repos that duplicate software (like gdb) provided by OS X
# but more recent/bugfixed versions
~ $: brew tap homebrew/dupes
~ $: brew install gdb
{% endhighlight %}

### Fixing gdb by adding to Mac's Keychain Access

My experience running gdb right after installing:
{% highlight bash linenos %}
# the -q switch means quiet, don't print introductory/copyright messages
~ $: gdb -q ./a.out
~ $: break main
~ $: run
Starting program: /Users/ryan/Documents/programming/c/hacking/a.out 
Unable to find Mach task port for process-id 83012: (os/kern) failure (0x5).
 (please check gdb is codesigned - see taskgated(8))
{% endhighlight %}

As the command mentions, `gdb` needs codesigned.  
This is needed because `gdb` must have special permissions to control other processes.  
The steps below can be found at [gcc.gnu.org](https://gcc.gnu.org/onlinedocs/gcc-4.8.1/gnat_ugn_unw/Codesigning-the-Debugger.html).

1. Open `Keychain Access`
2. Go to `Keychain Access > Certificate Assistant > Create a Certificate...`
3. Update the Name, Identity Type, Certificate Type, and override checkbox accordingly:
 * ![keychain cert info](/assets/keychain_screenshot_1.png)
4. Click continue multiple times until you see the image below. System MUST be chosen.
 * ![keychain cert location](/assets/keychain_screenshot_2.png)
5. Verify the certificate is on the System by looking at all certficates in Keychain Access
 * ![keychain cert list](/assets/keychain_screenshot_3.png)

#### Do the codesign
{% highlight bash linenos %}
which gdb
codesign -s gdb-cert <path to gdb>
{% endhighlight %}

### NOTES
I don't remember why, but all taskgated processes may need killed:
{% highlight bash linenos %}
sudo killall taskgated
{% endhighlight %}

If you make a mistake while creating the cert, such as creating it in `login` instead of `System`, you can delete and replace it using the `-f` switch.
{% highlight bash linenos %}
# after deleting and recreating in Keychain Access
codesign -f -s gdb-cert <path to gdb>
{% endhighlight %}
