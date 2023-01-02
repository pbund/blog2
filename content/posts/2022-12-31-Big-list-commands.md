---
title: "Big List of Cisco/misc commands"
date: 2022-12-31T10:46:43-06:00
draft: false
---

This is a compendium of useful or interesting commands I’ve gathered over the years. Mainly these apply to Cisco equipment, but there are a couple of Juniper and Linux commands.

Warning: Some of these command are dangerous, where they might brick your system if used. Don’t come crying to me if this happens. I would google their usage before any relying on them for production systems. But largely they are more useful than dangerous.


# H1

not text 2

## H2

- list
    - list
- list

# Useful commands
- to activate serial controllers and interfaces, when they are first installed, especially with a multiflex t1 card, not a wic-1dsu-t1-v2 type card
    - `card type t1 0 1` (example) will cause controllers to appear in config
    - `controller t1 0/1` > `channel-group 0 timeslots 1-24` will cause s0/1 interface to appear in config
-	use ctrl-v or esc+q, then a ?,  to enter a ? in a CLI password
-	term mon   =>  enables you see logging/debug output in a telnet/ssh session
-	term no mon   =>  turns it off, should also use no logging console at the same time, because it causes cpu interrupts which are cpu intensive
-	sh context, env, inventory, diag are good cmds
-	send log paul is great   =>  puts “%sys-2-logmsg: Message from 1(oldpaul): paul is great” in the logs, good way to annotate them
-	send *                           =>  send other logged in users a message
-	send *                           =>  from a terminal access server, send following cmds to all connected hosts, whether active or not (i.e. end, config t, <cmd>)
-	logging discriminator Foo msg-body includes starttext.*endtext   => define discriminator
-	logging buffered discriminator Foo 32768     => only log messages to buffer that match the discriminator
-	logging rate-limit console 1 except critical    => rate limit console messages to 1 per sec, except for critical and more important (lower) level msgs
-	ip access-list log-update threshold 1   => display log messages for each ACE (in an ACL) hit, as opposed to waiting for default 5 minutes, log acl
-	ip access-list logging interval 100        => one log-enabled ACE packet per 100 ms will be process-switched; limits the effects of ACL logging-induced process switching
-	term exec prompt timestamp   =>  display date/time/cpu on show cmds, in your current session
-	line vty 0 15> exec prompt timestamp   =>  same thing, but permanently for all users
-	term escape-character 3          => enable ctrl-c instead of ctrl-shift-6, just for this login session
-	escape-character 3                  => on console/vty lines, enable ctrl-c instead of ctrl-shift-6 everytime you login
-	escape-character x                  => same thing, but use the letter x
-	wr, reload in 010, do changes w/o saving, if you are cutoff then let router reload, but if changes work then “reload cancel”
-	logging synchronous, on console and vty lines  => helps with screen clutter
-	delete /for /rec <dir>, delete directory and all its contents
-	verify /md5 foo.bin        => show md5 for file foo.bin
-	crypto key gen rsa <#> -- transport input ssh  => enables ssh, also requires either ip domain name to be set or hostname, or both
-	no ip domain-lookup  => do not do a dns lookup when you mistype a cmd
-	exception dump 1.1.1.1  => look this up, think it sends crash dump to 1.1.1.1
-	tftp-server flash foo.bin 99  => make your router a tftp server of file foo.bin, to hosts allowed by acl 99, can use this multiple times 
-	boot system foo.bin 1.1.1.1  => tftp boot from image foo.bin on host 1.1.1.1
-	copy http://user1:pword@1.1.1.1/dir/foo.bin flash:   => copy file from web server
-	sh run | i p.*sum   => show lines with an “p” in it, then arbitrary text, then “sum”
-	on long output cmds, when “--More—“ appears at bottom, can use /, followed by text, to jump to that text
-	or use + text, which will grep the rest of output for that text
-	or use – text, which will grep –v the rest of the output for that text
-	debug ip tcp transactions   => debug tcp session
-	example TCL script, which pings multiple IPs:
tclsh
foreach i {
150.1.1.1
204.12.1.254
} {ping $i repeat 2}
-	example switch Macro script, which pings multiple IPs:
conf t
macro name Ping
Enter macro commands one per line. End with the character '@'.
do ping 150.1.1.1 repeat 3 source Lo0
do ping 204.12.1.254 repeat 3 source Lo0
@
switch(config)# macro global apply Ping 
-	another way to TCL ping IPs, but create executable file:
tclsh
  puts [open "flash:foo.tcl" w+] {
    foreach i {
      150.1.1.1
      204.12.1.254
      } { puts [ exec "ping $i repeat 2" ] }
  }
tclquit
tclsh foo.tcl
-	example switch Macro script, which pings multiple IPs:
conf t
macro name Ping
Enter macro commands one per line. End with the character '@'.
do ping 150.1.1.1 repeat 3 source Lo0
do ping 204.12.1.254 repeat 3 source Lo0
@
switch(config)# macro global apply Ping 
-	Use TCL to batch IOS commands:
tclsh
  ios_config "int g0/0” “speed auto” “duplex auto”
  tclquit
-	Use TCL to create blank config file in flash:
tclsh
  puts [ open "flash:blank.cfg" w+] {
  version 15.4
  !
  end
}
tclquit
-	Change to default (blank) config without rebooting, using local blank config file created in previous step, takes 10-15 sec:
router# config replace flash:blank.cfg
-	Use EEM to wait for two events to occur, in any order, then take some action:
event manager applet Foo
event tag e1 syslog pattern paul2
event tag e2 syslog pattern jill2
trigger
  correlate event e1 andM event e2
action 1.0 syslog msg “write some msg to syslog”
action 2.0 cli command “show clock | append foo.txt”
-	Enable shell cmds in this session or permanently, good on ios 15.1+. Enables multi-pipes, grep, man, cat, cut, echo, head, tail, sort, nl, redirect to file, unix-script for cmds and functions (see D Bombal youtube videos)
term shell           => for this session only
conf>shell processing full    => for everyone
-	example shell cmds on router
sh ip route | grep –i loopback | grep –v 123
sh run | grep –b banner | grep –u alias => start with first line containing banner, end before line containing alias 
-	example shell script to ping 3 consecutive IPs, from exec command line
for x in 1 2 3
   do
   ping 10.10.10.$x
   done              => starts the three pings
-	example Linux function, stays around for session
function testping (){
   ping 10.10.1.1
   ping 10.10.1.2
}
testping             => executes function testing
sh shell function    => display above function and others 

-	sh file <file> md5sum     => Nexus md5 verification
-	hardware access-list allow deny ace => Nexus 6.1(3) and up, allow ip tcp synwait-time 10   => lowers tcp timeout from default 30s to 10s
-	sh power inline  => show which ports are powering POE devices
-	mac address-table notification mac-move   => shows port flapping on switch
-	int> mac-address-table 0000.0000.0001   => change my mac address
-	denies in PBR acls
-	clear spanning-tree counters
-	sh spanning-tree interface f0/0 detail => show # of bpdus sent/received
-	sh spanning-tree detail | I ieee|from|is exec     => shows STP TCN events on a switch 
-	sh spanning-tree detail | incl ieee|from|occur     => shows STP TCN events on a switch 
-	control-plane host > management-interface f2/0 allow ssh
-	no mgmt. protocols except ssh, and only on f2/0 (deprecated now?)
-	test aaa group tacacs+ fc8466 <pw> legacy=> test aaa 
-	sh ip alias	=> shows all IPs associated with this device
-	mac address-table static <mac> vlan 1 int f0/1  => statically set mac
-	mac address-table static <mac> vlan 1 drop      => drop traffic to  this mac
-	no mac address-table learning vlan 1 => turn off all learning on vlan 1, would require static mac entries to work
-	show memory failures alloc	=> show memory failures
-	sh snmp mib		=> show all mib values
-	sh snmp mib ifmib ifindex details	=> shows interface ifindex values
-	snmp-server ifindex persist	=> maintain same ifindex values
-	3650 packet capture commands
-	no monitor capture 3
-	monitor cap 3 match any buffer size 2 interface g1/0/3 both (no filter)
-	monitor cap 3 match ipv4 any host 192.168.1.1 buffer size 2 int g1/0/3 both (simple filter)
-	monitor cap 3 start|stop
-	show monitor cap 3 buff	=> see packet dump on screen
-	monitor cap 3 export flash:foo.pcap
-	copy foo.pcap scp://172.16.100.100
-	43xx/44xx packet capture commands (also probably other ios-xe switches)
-	no monitor capture 1
-	monitor capture 1 match any OR
-	monitor capture 1 access-list Not_eigrp
-	monitor capture 1 interface f0/0 both
-	monitor capture 1 interface f0/1 both
-	monitor capture 1 start|stop
-	sh monitor cap 1 buffer		=> show number of packets captured
-	sh monitor cap 1 buffer	brief	=> show packets (one-liners)
-	monitor capture 1 export tftp://10.1.1.1/foo.pcap
-	3900 packet capture (this version works on 3900 routers)
-	monitor capture buffer Paulbuffer size 2048 max-size 100 linear
-	ip access-list ext Foo
-	permit ip host 1.1.1.1 any
-	monitor capture buffer Paulbuffer filter access-list Foo
-	monitor capture point ip cef Paulpoint g0/1 both
-	monitor capture point associate Paulpoint Paulbuffer
-	monitor capture point start Paulpoint  (can use “stop” too)
-	sh monitor capture point Paulpoint
-	sh monitor capture buffer Paulbuffer parameters (show # of packets captured)
-	sh monitor capture buffer Paulbuffer  (shows captured packets)
-	sh monitor capture buffer Paulbuffer dump  (show contents of packets)
-	monitor capture buffer Paulbuffer export tftp://1.1.1.1/capture.pcap
-	to decrypt a layer 7 encypted password directly on a router, paste the encrypted pw in a key chain, then do a “show key chain” to show the plaintext version
-	key chain Foo > key 1 > key-string 7 7 0522282A
-	show key chain
-	“Key-chain Foo: key 1 == text “INE” …”
-	sh ip bgp version recent 10	=> ten most recent bgp best paths
-	sh ip route | include 00:00		=> show most recent RIB updates
-	show facility-alarm status		=> on asr1001x, show alarms that activate idiot/critical lights (when port is configged up but status down)
-	test cable-diagnostics tdr interface g0/24, then sh cable-diag tdr int g0/24
-	-------------------
-	ACI cli cmds
-	-------------------
-	
-	APIC CLI cmds
-	---------------------
-	show endpoint ip <ip>    => show where ip is
-	show vpc map          => maps vpcs to interface policy name
-	show port-channel map   => similar to sh vpc map but shows less info
-	show stats granularity 15min leaf 203 interface ethernet 1/5
-	sh run
-	sh version
-	sh controller
-	sh switch                                   => shows all spines/leaves, with IPx3/ser#/ver
-	acidiag fnvread                          => subset of show switch
-	df -h
-	sh tenant
-	sh bridge-domain
-	sh vrf
-	sh vlan
-	sh vlan-domain
-	show endpoint                          => aka 's end', big list of macs/IPs, ctrl-c works
-	show endpoint mac 2880.23a4.2260     => shows the leaf and port where it sees this mac
-	sh endpoints leaf 201 interface ethernet 1/1-128
-	fabric 101-102 show version | grep supervisor   => show model numbers of spines
-	fabric 201-202 show ip int brief vrf all
-	 
-	Spine/Leaf CLI cmds
-	--------------------------
-	(must type out whole word 'show')
-	show int brief                             => shows which ints are up, also speed/trunk status
-	show int description                   => show port description, aka 'show int des'q
-	show ip int brief                         => similar to IOS, also shows VRFs
-	show cdp neighbor (detail)        => aka 'show cdp n'
-	show lldp neighbor (detail)        => aka 'show ll n d'
-	show version
-	show hardware                           => show version + show inventory
-	show vrf
-	show vlan
-	show interface                            => big list, ctrl-c works, so does piping to 'more'
-	show interface e1/1
-	show interface | grep ^[A-Z] | grep 'is up'         => show all up interfaces
-	show interface | egrep '^[A-Z].*is up|30 sec' | grep -v ' 0 packets'   => show rates on up interfaces
-	show interface e 1/1-3 switchport|status|transceiver      => show various info for these three interfaces
-	acidiag fnvread                          => subset of show switch
-	show endpoint mac 0050.56aa.1081    => not as much info as when entered on APIC
-	show mac address-table | grep <mac> =>
-	show vpc
-	iping <ip>, should specify tenant:vrf with '-V tenant:vrf' or it will use mgmt vrf
-	iping -V anc_prod:anc_prod_main_vrf -S 151.169.2.1 151.169.108.55
-	
-	Juniper firewall CLI cmds
-	----------------------------------
-	show interface terse
-	show route 1.1.1.1
-	show chassis alarm
-	show chassis cluster status
-	show chassis fpc details
-	show log mess | last 50


