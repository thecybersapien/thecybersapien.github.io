---
layout: post
title: 'OSINTEGO - DEF CON Delhi Group | DC9111 2021 CTF'
tags: [DC9111, DEF CON, CTF]
color: rgb(50,180,50)
permalink: osintego-writeup/
author: shivamsaraswat
excerpt_separator: <!--more-->
---

<!-- <center><img src="/assets/img/feature-img/vulnhub.png" alt="Machine Information"></center> -->
<br>
**Challenge Category: OSINT**

**Challenge:**
The author is a professional person and has a medium to run his own blog. He likes the white colour the most. Can you spot the flag?
<br>Author: cybersapien

<!--more-->

<hr>

<h1 id="contents-">Contents <a name="top"></a></h1>
* TOC
{:toc}

<hr>

## Overview

This challenge includes OSINT and Steganography, that’s why its name is **OSINTEGO**.<br>
I created this challenge as part of DC9111 2021 CTF.

<hr>

<!-- ## Strategy to Solve

- Network Scanning (arp-scan, Nmap)
- Take root access and capture the flag -->

<!-- <hr> -->

## How to Solve 

1). "*Professional*" means a professional platform which means LinkedIn and author name is given in the challenge, so go to LinkedIn of author - **[https://www.linkedin.com/in/cybersapien/](https://www.linkedin.com/in/cybersapien/)**

2). We can see 3 blogs links in the featured section.
<center><img src="/assets/img/ctf/dc9111-2021/featured-section.png" alt="Featured Section"></center>

3). "*own blog*" means blog hosted on own domain, so go to **[cybersapien.tech](https://cybersapien.tech/)**.

4). See the source code of the web page. 
<center><img src="/assets/img/ctf/dc9111-2021/source-code.png" alt="Featured Section"></center>

5). "*white colour*" means White Space Steganography. Select all the text and we can see a weird thing that a long blank line selected as a text.

6). Copy this line, and paste it in Sublime Text or VS code editor.

7). Select the pasted text, this will highlight the text.
<center><img src="/assets/img/ctf/dc9111-2021/selected-text.png" alt="Featured Section"></center>

8).	We can see tabs (arrow) and spaces.

9).	Tabs and spaces can be converted to binary, tabs to 0 and spaces to 1, using the following Python script- 

```
pasted_text = "	 				  	  	    	  	   		  		   	   		 		  				 	   	 			   	 	 	  	  			  				 	   	 			  	 		 	  	    	  	   		   		  		 				 		 				 		 				 				 	 		 	  		 	  	    	   	 	 		 						  		   	  	    	   	 				 						  	 		 	   	 				 						   		 		  	 		 	  		   	  	 				   	 				 	   					 	 		 		 				  		 	 	   		 		  		 	 		 						  	 		 	   		  		 						    		 	  	    	   	 	 	   		 			 						  		  		  	  			  				 	  		   		 	  	 				 	 		  		 			  			  	 	     		   		 		  			 		  			 		  			 	    	  	 	  		 	 		    	 	 	 	 	 	     	 					 	 	 		 		 			 	 	 	     	 		    	 	 		  	 		 		 	 		   		 	 	 			 	     	 		  	 	 					 	 	 		  	 	 	 			 			 	 	 	 		 		     	 "
print(pasted_text.replace('\t', '0').replace(' ', '1'))
```
<center><img src="/assets/img/ctf/dc9111-2021/python-script.png" alt="Featured Section"></center>

10). We will get a text containing 0's and 1's, we can convert it to ASCII using website - **[https://www.convertbinary.com/to-text](https://www.convertbinary.com/to-text/)**.
<center><img src="/assets/img/ctf/dc9111-2021/convert-to-text.png" alt="Featured Section"></center>


<hr>

This is the flag.<br>
This confirms that we have successfully completed the challenge.

<hr>

**I hope, this post helped you to solve this CTF easily and you must have learned something new.**

**Feel free to [contact me](/contact/) for any suggestions and feedbacks. I would really appreciate those.**

Thank you for reading!

You can also **[Buy Me A Coffee](https://www.buymeacoffee.com/cybersapien)** if you love the content and want to support this blog page!

<script type="text/javascript" src="https://cdnjs.buymeacoffee.com/1.0.0/button.prod.min.js" data-name="bmc-button" data-slug="cybersapien" data-color="#FFDD00" data-emoji=""  data-font="Cookie" data-text="Buy me a coffee" data-outline-color="#000000" data-font-color="#000000" data-coffee-color="#ffffff" ></script>

<a href="#top" style="float: right"><strong>Back to Top⮭</strong> </a>