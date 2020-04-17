---
title: RedTeam Exercises with OpenSource Tools part 2
author: WHmacmac
layout: post
permalink: RedTeam_Exercises_with_OpenSource_Tools_Part_2
category: blog
---

Hello again :).<br/> 
It is time to learn new things and to apply what we learnt in the first <a href="https://whmacmac.github.io/RedTeam_Exercises_with_OpenSource_Tools_Part_1" style="text-decoration: none;">part</a> of my article where I showed few ways for bypassing the Windows Defender and the AMSI rules. <br/>

## Contents
* [Introduction](#shortintro)

## Introduction {#shortintro}

I was speaking about a RedTeam exercise in my first article, am I right? What RedTeam Exercise is that where we don't make use of a 'lil phishing mail? <br/>
I decided to deliver my payload to our vicim using a phishing mail. Lets speak a bit about how our phishing mail will look, how I was thinking its arhitecture, what difficulties I am expecting to encounter during its delivery or execution.
Based on the below scheme, I explain how I thought each phase:
<div>
<center><img src="/images/2020-04-16-RedTeam-Exercises-with-OpenSource-Tools-Part-2.md/arhitecturef.png">
 </center>
</div>
<ol>
<li>Phase 1 and 2: They are referring to successful delivery the email in the user's inbox. Depending on the type of infrastructure I am targeting, I expect to encounter a mail protection with the ability to detect and eventualy detonate my attachment in a sandbox. We have to take the proper measures to avoid being detected by the security gateway and in case we are, then we should make sure to instruct the our payload to not start the execution process in the analysis machine. In case I am flagged by a security protection, I will have to remake my payload.</li> 
<li>Phase 3: It implies our social engineering skills. I have to make the user to enable the macro or execute it through a tricky mechanism (example: macro in excel fields). Since Microsoft Office is the most used email client, I will focus only on its security protections. Microsoft has developed a sandbox mode which is a security feature that prevents access from running certain expressions that could be unsafe.</li>
<li>Phase 4: If I made the user to enable the macros or to do whatever method I thought to execute it in background, my script will check if it is running in a sandbox and in case it does, then it will stop the execution. The VBs script beside the sandbox checkings, it contains the code for loading the dropper in memory. As a process tree, it will look like the following: "Office" process -> VBs interpreter -> Powershell.</li>
<li>Phase 5: The dropper will contact a decoy server for downloading the first stager in memory. I have chosen to make use of a decoy server in my malware arhitecture for covering up the posibility to have blocked my primary C2 by a SOC team.</li>  
<li>Phase 6: My decoy server that is hosting the first stager will send it to the victim.</li>  
<li>Phase 7,8,9,10: The stager 0 is the launcher, the part that I have to obfuscate it. When it is successfully run, it will reach the Empire server, asking for . The C2 server will respond with the "stage 1" that does all of my key negotiation and finally it loads the Empire agent that enables all of my command and control functionalities. If you prefer, you can take a look at the scheme of how Empire agents are working <a href="https://testmactest.github.io/RedTeam_Exercises_with_OpenSource_Tools_Part_1#howdoimakeuseofopensource" style="text-decoration: none;">here</a></li>
</ol>




I was thinking at developing a simple malware that will make use of the multi stager feature of the Empire framework. I wanted from start that all what I will present, will fits on the real world scenarios.


References:<br/>
https://support.office.com/en-us/article/Turn-sandbox-mode-on-or-off-to-disable-macros-8CC7BAD8-38C2-4A7A-A604-43E9A7BBC4FB
