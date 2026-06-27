# HANDS-ON ASSIGNMENT #1:  PENETRATION TESTING LAB SET UP
## Pen Testing Lab Setup: Penetration  Testing with Kali Linux

- Version 3.0
- Latest Revision:  5/10/2026

## Overview (you have extra time)
- You will do this assignment individually on your local PC, but you will collaborate with your group for help. 
- The goal of this lab if to set up a Pen Testing environment on your local PC using Kali Linux running in a Virtual Machine.  
- There are many ways to get through this lab, but the procedure in the book is a starting point. 
- The procedure in the book is based on using Virtual Box running locally on your PC.
- Virtual box support on the Mac has become questionable with Apple Silicon.  For the Mac, you may need to use a different Type 2 Hypervisor, like VM Ware.  
- You set up a minimul version of the lab described in the book.  You will just need to set up one Kali box for penetration testing and another box that you will attack. 

- Here is an article on using Virtual Box on the Mac if you want to try that: 
[You Can Now Run VirtualBox on Apple Silicon M1/M2](https://osxdaily.com/2022/10/22/you-can-now-run-virtualbox-on-apple-silicon-m1-m2/)

- Your lab needs to be working by Week 5 so that you can complete the assignments due at the end of Week 6 and Week 8. 
 
## What to turn In

- In your repo, submit a pull request to merge your assignment into the main branch. 
- Submit your assignment in Markdown in assignments > weekly-projects > project-1-labsetup.md (see "sample-github-structure" folder)
- Put your images in a separate "images" directory (see "sample-github-structure" folder)
- Write up the final state of your lab using screen shots to illustrate what you did. 
- In the week, when the assignment it due, post in Slack a link to your assignment (main branch only). 


### Key Deliverabiles in Write up
- Your write up should include the following (at a minimum) 	

1. An overview discussing the technology stack:  Mac/PC, Type of Hypervisor (Virtual Box, VM Ware, etc.).
2. An archtectural diagram of your lab that includes at least two components (two VMs).  One component needs to be your Kali Linux box and the other needs to be the box you attack, e.b. Metasploitable (see page 41). 
    -Use a diagraming tool and post an image in your document. 
3. Screen shot of the running Virtualization Environment (e.g. Virtual Box.)
4. Screen shot of the running Kali Linux Box (show that you are logged in).
5. Screen shot showing you installed Nessus.
6. Screen shot showing running Metasploitable 2. 
7. Screen shot showing that you can ping the Metasploitable 2 box (the box to be attacked) from the Kali Linux box. 
8. Description of any problems you ran into and how you solved them. 
