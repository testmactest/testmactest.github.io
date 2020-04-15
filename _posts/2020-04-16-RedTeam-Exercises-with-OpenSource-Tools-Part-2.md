---
title: RedTeam Exercises with OpenSource Tools part 2
author: WHmacmac
layout: post
permalink: RedTeam_Exercises_with_OpenSource_Tools_Part_2
category: blog
---

Hello again :). 
In the first <a href="https://whmacmac.github.io/RedTeam_Exercises_with_OpenSource_Tools_Part_1" style="text-decoration: none;">part</a> of my article, I showed you few basic ways for bypassing the Windows Defender and AMSI rules.
Now it is time to discuss how to apply what we learnt in the first part. <br/>

I was thinking at developing a simple malware that will make use of the multi stager feature of the Empire framework. I wanted from start that all what I will present, will fits on the real world scenarios.

<div>
<center><img src="/images/2020-04-16-RedTeam-Exercises-with-OpenSource-Tools-Part-2.md/arhitecture.png">
 </center>
</div>

<ol>
<li>Token\String\1,2</li>
<li>Whitespace\1</li>  
<li>Encoding\1</li>  
<li>Compress\1</li>  
</ol>
