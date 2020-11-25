---
layout: post
title: DLL Hijacking & AppLocker
summary: Figuring out how to get around AppLocker by using DLL Hijacking.
---

I wanted to take the AppLocker lab from the *Getting Started in Security* class off-road a bit. I started by asking a single question:

> How could an attacker bypass AppLocker?

A search brought me to a post from Will Dormann called [Bypassing Application Whitelisting](https://insights.sei.cmu.edu/cert/2016/06/bypassing-application-whitelisting.html). He explains how malicious code doesn't always take the form of an executable application file. So what form can it take? Let's use Windows Calculator as an example.

Windows Calculator utilizes dynamic link libraries (DLL) to function. As soon as the Calculator boots up, these libraries are called by the program. Each Windows machine has a set of dynamic link libraries (usually in the `System32` folder) that programs call upon to perform everything from DNS lookups to simple addition.

What if Windows Calculator could be tricked into running a DLL that put malware onto the machine instead of performing simple addition? Enter DLL hijacking.

## DLL Hijacking

Dormann outlines the steps of DLL highjacking in his post, but I'll recontexualize them for the lab VM.

So first let's make sure that all the AppLocker defaults are on (`Local Security Policy -> Application Control Policies -> AppLocker`) and that `Application Indentity` in Services is running. We'll then go into the  `C:\` directory and create a new folder — `asdf` for example. This is where we'll take a copy of Windows Calculator (`C:\Windows\system32\calc.exe`) and place it into the `asdf` folder.

This is where it get's interesting. Dormann recommends we install a Windows program called [Process Monitor](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon) (Procmon for short). Here's a description of what Procmon does from the Microsoft page:

> *Process Monitor* is an advanced monitoring tool for Windows that shows real-time file system, Registry and process/thread activity.

What this means is that Procmon can monitor what DLL's are called when an application runs. Let's try using it on our copied Calculator program.

Since there are _a lot_ of processes going on, we'll need to use Procmon to filter through through the noise. How can we get it to focus on our copied Calculator program? Open Procmon, select `Filter` from the navigation bar, and create two filters:

> Path contains Folder Name
>
> Process Name contains calc

Minimize Procmon and run the copied Calculator program. Let's check out our results in the Procmon window.

![Procmon Results of running the Calculator program](/images/Procmon-Results.png)

One thing that sticks out is that some DLL files come with a `NAME NOT FOUND` result. This means that Windows Calculator couldn't find those DLL's to call upon. Dormann mentions how these DLL's are the prime candidates for hijacking — we can create a malicious DLL file and rename it as one of the DLL's that the Calculator program is looking for. When the Calculator program runs, it'll call upon that malicious DLL.

 So let's take the highlighted DLL in the image, `profapi.dll`, as our candidate for DLL hijacking. What we'll do is use Dormann's suggestion, a library from Stefan Kanthak called SENTINEL.DLL ([download here](https://skanthak.homepage.t-online.de/download/SENTINEL.CAB)). Here's Dormann's description of the library (with a caveat)

> When certain exported functions are executed, this library displays a dialog that indicates the vulnerable behavior. Versions of this DLL are available for several architectures in the `SENTINEL.CAB` file provided by Stefan. As with any untrusted code, this testing should take place only in an isolated environment, such as a standalone virtual machine.

I didn't have any issues with downloading and using SENTINEL.DLL but I understand Dormann's caution. Anyways, once downloaded, we'll drag the 64-bit version of SENTINEL (should be the first DLL file in the download) into our copied Windows Calculator folder. Once there, rename it as `profapi.dll`. Your folder should look something like this:

![Calculator and DLL in folder](/images/Procmon-Results.png)

Now let's switch over to the Allowlist user and see if DLL hijacking can work. Go to the copied Calculator program folder and try to  run it. You should see something like this:

![Screen Shot 2020-11-23 at 8.23.34 PM](/images/DLL-Message.png)

This should then be followed by the normal blocked application screen:

![Screen Shot 2020-11-23 at 8.23.40 PM](/images/Blocked-App.png)

Emphasis on "should" here. Your results may vary — sometimes AppLocker blocked my attempts to run the program entirely. Other times, like the one I snapshot, the "malicious" DLL ran anyway. It goes to show that we *can* get around Application Whitelisting. As Dormann writes,

> [...] we've run an executable file, which is signed by Microsoft, but the code that executed was definitely not the expected code!

A wrinkle we can add to AppLocker to prevent this kind of attack is DLL Whitelisting. Apparently it's disabled by default because of performance issues. I guess you can't blame them if you think about it. There are a lot of DLL's that have to run through DLL whitelisting policies. That'll take time and effort, maybe enough to make a noticeable performance difference. If you want to enable DLL Whitelisting, check out [the Microsoft docs](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/ee460947(v=ws.11)?redirectedfrom=MSDN). Try to run the DLL hijacking program again and see if it executes this time.

## And More?

But DLL hijacking isn't the only way to bypass AppLocker & Application Whitelisting. Dormann mentions other viable methods including the use of [rundll32.exe](https://github.com/kasif-dekel/Microsoft-Applocker-Bypass), [regsvr32.exe](https://pentestlab.blog/2017/05/11/applocker-bypass-regsvr32/), and [Powershell](http://www.powershellempire.com/). Even if we restrict application execution to paths not writable by normal user accounts, these utilities circumvent security policies in one form or another.

I haven't tried any of these exploits out yet but y'all totally should (and let me know how it goes).

## Not Enough (By Itself)

This conclusion goes back to what John mentioned about AppLocker & Application Whitelisting in our class — you should have these utilities turned on but they shouldn't be your only line of defense. AppLocker & Application Whitelisting works great for drive-by exploits but won't stand a chance against concentrated efforts where an attacker knows what to do when she encounters these defenses. Dormann puts it well at the end of his post:

> Application whitelisting is a useful capability that can help prevent users and attackers from running unexpected applications. Application whitelisting should **not**, however, be treated as a feature that prevents malicious code from executing. Microsoft admits that "[AppLocker is not a security feature and was never designed as a security boundary](https://github.com/kasif-dekel/Microsoft-Applocker-Bypass#microsfts-response)." Other vendors may not be as forthcoming about what their application whitelisting products can and cannot practically achieve.
>
> [...]
>
> Don't consider application whitelisting to be a silver bullet. While it is a valuable part of any enterprise's protections, application whitelisting will not stop a motivated attacker.

And that's where a motivated defender comes in.
