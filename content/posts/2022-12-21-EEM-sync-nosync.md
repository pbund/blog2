---
title: "EEM: To sync or not to sync"
date: 2022-12-21T07:01:24-06:00
draft: false
---

For those of you familiar with Cisco’s EEM (Embedded Event Manager), this post will try to explain how to use the “sync” parameter in an EEM applet. Its usage seems to confuse users, including myself sometimes. So I did some reading and testing, and here’s what I found.

Note that EEM usually requires an event detector to be specified, i.e. CLI events, syslog events, routing events, etc. The sync argument is only used with the CLI event detector.

As far as I can tell, synchronous mode (“sync yes”) is required ***only*** if you want to run both the EEM applet ***and*** the original CLI command, ***in that order***. If you don’t care about the order, or don’t want to run the original CLI command, then it doesn’t matter whether you use sync mode or not.

Let’s show some examples. (These were run on a Cisco ISR 4331 running 16.9.6 code.)

### Synchronous mode (“sync yes”)

So you've decided to run the applet before the matched CLI command. In this example, whenever the user types the `show alias` command, you want a `show clock` to be executed first:

```
Router# conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)# event manager applet SyncYes   
Router(config-applet)# event cli pattern "^show alias" sync yes
Router(config-applet)# action 2.1 cli command "enable"
Router(config-applet)# action 2.3 cli command "show clock"
Router(config-applet)# action 2.5 puts "$_cli_result"
Router(config-applet)# action 9.9 set _exit_status "1" 
Router(config-applet)# end
Router#
Router#show alias

02:30:24.551 UTC Wed Dec 28 2022   <== SHOW CLOCK EXECUTED FIRST
Router#

Exec mode aliases:                 <== THEN SHOW ALIAS
  h                     help
  lo                    logout
  p                     ping
  r                     resume
  s                     show
  u                     undebug
  un                    undebug
  w                     where

Router#
```

Note the “sync yes” arguments added to the end of the “event cli...” command, which sets synchronous mode. (I also used a caret in front of the word “show” to only match lines beginning with “show alias”). To execute the original show alias command afterwards, you need the `set _exit_status “1”` command. Setting it to 1 executes the command. If you don’t set this variable, or set it to 0, then the command is skipped.

### Asynchronous mode (“sync no”)

In async mode, you are required to add the "skip" parameter to indicate whether to run the original CLI command in addition to your applet. If “skip no” is used, then both the applet and the matched CLI command will run independently (asynchronously) of each other. However, in my testing the CLI command was always executed first, then the EEM applet:

```
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#no event manager applet SyncYes
Router(config)#event manager applet SyncNo    
Router(config-applet)#event cli pattern "^show alias" sync no skip no
Router(config-applet)# action 2.1 cli command "enable"
Router(config-applet)# action 2.3 cli command "show clock"
Router(config-applet)# action 2.5 puts "$_cli_result"
Router(config-applet)#end        
Router#
Router#sh alias
Exec mode aliases:
  h                     help
  lo                    logout
  p                     ping
  r                     resume
  s                     show
  u                     undebug
  un                    undebug
  w                     where

Router#
Dec 28 04:14:26.287: %HA_EM-6-LOG: SyncNo:    <== SYSLOG MESSAGE
04:14:26.204 UTC Wed Dec 28 2022              <== NOW THE CLI COMMAND OCCURS AFTERWARDS
Router#
```

You can see where I added “skip no” arguments at the end of the “event cli" line, which runs the CLI command in addition to the EEM applet. I also don’t need the `set _exit_status “1”` command anymore, since that only applies to sync mode. 

If you don’t want to run the CLI command, either sync or async mode could be used. In this case I would suggest using async mode, since its commands are clearer to the user.

### Summary

-	If you want to run an EEM applet ***then*** the matched CLI command, use sync mode (`event cli pattern “show alias” sync yes` and `set _exit_status “1”` lines, for example)
-	If you want to run an EEM applet and the matched CLI command, in no particular order, use async mode (`event cli pattern “show alias” sync no skip no` for example). Remember this will probably run the CLI command first.
-	If you don’t want to run the matched CLI command at all, use either mode (although async mode is clearer, using `event cli pattern “show alias” sync no skip yes`, for example)

Googling “eem sync skip” will show many good links on this subject. I recommend any article under ipspace.net – author Ivan Pepelnjak is a network stud.

Revised 12/29/22 - email oldpaul99@gmail.com if you have comments/corrections.
