---
title: WhatsApp two-step verification reminder bypass.
classes: wide
---

In February 2019 I contacted Facebook team regarding a bug I found in WhatsApp two-step verification reminder feature, a feature intended to help users remember its PIN. 

{% include video id="KnD_BiSrTcU" provider="youtube" %}

# Whatsapp Two-Step verification?

Two-step verification is an optional feature that adds more security to your account. When you have two-step verification enabled, any attempt to verify your phone number on WhatsApp must be accompanied by the six-digit PIN that you created using this feature.

For more information check Whatsapp FAQ. [https://faq.whatsapp.com/en/android/26000021](https://faq.whatsapp.com/en/android/26000021)

![ws-two-step](/assets/images/ws-two-step2.jpg)

# How it works?
 
To help you remember your PIN, WhatsApp will periodically ask you to enter your PIN. There is no option to disable this without disabling the two-step verification feature. 
 
Did you notice the "There is no option to disable this without disabling the two-step verification feature"?

![shaq](/assets/images/shaq.gif)

# How I found the bug?
 
While using WhatsApp I get the prompt for my PIN and I start doing weird things in my iPhone to see how the app reacts until I found that if I move the iPhone right or left and open WhatsApp while in that position, it will allow me to open the app and access without disabling the two-step verification feature.
 
I won't take credit for this discovery, I have to confess that my little girl teach me how to do weird things in my iPhone, she it's always doing weird things in all peoples smartphone's so, this bug have your name my princess :) 
 
The bug affects iOS and the latest WhatsApp, I tested it on iPhone 8 plus & iPhone 6 (even though this this doesn't rotate the home screen), it may affect Android or other iPhone's, that's up to you to test and report back. 
 
# Bug reported to Facebook
 
If you find a bug in one of the Facebook app you can report it using this link: [https://www.facebook.com/whitehat/](https://www.facebook.com/whitehat/)
 
Since this bug is in the reminder function and it does not affect the registration process or in any way your privacy. It was not considered as a Security. Here their response regarding my report: 

![fb-response](/assets/images/fb-response.png)

I agreed with the Facebook team, since this two-steps verification it's not a two-factor authentication but a reminder to help the user remember it's PIN. 
 
While I agree with Facebook team, I would like to ask you, what are your thoughts about their answer? Do you think it's a security vulnerability? or do you also agree with them? 
 
# Conclusion
 
As Security Specialist sometimes we overcomplicate things when we are looking for security vulnerabilities or applications bugs. Numerous time I found myself trying to do ninja stuffs on a Pentesting, Web Application or CTF game and the solution was way simpler than I thought in the first place. 
 
I just want to recommend all security professionals to start fixing the low hanging fruits security issues in their networks, applications, systems, etc., that may cause big impact in your operations, and then move to build defenses against advance threats or any 0 days that gets on twitter. 
 
Thanks for reading! :)

**God bless you!**

**Serving Christ is not a task, but a relationship. Friends of God Jn 15:15**
