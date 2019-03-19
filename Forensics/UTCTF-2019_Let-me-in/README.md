UTCTF 2019: RIP
==================================

Challenge description:

*My Friend John sent me this password protected zip, but forgot the password. I'm sure you can still find a way in,though.*

The first step qhen we download the zip i justo to crack the zip file with fcrackzip using rockyou list file, but we failed. The next step was to use John the ripper to due to the challengue description. So we need try to crack zip file by using john and modified the config of john for adding variations of the rockyou list file. To make this , we need to change the configuration , adding a specific rule for this challenge , so we firstly try to add 3 numbers at the end of every word in the list:

![](pics/pic1.png?raw=true)

But we failed!
So we did the same for only 2 numbers , and again... we failed, so one more and last time we do the same but only adding 1 number and BINGO!!, after a few seconds we got the password.

![](pics/pic2.png?raw=true)

When we got the password when can unzip de zip file and get a png with the flag.

![](pics/flag.png?raw=true)

Flag : *utflag{m1n1_C00p3r_f4n}*
