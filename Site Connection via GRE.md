
<!-- #$34T# = Your Monitor Number -->

### REMOVE L3 OPERATIONS
~~~
!@EDGE
conf t
 no ip routing
 end
~~~

## Site to Site connection via VTI (GRE)

### SET GATEWAY
~~~
!@EDGE
conf t
 int g0/0/1
  ip add 200.0.0.#$34T# 255.255.255.0
  no shut
  exit
 ip route 0.0.0.0 0.0.0.0 200.0.0.1
 end
~~~

&nbsp;
---
&nbsp;

### CONFIGURE NAT
1. Define INSIDE and OUTSIDE
~~~
!@EDGE
conf t
 int g0/0/1
  ip nat outside
 int g0/0/0
  ip nat inside
~~~

<br>

2. Create an ACL to match traffic to be translated.
~~~
!@EDGE-~~       MARK AN EXCLAMATION !
conf t
 int g0/0/0
  ip nat inside
 int g0/0/1
  ip nat outside
 ip access-list NAT-TRAFFIC
  deny ip 10.~~.0.0 0.0.255.255 10.11.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.12.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.21.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.22.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.31.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.32.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.41.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.42.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.51.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.52.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.61.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.62.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.71.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.72.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.81.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.82.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.91.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.92.0.0 0.0.255.255
  permit ip any any
  end
~~~

<br>

3. Define NAT
~~~
!@EDGE
conf t
 ip nat inside source list NAT-TRAFFIC int g0/0/1 overload
 end
~~~

<br>
<br>

Verify: 
~~~
!@TAAS, BABA, CUCM, EDGE
ping 8.8.8.8
ping 8.8.4.4
~~~

~~~
!@EDGE
show ip nat translations
~~~

&nbsp;
---
&nbsp;

### MODIFY ROUTING FOR INSIDE
~~~
!@EDGE
conf t
 no router ospf 1
router ospf 1
 router-id #$3AT#.0.0.1
 network 10.#$3AT#.#$3AT#.0 0.0.0.255 area 0
 default-information originate always
 end
~~~

&nbsp;
---
&nbsp;

### CONFIGURE GRE TUNNELS

~~~
ip nhrp auth cisco
ip nhrp map multi dynamic
ip nhrp net ip 1
tunnel source g0/0/1
tunnel mode gre multi


**********

!@I1
conf t
 hostname I1
 ip domain lookup
 no logging cons
 int vlan 1
  ip add 200.0.0.1 255.255.255.0
  no shut
  end


!@HQ
conf t
 hostname HQ
 ip domain lookup
 no logging cons
 int lo 0
  ip add 10.10.10.10 255.255.255.255
  exit
 int e0/0
  ip add 200.0.0.10 255.255.255.0
  no shut
  end

!@E1
conf t
 hostname E1
 ip domain lookup
 no logging cons
 int lo 0
  ip add 1.1.1.1 255.255.255.255
  exit
 int e0/0
  ip add 200.0.0.11 255.255.255.0
  no shut
  end

!@E2
conf t
 hostname E2
 ip domain lookup
 no logging cons
 int lo 0
  ip add 2.2.2.2 255.255.255.255
  exit
 int e0/0
  ip add 200.0.0.12 255.255.255.0
  no shut
  end

!@E3
conf t
 hostname E3
 ip domain lookup
 no logging cons
 int lo 0
  ip add 3.3.3.3 255.255.255.255
  exit
 int e0/0
  ip add 200.0.0.13 255.255.255.0
  no shut
  end



!@I1
conf t
 int lo8
  ip add 8.8.8.8 255.255.255.255
  exit
 router bgp 1
  bgp log-neighbor-changes
  neighbor 200.0.0.10 remote-as 10
  neighbor 200.0.0.11 remote-as 11
  neighbor 200.0.0.12 remote-as 12
  neighbor 200.0.0.13 remote-as 13
  address-family ipv4
   neighbor 200.0.0.10 activate
   neighbor 200.0.0.11 activate
   neighbor 200.0.0.12 activate
   neighbor 200.0.0.13 activate
   network 200.0.0.0 mask 255.255.255.0
   network 8.8.8.8 mask 255.255.255.255
   end

!@HQ
conf t
 router bgp 10
  bgp log-neighbor-changes
  neighbor 200.0.0.1 remote-as 1
  address-family ipv4
   neighbor 200.0.0.1 activate
   network 200.0.0.0 mask 255.255.255.0
   end

!@E1
conf t
 router bgp 11
  bgp log-neighbor-changes
  neighbor 200.0.0.1 remote-as 1
  address-family ipv4
   neighbor 200.0.0.1 activate
   network 200.0.0.0 mask 255.255.255.0
   end

!@E2
conf t
 router bgp 12
  bgp log-neighbor-changes
  neighbor 200.0.0.1 remote-as 1
  address-family ipv4
   neighbor 200.0.0.1 activate
   network 200.0.0.0 mask 255.255.255.0
   end

!@E3
conf t
 router bgp 13
  bgp log-neighbor-changes
  neighbor 200.0.0.1 remote-as 1
  address-family ipv4
   neighbor 200.0.0.1 activate
   network 200.0.0.0 mask 255.255.255.0
   end






!@Device
conf t
 int e0/0
  ip nat outside
 int e0/1
  ip nat inside
 access-list 1 permit 172.16.1.0 0.0.0.255
 ip nat inside source list 2 int e0/0 overload
 ip route 0.0.0.0 0.0.0.0 200.0.0.1
 !
 access-list 2 permit 10.0.0.0 0.255.255.255
 ip nat inside source list 1 int tun1 overload
 end
 



---


!@HQ
conf t
 int tun1
  ip add 172.16.1.10 255.255.255.0
  tunnel mode gre multipoint
  ip nhrp map multicast dynamic
  ip nhrp network-id 1010
  ip nhrp map 172.16.1.11 200.0.0.11
  ip nhrp map 172.16.1.12 200.0.0.12
  ip nhrp map 172.16.1.13 200.0.0.13
  no ip nhrp nhs 172.16.1.10
  exit
 router eigrp 1
  no auto-summary
  network 172.16.1.0 0.0.0.255
  network 200.0.0.0 0.0.0.255
  end

!@E1
conf t
 int tun1
  ip add 172.16.1.11 255.255.255.0
  tunnel source e0/0
  tunnel mode gre multipoint
  ip nhrp map multicast dynamic
  ip nhrp network-id 1010
  ip nhrp map 172.16.1.10 200.0.0.10
  ip nhrp map 172.16.1.12 200.0.0.12
  ip nhrp map 172.16.1.13 200.0.0.13
  no ip nhrp nhs 172.16.1.10
  end

!@E2
conf t
 int tun1
  ip add 172.16.1.12 255.255.255.0
  tunnel source e0/0
  tunnel mode gre multipoint
  ip nhrp map multicast dynamic
  ip nhrp network-id 1010
  ip nhrp map 172.16.1.10 200.0.0.10
  ip nhrp map 172.16.1.11 200.0.0.11
  ip nhrp map 172.16.1.13 200.0.0.13
  no ip nhrp nhs 172.16.1.10
  end

!@E3
conf t
 int tun1
  ip add 172.16.1.13 255.255.255.0
  tunnel source e0/0
  tunnel mode gre multipoint
  ip nhrp map multicast dynamic
  ip nhrp network-id 1010
  ip nhrp map 172.16.1.10 200.0.0.10
  ip nhrp map 172.16.1.11 200.0.0.11
  ip nhrp map 172.16.1.12 200.0.0.12
  no ip nhrp nhs 172.16.1.10
  end




Routing (Static)
!@E1
conf t
 ip route 10.0.2.0 255.255.255.0 172.16.1.12
 ip route 10.0.3.0 255.255.255.0 172.16.1.13
 end

!@E2
conf t
 ip route 10.0.1.0 255.255.255.0 172.16.1.11
 ip route 10.0.3.0 255.255.255.0 172.16.1.13
 end

!@E3
conf t
 ip route 10.0.2.0 255.255.255.0 172.16.1.12
 ip route 10.0.1.0 255.255.255.0 172.16.1.11
 end


Dynamic
NAT EXCEMPT -1
!@E1
conf t
 access-list 11 permit 10.0.1.0 0.0.0.255 10.0.2.0 0.0.0.255
 access-list 11 permit 10.0.1.0 0.0.0.255 10.0.3.0 0.0.0.255
 ip nat inside source
!@E1
conf t
 router ospf 1
  network 172.16.1.0 0.0.0.255 area 0
  end




-2
!@E1
conf t
ip access-list ex EX 
 deny ip 10.0.1.0 0.0.0.255 10.0.2.0 0.0.0.255
 deny ip 10.0.1.0 0.0.0.255 10.0.3.0 0.0.0.255
 permit ip any any             
ip nat inside source list EX int e0/0 overload

!@E2
conf t
ip access-list ex EX 
 deny ip 10.0.2.0 0.0.0.255 10.0.1.0 0.0.0.255
 deny ip 10.0.2.0 0.0.0.255 10.0.3.0 0.0.0.255
 permit ip any any             
ip nat inside source list EX int e0/0 overload

!@E3
conf t
ip access-list ex EX 
 deny ip 10.0.3.0 0.0.0.255 10.0.2.0 0.0.0.255
 deny ip 10.0.3.0 0.0.0.255 10.0.1.0 0.0.0.255
 permit ip any any             
ip nat inside source list EX int e0/0 overload
~~~


&nbsp;
---
&nbsp;


### SET STATIC ROUTING THROUGH GRE TUNNELS

<br>
<br>

Verify:

~~~
!@TAAS, BABA, CUCM, EDGE
traceroute 10.11.1.10
traceroute 10.12.1.10
~~~

&nbsp;
---
&nbsp;

### SET NAMESERVER 
> [!IMPORTANT] 
> MUST be WINSERVER.  
> NOT a public DNS

<br>

~~~
!@TAAS, BABA, CUCM, EDGE
conf t
 ip domain lookup
 ip name-server 10.#$34T.1.10
 end
~~~

<br>

Configure WINSERVER with a DNS Forwarder for 8.8.8.8 and 1.1.1.1
