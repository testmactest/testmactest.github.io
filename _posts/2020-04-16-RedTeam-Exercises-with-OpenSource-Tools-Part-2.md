---
title: RedTeam Exercises with OpenSource Tools part 2
author: WHmacmac
layout: post
permalink: RedTeam_Exercises_with_OpenSource_Tools_Part_2
category: blog
---

Hello again :).<br/> 
In the first <a href="https://whmacmac.github.io/RedTeam_Exercises_with_OpenSource_Tools_Part_1" style="text-decoration: none;">part</a> of my article, I showed you few ways for bypassing the Windows Defender and AMSI rules.
Now it is time to learn new things and to apply what we learnt in the first part. <br/>

## Contents
* [Introduction](#shortintro)

## Introduction {#shortintro}

I was speaking about a RedTeam exercise in my first article, am I right? What RedTeam Exercise is that where we don't make use of a 'lil phishing mail? <br/>
I decided to deliver my payload to our vicim using a phishing mail. Lets speak a bit about how our phishing mail will look, how I was thinking its arhitecture, what difficulties I am expecting to encounter during its delivery/execution:
Based on the scheme that I presented below, I will explain how I thought each phase:
<ol>
<li>Phase 1 and 2: they are referring to successful delivery the email in the user's inbox. Depending on the type of infrastructure we are targeting, I expect to encounter a mail protection with the ability to detect and eventualy detonate my attachment in a sandbox. We have to take the proper measures to avoid being detected by a security gateway and in case we are, then we should make sure to not start the execution process in the analysis machine. In case I am flagged by a security protection I will have to remake my payload.</li> 
<li>Phase 3: it implies our social engineering skills. I have to make the user to enable the macro or execute it through a tricky mechanism (macro in excel fields). Since Microsoft Office is the most used email client, I will focus only on its security protections which apply to other email clients too. Microsoft has developed a sandbox mode which is a security feature that prevents access from running certain expressions that could be unsafe. I risk at this step not to running my macros because I didn't convince the user enough to trigger macros or because of Office's protections. </li>
<li>Phase 4: If I made the user to enable the macros or to do whatever method I thought to execute it in background, my script will check if it is running in a sandbox and in case it does, then it will stop the execution. The VBs script beside the sandbox checkings, it contains the code for loading the dropper in memory. As a process tree, it will look like the following: "Office" process -> VBs interpreter -> Powershell.</li>
<li>Phase 5: The dropper will contact a decoy server for downloading the first stager. In this way ife are catched, </li>  

<li>Phase 7:</li>  
<li>Phase 8:</li>  
</ol>

<div>
<center><img src="/images/2020-04-16-RedTeam-Exercises-with-OpenSource-Tools-Part-2.md/arhitecture.png">
 </center>
</div>



I was thinking at developing a simple malware that will make use of the multi stager feature of the Empire framework. I wanted from start that all what I will present, will fits on the real world scenarios.


References:<br/>
https://support.office.com/en-us/article/Turn-sandbox-mode-on-or-off-to-disable-macros-8CC7BAD8-38C2-4A7A-A604-43E9A7BBC4FB
