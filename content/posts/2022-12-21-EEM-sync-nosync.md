---
title: "EEM: To sync or not to sync"
date: 2022-12-21T07:01:24-06:00
draft: false
---

For those of you familiar with Cisco’s EEM (Embedded Event Manager), this post will try to explain how to use the “sync” parameter in an EEM applet. It’s usage seems to confuse users, including myself sometimes. So I did some reading and testing, and here’s what I found.

Note that EEM usually requires an event detector to be specified, i.e. CLI events, syslog events, routing events, etc. The sync argument is only used with the CLI event detector.

As far as I can tell, synchronous mode (“sync yes”) is required ***only*** if you want to run both the EEM applet ***and*** the original CLI command, ***in that order***. If you don’t care about the order, or don’t want to run the original CLI command, then it doesn’t matter whether you use sync mode or not.

Let’s show some examples. (These were run on a Cisco ISR 4331 running 16.9.6 code.)

### Synchronous mode (“sync yes”)

So you want run the applet before the matched CLI command. In this example, whenever the user types the `show alias` command, you want a `show clock` to be executed first:

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

Note the “sync yes” arguments added to the end of the “event cli...” command, which sets synchronous mode. (I also used a caret in front of the word “show” to only match lines beginning with “show alias”). To execute the original show alias command afterwards, you need the `set _exit_status "1"`  command. Setting it to 1 executes the command. If you don’t set this variable, or set it to 0, then the command is skipped.

### Asynchronous mode (“sync no”)

This will run the applet and the matched CLI command independently (asynchronously) of each other. In async mode, you are required to specify the skip parameter to indicate whether to execute the original CLI command. Assuming “skip no” is used, in my testing the CLI command was always executed first, then the EEM applet:

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
*Dec 28 04:14:15.609: %SYS-5-CONFIG_I: Configured from console by console
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
Dec 28 04:14:26.287: %HA_EM-6-LOG: SyncNo:    <== NOW THE APPLET EXECUTES AFTER THE CLI COMMAND
04:14:26.204 UTC Wed Dec 28 2022
Router#
```

Revised 2022-12-28


