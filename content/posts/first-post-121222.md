---
title: "Fun with TTCP"
date: 2022-12-12T18:30:12-06:00
draft: false
---


I’ve always been interested in lesser-known or unusual Cisco IOS commands. TTCP is one of these.

TTCP (“Test-TCP”) is a hidden command available on most Cisco routers and switches. It can be used to test TCP connection speeds between Cisco devices, if both devices support it. It tries to send data as fast as possible (normal TCP behavior), but my tests show the speed is nowhere near the maximum link speed (it generated 22 Mbps on a 100 Mbps connection between two Catalyst 3560 switches). But I use it to generate traffic to see if line errors are increasing, or maybe test QoS or ACL policies.

Since this is an undocumented command, no command line help or tab completion will show its existence. Also note that the CPU usage jumps to near 100% on both sides when data is being sent -- this can negatively affect your boss’s youtubing activities.

## How to do it

You first need to start ttcp on the receiving device, where you can select all the defaults. Note you don’t need to specify any IP information on the listening device (basically it says “Let me open a session listening on TCP port 5001 on all interfaces”):


C3560-7.4# **ttcp**
transmit or receive [receive]: 
receive packets asynchronously [n]: 
perform tcp half close [n]: 
receive buflen [32768]: 
bufalign [16384]: 
bufoffset [0]: 
port [5001]: 
sinkmode [y]: 
rcvwndsize [32768]: 
ack frequency [0]: 
delayed ACK [y]: 
show tcp information at end [n]: 

ttcp-r: buflen=32768, align=16384/0, port=5001
rcvwndsize=32768, delayedack=yes  tcp


At this point the CLI will just hang, waiting for a ttcp connection from another device. 

On the other (transmitting) device, you similarly use ttcp to establish a transmitting session to the listening ttcp device:

C3560-7.5# **ttcp**
transmit or receive [receive]: **transmit**
Target IP address: **192.168.7.4**     (this is the IP of the listening box)
calculate checksum during buffer write [y]: 
perform tcp half close [n]: 
send buflen [32768]: 
send nbuf [2048]: 
bufalign [16384]: 
bufoffset [0]: 
port [5001]: 
sinkmode [y]: 
buffering on writes [y]: 
show tcp information at end [n]: **y**

ttcp-t: buflen=32768, nbuf=2048, align=16384/0, port=5001  tcp  -> 192.168.7.4
ttcp-t: connect
ttcp-t: 67108864 bytes in 23790 ms (23.790 real seconds) (~2753 kB/s) +++
ttcp-t: 2048 I/O calls
ttcp-t: 0 sleeps (0 ms total) (0 ms average)
Connection state is ESTAB, I/O status: 1, unread input bytes: 0
Mininum incoming TTL 0, Outgoing TTL 255
Local host: 192.168.7.5, Local port: 47996
Foreign host: 192.168.7.4, Foreign port: 5001

Enqueued packets for retransmit: 17, input: 0  mis-ordered: 0 (0 bytes)

Event Timers (current time is 0x9D1F9F2B):
Timer          Starts    Wakeups            Next
Retrans          5761          0      0x9D1FA052
TimeWait            0          0             0x0
AckHold             0          0             0x0
SendWnd             0          0             0x0
KeepAlive           0          0             0x0
GiveUp              0          0             0x0
PmtuAger            0          0             0x0
DeadWait            0          0             0x0

iss: 1036349108  snduna: 1103433965  sndnxt: 1103457973     sndwnd:  32768
irs: 4060423733  rcvnxt: 4060423734  rcvwnd:       4128  delrcvwnd:      0

SRTT: 300 ms, RTTO: 303 ms, RTV: 3 ms, KRTT: 0 ms
minRTT: 0 ms, maxRTT: 300 ms, ACK hold: 200 ms
Flags: higher precedence, retransmission timeout

Datagrams (max data segment is 1460 bytes):
Rcvd: 45060 (out of order: 0), with data: 0, total data bytes: 0
Sent: 47106 (retransmit: 0), with data: 47104, total data bytes: 67108864

You can see above that 67 Mbytes were sent in 24 seconds, for a rate of 2.7 MB/s, or 22 Mbps.

Note the high CPU usage during the data transfer (both switches show similar levels):

C3560-7.5#sh proc cpu h
                                                              
    66669999999999999999999911111                             
    7777666666666633333333332222244444555554444444444555555555
100     **********                                            
 90     ********************                                  
 80     ********************                                  
 70 ************************                                  
 60 ************************                                  
 50 ************************                                  
 40 ************************                                  
 30 ************************                                  
 20 ************************                                  
 10 *****************************     *****          *********
   0....5....1....1....2....2....3....3....4....4....5....5....
             0    5    0    5    0    5    0    5    0    5    
               CPU% per second (last 60 seconds)
<snipped>

## Links to other TTCP info

Some other sites with info on ttcp
http://technologyordie.com/testing-throughput-with-ttcp-and-cisco-devices

-	No VRF support, available on ISR 4321

If you really want to test maximum speed, I would use alternative speed tests between PCs/Macs attached directly to the network devices in question.

