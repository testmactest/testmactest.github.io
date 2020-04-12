---
title: RedTeam Exercises with OpenSource Tools
author: WHmacmac
layout: post
permalink: RedTeam_Exercises_with_OpenSource_Tools
category: blog
---

Can RedTeam exercises be done today only using open source tools and do they have 100% yield/success? Can I affirm that I can compete with the top solutions in a red team vs blue team engagement, blackbox or whitebox exercise, using only open source tools and having the same result as the enterprise solutions like: Canvas, Metasploit Pro, Open Core, Cobaltstrike ?

In the series that I will approach, I will present ways to compete with the top security research undertaken by companies focused on security products and how we can combine and customize multiple open source sollutions for achieving the same result. 


## Contents
* [Short Introduction](#shortintro)
* [What is AMSI?](#whatisamsi)
* [How do I make use of opensource](#howdoimakeuseofopensource)
* [Code Obfuscation](#codeobfuscation)

## Short Introduction {#shortintro}
Because it is less probable to encounter a Linux based system with a security solution in an enterprise environment, I have chosen to focus on how we can compromise a Windows based system up to date (11.04.2020 - the date I am writing this article) having all of the security modules enabled.

We can see a number of measures implemented in Microsoft's operating system that have no role other than to provide a greater protection system for user protection:
<ul>
  <li>Windows Defender</li>
  <li>Antimalware Scan Interface (AMSI)</li>
  <li>Control flow guard</li>
  <li>Data Execution Prevention (DEP)</li>
  <li>Randomized memory allocations</li>
  <li>Arbitrary Code guard (ACG)</li>
  <li>Block child processes</li>
  <li>Simulated Execution (SimExec)</li>
  <li>Valid stack integrity (StackPivot)</li>
</ul>          
I do not want to advertise Microsoft products but as I focus on proving we still can bypass their sollutions, I can not say that they didn't do a good job regarding thier security solutions (maybe they are intending to sell security services like many others if they already have not started). One security sollution from Microsoft I encountered more than I intended to, and I burned my arsenal in development or pentests before finding out what stopped me and why, I am pretty sure you encountered it too even if Windows Defender was not the main AV/AntiMalware solution from your target. I think you got about what i am speaking of :).
                    

## What is AMSI? {#whatisamsi}
As you guessed, I was reffering at AMSI; before starting how to make use of the open source tools for a red team exercise, we have to bypass Microsoft's security modules, this meaning we have to bypass AMSI too. 
Before getting into the methods of bypassing AMSI, we need to clarify a little about what AMSI is, under what principles AMSI works and how it has the ability to catch even the most exotic payloads.

<div>
<center><img src="/images/2020-04-11-RedTeam-Exercises-with-OpenSource-Tools/amsi.png">
 </center>
</div>

AMSI is an interface that allows to the OS's applications and services to integrate with any antimalware product that's present on a machine. It supports a calling structure allowing for a file and memoy or stream scanning, content source URL/IP reputation checks and other techniques. It supports also the notion of a session so different antimalware vendors can correlate different scan requests, this being pretty bad for our stagers if they are catched by AMSI.
  
As we can see in the above image, AMSI is integrated by default in the following Windows' components meaning that for example we have a powershell script containing some commands, they will be analyzed based on some string patterns and in case they something match, then Windows Defender/ Microsoft's Sandbox will enter in action:

<ul>
  <li>User Account Control, or UAC (elevation of EXE, COM, MSI, or ActiveX installation)</li>
   <li>PowerShell (scripts, interactive use, and dynamic code evaluation)</li>
   <li>Windows Script Host (wscript.exe and cscript.exe)</li>
   <li>JavaScript and VBScript</li>
   <li>Office VBA macros</li>
</ul>
                    
As things were not enough bad, Microsoft improve the logging capabilities of Windows Defender so we have the risk to alarm the whole network of our presence if there is a SOC team.

<div>
<center><img src="/images/2020-04-11-RedTeam-Exercises-with-OpenSource-Tools/amsi_detection.png">
 </center>
</div>

{% highlight powershell %}
> Get-WinEvent ‘Microsoft-Windows-Windows Defender/Operational’ -MaxEvents 1 | Where-Object Id -eq 1116 | format-list

{% endhighlight %}

As we can see Windows Defender already logged our activity of trying to execute our payload in memory. As source of detection is menitoned AMSI. This can be easily automated in a SIEM to trigger an alert and all of our efforts of getting on the workstation would be useless.

AMSI may provide a result between 1 and 32767. The larger the result, the riskier it is to continue with the content. These values are provider specific, and may indicate a malware family or ID. Any return result equal to or larger than 32768 is considered malware,  and the result is blocked. A list from Microsoft docs will explain each range of values what means:
<ul>
  <li>AMSI_RESULT_CLEAN  - Known good. No detection found, and the result is likely not going to change after a future definition update.</li>
   <li>AMSI_RESULT_NOT_DETECTED - No detection found, but the result might change after a future definition update.</li>
   <li>AMSI_RESULT_BLOCKED_BY_ADMIN_START - Administrator policy blocked this content on this machine (beginning of range).</li>
   <li>AMSI_RESULT_BLOCKED_BY_ADMIN_END - Administrator policy blocked this content on this machine (end of range).</li>
   <li>AMSI_RESULT_DETECTED - Detection found. The content is considered malware  and should be blocked.</li>
</ul>

According to Microsoft docs “results within the range of AMSI_RESULT_BLOCKED_BY_ADMIN_START and AMSI_RESULT_BLOCKED_BY_ADMIN_END values (inclusive) are officially blocked by the admin specified policy. In these cases, the script in question will be blocked from executing. The range is large to accommodate future additions in functionality”.

This looks bad until now but courage, there is hope :).


## How do I make use of open source? {#howdoimakeuseofopensource}
As a penetration tester, I have to prioriteze what I want to complete first for having a prepared arsenal for any penetration test. Because time is precious to every one of us, we have to move fast: 
<ol>
<li>Get a working base code first from Empire, Metasploit, etc., would be ideally instead of reinventing the whell (reinventing the whell is a good thing if you are a malware developer - but this is a story for other article :) ).</li>
<li>Customize functions - change default parameters/functions names, delete comments, write the functionality in other way, etc.</li>  
<li>Obfuscate code.</li>  
<li>Test against AV - Make sure you disable the option for permitting to Microsoft to upload your sample in their cloud sandbox. This will just waste your precious payload.</li>  
</ol>

So this being said, i have chosen to go on the BC SECURITY's solution, the reborn Empire (epic music should be played on at this phrase).

<b>Why Empire and not other solution?</b><br/>
At this moment Empire is a robust and mature framework for post-exploitation, the guys from BC-Security really did a good job after they forked the official unsupported project. I personally, when it comes to chose a platform, i prefer a solution that can be easily used by my team mates/ eventually colaborate all together on it through an API/ web interface. Empire has both API and web interface for multiple user colaboration.<br/>
Few pros why i have chosen Empire and you should go for it too, in case you are the clasic guy who works with classic tools:
<ul>
<li>Encrypted C2 channels and multiple protocols/services supported for communications.</li>
  <li>Adaptable modules: .bat, .vbs, .ddl, etc.</li>
  <li>Alot of evasion capabilities enabled by default.</li>
</ul>

Even if a lot of research articles are mention that powershell for redteams is no longer used by APTs as main tool of exploitation, i do not see their point. Powershell for redteams is a great tool if you are using it just for a legit pentest and nothing more:
<ul>
  <li>Operated in memory.</li>
  <li>Installed by default on Windows systems</li>
  <li>Full .NET access.</li>
  <li>Direct access to Win32 API.</li>
  <li>Admins typically leave it enabled.</li>
  </ul>

Below is a scheme of how Empire stager is deployed.
<div>
<center><img src="/images/2020-04-11-RedTeam-Exercises-with-OpenSource-Tools/stager.png">
 </center>
</div>
Okay so we completed the first step. I have chosen a base code to use as our main tool of attack. However default Empire will get me caught. It can not pass the AMSI protection of Windows system so obfuscation or changes are needed.


## Code Obfuscation {#codeobfuscation}
Because we have chosen powershell as our main programming language for malware developing, I will present few ways of how can we can obfuscate our code.

<b>1.Randomized Capitalization</b><br/>
Powershell is ignoring capitalization. Every antivirus/antimalware solution is heavily dependent upon signatures, this including signatures based on hashes.
<div>
<img src="/images/2020-04-11-RedTeam-Exercises-with-OpenSource-Tools/write-host.png">
</div>
Even if AMSI is ignoring capitalization, changing the payload's hash is a good practice.


<b>2.Concatenation</b><br/>
AMSI is still heavily dependent upon signatures, however simple concatenation can prevent most alerts. Microsoft has implemented a custom EICAR string 'amsicontext' for testing the AMSI's detection capabilities.
<div>
<center><img src="/images/2020-04-11-RedTeam-Exercises-with-OpenSource-Tools/concat.png">
 </center>
</div>


<b>3.Variable Insertion</b><br/>


<b>4.Format String</b><br/>