
<!-- Your Monitor Number == #$34T# -->

## Configure Multisite Connectivity
~~~
!@EDGE
conf t
 hostname EDGE-#$34T#
 enable secret pass
 service password-encryption
 no logging console
 no ip domain-lookup
 line cons 0
  password pass
  login
  exec-timeout 0 0
 line vty 0 14
  password pass
  login
  exec-timeout 0 0
 int gi 0/0/0
  no shut
  ip add 10.#$34T#.#$34T#.1 255.255.255.0
  desc INSIDE
 int gi 0/0/1
  no shut
  ip add 200.0.0.#$34T# 255.255.255.0
  desc OUTSIDE
 int loopback 0
  ip add #$34T#.0.0.1 255.255.255.255
  desc VIRTUALIP
 end
~~~

<br>

~~~
!@EDGE
conf t
 no router ospf 1
 router ospf 1
  router-id #$34T#.0.0.1
  network 200.0.0.0 0.0.0.255 area 0
  network 10.#$34T#.#$34T#.0 0.0.0.255 area #$34T#
  network #$34T#.0.0.1 0.0.0.0 area #$34T#
 int gi 0/0/0
  ip ospf network point-to-point
  end
~~~

<br>

~~~
!@CoreBABA
conf t
 ip routing
 no router ospf 1
 router ospf 1
  router-id 10.#$34T#.#$34T#.4
  network 10.#$34T#.0.0 0.0.255.255 area #$34T#
  exit
 int gi 0/1
  ip ospf network point-to-point
  end 
~~~

<br>

~~~
!@CUCM
conf t
 no router ospf 1
 router ospf 1
  router-id 10.#$34T#.100.8
  network 10.#$34T#.100.0 0.0.0.255 area #$34T#
  end
~~~

<br>

~~~
!@windowsCMD
route  add   10.0.0.0   mask   255.0.0.0    10.#$34T#.1.4
route  add  200.0.0.0   mask  255.255.255.0   10.#$34T#.1.4
~~~

<br>
<br>

---
&nbsp;

### Establish Internet Connectivity
~~~
!@EDGE
ping 200.0.0.1
!
conf t
 ip route 0.0.0.0 0.0.0.0 200.0.0.1
 end
~~~

<br>

Configure Network Address Translation
1. Define INSIDE & OUTSIDE
2. Match Traffic
3. Define Translations

<br>

~~~
!@EDGE
conf t
 int lo0
  ip nat inside
  exit
 int g0/0/0
  ip nat inside
  exit
 int g0/0/1
  ip nat outside
  end
~~~

<br>

~~~
!@EDGE
conf t
 ip access-list extended NAT-POLICY
  deny ip 10.#$34T#.0.0 0.0.255.255 10.11.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.12.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.21.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.22.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.31.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.32.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.41.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.42.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.51.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.52.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.61.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.62.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.71.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.72.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.81.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.82.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.91.0.0 0.0.255.255
  deny ip 10.#$34T#.0.0 0.0.255.255 10.92.0.0 0.0.255.255
  no deny ip 10.#$34T#.0.0 0.0.255.255 10.#$34T#.0.0 0.0.255.255
  permit ip any any
  end
~~~

<br>

~~~
!@EDGE
conf t
 ip nat inside source list NAT-POLICY int g0/0/1 overload
 end
~~~

<br>

~~~
!@EDGE
conf t
 ip domain lookup
 ip name-server 10.#$34T#.1.8 10.#$34T#.1.10
 end
~~~

<br>
<br>

---
&nbsp;

### Multipoint Generic Routing Encapsulation
~~~
!@EDGE
conf t
 int tun1
  ip add 172.16.1.#$34T# 255.255.255.0
  tunnel source g0/0/1
  tunnel mode gre multipoint
  no shut
  tun key 123
  ip nhrp authentication C1sc0123
  ip nhrp map multicast dynamic
  ip nhrp network-id 1337
  ip nhrp map 172.16.1.11 200.0.0.11
  ip nhrp map 172.16.1.12 200.0.0.12
  ip nhrp map 172.16.1.21 200.0.0.21
  ip nhrp map 172.16.1.22 200.0.0.22
  ip nhrp map 172.16.1.31 200.0.0.31
  ip nhrp map 172.16.1.32 200.0.0.32
  ip nhrp map 172.16.1.41 200.0.0.41
  ip nhrp map 172.16.1.42 200.0.0.42
  ip nhrp map 172.16.1.51 200.0.0.51
  ip nhrp map 172.16.1.52 200.0.0.52
  ip nhrp map 172.16.1.61 200.0.0.61
  ip nhrp map 172.16.1.62 200.0.0.62
  ip nhrp map 172.16.1.71 200.0.0.71
  ip nhrp map 172.16.1.72 200.0.0.72
  ip nhrp map 172.16.1.81 200.0.0.81
  ip nhrp map 172.16.1.82 200.0.0.82
  ip nhrp map 172.16.1.91 200.0.0.91
  ip nhrp map 172.16.1.92 200.0.0.92
  no ip nhrp map 172.16.1.#$34T# 200.0.0.#$34T#
  end
~~~

<br>

~~~
!@EDGE
conf t
 ip route 10.11.0.0 255.255.0.0 172.16.1.11 252
 ip route 10.12.0.0 255.255.0.0 172.16.1.12 252
 ip route 10.21.0.0 255.255.0.0 172.16.1.21 252
 ip route 10.22.0.0 255.255.0.0 172.16.1.22 252
 ip route 10.31.0.0 255.255.0.0 172.16.1.31 252
 ip route 10.32.0.0 255.255.0.0 172.16.1.32 252
 ip route 10.41.0.0 255.255.0.0 172.16.1.41 252
 ip route 10.42.0.0 255.255.0.0 172.16.1.42 252
 ip route 10.51.0.0 255.255.0.0 172.16.1.51 252
 ip route 10.52.0.0 255.255.0.0 172.16.1.52 252
 ip route 10.61.0.0 255.255.0.0 172.16.1.61 252
 ip route 10.62.0.0 255.255.0.0 172.16.1.62 252
 ip route 10.71.0.0 255.255.0.0 172.16.1.71 252
 ip route 10.72.0.0 255.255.0.0 172.16.1.72 252
 ip route 10.81.0.0 255.255.0.0 172.16.1.81 252
 ip route 10.82.0.0 255.255.0.0 172.16.1.82 252
 ip route 10.91.0.0 255.255.0.0 172.16.1.91 252
 ip route 10.92.0.0 255.255.0.0 172.16.1.92 252
 !
 no ip route 10.#$34T#.0.0 255.255.0.0 172.16.1.#$34T# 252
 end
~~~

<br>

~~~
!@EDGE
conf t
 no router ospf 1
 router ospf 1
  router-id #$34T#.0.0.1
  network 10.#$34T#.#$34T#.0 0.0.0.255 area #$34T#
  network #$34T#.0.0.1 0.0.0.0 area #$34T#
  default-information originate
  end
~~~

&nbsp;
---
&nbsp;

## Site-to-Site VPN (PSK)

Deploy the following VMs
- CSR1000v
- YVM

VPN-PH
- NetAdapter: NAT
- NetAdapter 2: VMNet2
- NetAdapter 3: VMNet3

VPN-JP
- NetAdapter: NAT
- NetAdapter 2: VMNet2
- NetAdapter 3: VMNet4

BLDG-PH
- NetAdapter: VMNet3

BLDG-JP-1
- NetAdapter: VMNet4

BLDG-JP-2
- NetAdapter: VMNet4



!@VPN-PH
conf t
 hostname VPN-PH
 enable secret pass
 service password-encryption
 no logging cons
 no ip domain lookup
 line vty 0 14
  transport input all
  password pass
  login local
  exec-timeout 0 0
 int g1
  ip add 208.8.8.11 255.255.255.0
  no shut
 int g2
  ip add 192.168.102.11 255.255.255.0
  no shut
 int g3
  ip add 11.11.11.113 255.255.255.224
  no shut
 !
 username admin privilege 15 secret pass
 ip http server
 ip http secure-server
 ip http authentication local
 end
wr

!@VPN-JP
conf t
 hostname VPN-JP
 enable secret pass
 service password-encryption
 no logging cons
 no ip domain lookup
 line vty 0 14
  transport input all
  password pass
  login local 
  exec-timeout 0 0
 int g1
  ip add 208.8.8.12 255.255.255.0
  no shut
 int g2
  ip add 192.168.102.12 255.255.255.0
  no shut
 int g3
  ip add 21.21.21.213 255.255.255.240
  ip add 22.22.22.223 255.255.255.192 secondary
  no shut
 !
 username admin privilege 15 secret pass
 ip http server
 ip http secure-server
 ip http authentication local
 end
wr

!@BLDG-PH
sudo su
ifconfig eth0 11.11.11.100 netmask 255.255.255.224 up
route add default gw 11.11.11.113
ping 11.11.11.113

!@BLDG-JP-1
sudo su
ifconfig eth0 21.21.21.211 netmask 255.255.255.240 up
route add default gw 21.21.21.213
ping 21.21.21.213

!@BLDG-JP-2
sudo su
ifconfig eth0 22.22.22.221 netmask 255.255.255.192 up
route add default gw 22.22.22.223
ping 22.22.22.223


#####
Access GUI & Telnet Session
- https://192.168.102.11 (VPN-PH)
- https://192.168.102.12 (VPN-JP)


Activity 01: Configure Site-to-Site Connectivity between BLDG-PH & BLDG-JP-1 & View packets.
Verify by ICMP & SSH



Exercise 01: Configure Site-to-Site VPN for BLDG-PH, BLDG-JP-1, BLDG-JP-2





#####
Site-to-Site VPN (Signature)

Deploy the following VMs:
- NetOps

NetOps
- NetAdapter: NAT
- NetAdapter 2: Bridge (Replicate)
- NetAdapter 3: Host-only
- NetAdapter 4: Host-only

Login: root
Pass: C1sc0123


Identify the NAT IP & connect via SecureCRT

!@NetOps
ip -4 addr


Set a static IP address to connect to the LAN
!@NetOps
nmcli connection add type ethernet con-name static-net2 ifname ens192 ipv4.method manual ipv4.addresses 10.#$34T#.1.6/24 autoconnect yes
nmcli connection up static-net2

Set static routes
!@NetOps
ip route add 10.0.0.0/8 via 10.#$34T#.1.4
ip route add 200.0.0.0/24 via 10.#$34T#.1.4


Activity 02: Create a Selfsigned root CA with the following subject names
For the CA (NetOps)

Country Name (2 letter code) [XX]:PH
State or Province Name (full name) []:NCR
Locality Name (eg, city) [Default City]:Makati
Organization Name (eg, company) [Default Company Ltd]:Rivancorp
Organizational Unit Name (eg, section) []:HQ
Common Name (eg, your name or your server's hostname) []:rivan.com
Email Address []:admin@rivancorp.com

1. Create a directory for the certstore
!@NetOps
mkdir certstore
cd certstore


2. Create a Private key with a Selfsigned Certificate (SSH keys:ssh-keygen vs TLS/SSL keys:openssl)
!@NetOps
openssl req -x509 -newkey rsa:2048 -days 365 -keyout rivan.key -out ca-rivan.crt -nodes

Verify the certificate
!@NetOps
openssl x509 -in ca-rivan.crt -text -noout

3. Create a configuration file to specify subject alternate names.
!@NetOps ext.cnf
###########

[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
x509_extensions    = v3_req
prompt             = no

[ req_distinguished_name ]
C  = PH
ST = NCR
L  = Makati
O  = Rivancorp
OU = HQ
CN = RivanCorporation

[ v3_req ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1   = rivan.com
DNS.2   = www.rivan.com
DNS.3   = api.rivan.com
IP.1    = 10.#$34T#.1.6

###########

4. Generate root CA with subject alt names
!@NetOps
openssl req -x509 -newkey rsa:2048 -days 365 -keyout rivan.key -out ca-rivan.crt -nodes -config ext.cnf -extensions v3_req

Verify:


5. Import CA to Cisco Devices

!@VPN-PH
conf t
 crypto pki trustpoint rivantrust
  enrollment terminal pem
  hash sha512
  subject-name CN=siteph.rivan.com, C=PH, ST=NCR, L=Makati, O=Rivancorp, OU=SitePH, E=siteph@rivancorp.com
  subject-alt-name siteph.rivan.com
  subject-alt-name ph.rivan.com
  storage nvram: 
  primary
  revocation-check none
  rsakeypair rivankeys
  exit
 crypto pki authenticate rivantrust
> Paste the CA


!@VPN-JP
conf t
 crypto pki trustpoint rivantrust
  enrollment terminal pem
  hash sha512
  subject-name CN=siteph.rivan.com, C=JP, ST=Kanto, L=Tokyo, O=Rivancorp, OU=SiteJP, E=sitejp@rivancorp.com
  subject-alt-name sitejp.rivan.com
  subject-alt-name jp.rivan.com
  storage nvram: 
  primary
  revocation-check none
  rsakeypair rivankeys
  exit
 crypto pki authenticate rivantrust
> Paste the CA


6. Generate a CSR for both Routers
!@VPN-PH, VPN-JP
crypto pki enroll rivantrust

> Outputs a CSR . Must be signed by the CA


7. Import the CSR to the CA Server (NetOps)
!@NetOps
nano req-ph.pem
> paste VPN-PH's CSR
> ctrl + s (save)
> ctrl + x (exit)

!@NetOps
nano req-jp.pem
> paste VPN-JP's CSR
> ctrl + s (save)
> ctrl + x (exit


8. Create Configuration files for each VPN Routers
!@NetOps
nano vpnph.cnf
###########

[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
x509_extensions    = v3_req
prompt             = no

[ req_distinguished_name ]
C  = PH
ST = NCR
L  = Makati
O  = Rivancorp
OU = SitePH
CN = RivanCorpPH

[ v3_req ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1   = rivan.ph
DNS.2   = www.rivan.ph
DNS.3   = api.rivan.ph
IP.1    = 208.8.8.11

###########


!@NetOps
nano vpnjp.cnf
###########

[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
x509_extensions    = v3_req
prompt             = no

[ req_distinguished_name ]
C  = JP
ST = Kanto
L  = Tokyo
O  = Rivancorp
OU = SiteJP
CN = RivanCorpJP

[ v3_req ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1   = rivan.jp
DNS.2   = www.rivan.jp
DNS.3   = api.rivan.jp
IP.1    = 208.8.8.12

###########


8. Sign the CSRs
!@NetOps
openssl x509 -req -in req-ph.pem -CA ca-rivan.crt -CAkey rivan.key -out signed-ph.pem -extfile vpnph.cnf -extensions v3_req
openssl x509 -req -in req-jp.pem -CA ca-rivan.crt -CAkey rivan.key -out signed-jp.pem -extfile vpnjp.cnf -extensions v3_req

Verify:

!@NetOps
cat signed-ph.pem
cat signed-jp.pem


9. Import the signed CSRs

!@VPN-PH, VPN-JP
conf t
 crypto pki import rivantrust certificate

> Paste the signed CSR


Verify Cisco Certificates and Trustpoints
!@VPNs
show crypto pki certificates
show crypto pki trustpoint







#####
Site-to-Site VPN (Sign)

1. GRE Tunnel
!@VPN-PH
conf t
 int tun1
  ip add 172.16.10.1 255.255.255.0
  tunnel source g1
  tunnel destination 208.8.8.12
  tunnel mode gre ip
  end

!@VPN-JP
conf t
 int tun1
  ip add 172.16.10.2 255.255.255.0
  tunnel source g1
  tunnel destination 208.8.8.11
  tunnel mode gre ip
  end


2. Routing Interesting traffic
!@VPN-PH
conf t
 ip route 21.21.21.208 255.255.255.240 172.16.10.2
 ip route 22.22.22.192 255.255.255.192 172.16.10.2
 end
 
!@VPN-JP
conf t
 ip route 11.11.11.96 255.255.255.224 172.16.10.1
 end


3. Configure ISAKMP Policy
!@VPN-PH, VPN-JP
conf t
 crypto isakmp policy 1
  authentication rsa-sig
  encryption aes 256
  hash sha512
  group 14
  end


4. Configure IPSec Profile
!@VPN-PH, VPN-JP
conf t
 crypto ipsec transform-set IPSECTUNNEL esp-aes 256 esp-sha-hmac
  mode transport
  exit
 crypto ipsec profile RIVAN
  set transform-set IPSECTUNNEL
  set pfs group14
  end


5 Apply IPSec Profile Protection to Tunnel

!@VPN-PH, VPN-JP
conf t
 int tunnel 1
  tunnel protection ipsec profile RIVAN 
  end


Verify:
!@VPN-PH, VPN-JP
show crypto isakmp sa
show crypto ipsec sa





#####
Open Ports

!@VPN-PH
config t
 int gi 3
  no shut
  ip add 192.168.103.11 255.255.255.0
  ip add 192.168.103.10 255.255.255.0 Secondary
 service finger
 service tcp-small-servers
 service udp-small-servers
 ip dns server
 ip http server
 ip http secure-server
 ip host www.web310.com 192.168.103.10
 ip host www.web311.com 192.168.103.11
 ip name-server 8.8.8.8 1.1.1.1
 ip route 0.0.0.0 0.0.0.0 208.8.8.2
 ip domain lookup
 end

ENABLE VoIP Features
!@UTM-PH
conf t
 no telephony-service
 telephony-service
  no auto assign
  no auto-reg-ephone
  max-ephones 5
  max-dn 20
  ip source-address 208.8.8.11 port 2000
  exit
 voice service voip
  allow-connections h323 to sip
          
  allow-connections sip to h323
  allow-connections sip to sip
  supplementary-service h450.12
 sip
   bind control source-interface g1
   bind media source-interface g1
   registrar server expires max 600 min 60
 voice register global
  mode cme
  source-address 208.8.8.11 port 5060
  max-dn 12
  max-pool 12
  authenticate register
  create profile sync syncinfo.xml
  end

Modify the hosts file: c:\Windows\system32\drivers\etc\hosts
Scan the sites:


www.sti.edu.ph    vs    www.cia.gov     vs     www.neu.edu.ph

@cmd
nmap -v www.sti.edu.ph
nmap -v www.dlsu.edu.ph
nmap -v www.neu.edu.ph

Activity 03: Open the following ports for www.web310.com
Allow: HTTP, HTTPS, DNS, SSH

Activity 04: Open the following ports for www.web310.com
Allow: TELNET, SCCP, SIP, SMTP, IMAP




















































----
SSH Key Authentication
Privilege Access Management

1. Jumpserver
- VPN-PH
- VPN-JP
- CBABA
- EDGE

SSH Authentication Methods
- Password
- Public Key Authentication



---
1. Password Authentication

1. Ephemeral KeyPair

V
Server             Client
Priv & Pub Keys    Priv & Pub Keys

Pub Key Exchange

- Combine Pub Key
Symmetrical Encryption


Create a user account on linux: rivan
!@NetOps
adduser -m rivan
passwd rivan
> C1sc0123
> C1sc0123

Make Rivan a Sudoer
!@NetOps
usermod -aG wheel rivan

Login as rivan
!@NetOps
su rivan
> C1sc0123


2. Publickey Authentication

LINUX

Create a user account on linux: alumni

!@NetOps
adduser -m alumni -p C1sc0123 -G wheel

Login as alumni
!@NetOps
su alumni
> C1sc0123

Create SSH Key Pair directory
!@NetOps
cd /home/alumni
mkdir .ssh
mkdir .ssh/sshstore
cd .ssh/sshstore

Generate Key Pair for alumni
!@NetOps
ssh-keygen -t rsa -b 2048 -f alumni
cat alumni.pub >> ../authorized_keys

Decrypt private key
!@NetOps
ssh-keygen -p

Disable Password Authentication
!@NetOps
nano /etc/ssh/sshd_config

PasswordAuthentication no

systemctl restart sshd


CISCO

Create a user account in Cisco with a key pair
!@CSR1000v & CoreTAAS
conf t
 line vty 0 14
  transport input all
  login local
  exec-timeout 0 0
  exit
 ip domain name rivan.com
 username admin privilege 15 secret pass
 username rivan privilege 15 
 crypto key generate rsa modulus 2048 label sec
 ip ssh rsa keypair-name sec
 ip ssh version 2
 end

Assign a public key to rivan
!@CSR1000v & CoreTAAS
conf t
 ip ssh pubkey-chain
  username rivan
   key-string

Paste the public key

On Jumpserver, enter the private key to access CoreTAAS & CSR1000v


Turn off password authentication for Cisco and Linux.
@NetOps
cd /etc/ssh/sshd_config

@UTM-PH
conf t
 no ip ssh server algorithm authentication password
 end
 
 
3. GSSAPI Authentication - KERBEROS - Token Authentication

Install Kerberos Server
sudo yum install krb5-server krb5-libs krb5-workstation

Set a default realm for the config file:
nano /etc/krb5.conf

    [libdefaults]
        default_realm = EXAMPLE.COM

    [realms]
        EXAMPLE.COM = {
            kdc = kerberos.example.com
            admin_server = kerberos.example.com
        }

    [domain_realm]
        .example.com = EXAMPLE.COM
        example.com = EXAMPLE.COM

        
Create Kerberos database
kdb5_util create -s

Make sure the default re
sudo kadmin.local 
addprinc admin/admin
exit

Start kerberos services
sudo systemctl start krb5kdc.service 
sudo systemctl start kadmin

kinit
klist



----

Firewall, ACLs & Port Forwarding
Cloud Meraki.

!@UTM-PH
config t
 int gi 3
  no shut
  ip add 192.168.103.11 255.255.255.0
  ip add 192.168.103.10 255.255.255.0 Secondary
 service finger
 service tcp-small-servers
 service udp-small-servers
 ip dns server
 ip http server
 ip http secure-server
 ip host www.web310.com 192.168.103.10
 ip host www.web311.com 192.168.103.11
 end

Review: Create a DNS A record for www.ccna#$34T#.com
192.168.103.10 www.web310.com
192.168.103.11 www.web311.com
  
  or
  
Modify the hosts file: c:\Windows\system32\drivers\etc\hosts


Scan the sites:

@cmd
nmap -v www.web310.com
nmap -v www.web311.com




!@BLDG-1
!@BLDG-2


Logging SNORT - Windows Event Manager


&&

BUT FIRST: Go to UTM-PH GUI and activate CME

Then,

@UTM-PH
conf t
 no telephony-service
 telephony-service
  no auto assign
  no auto-reg-ephone
  max-ephones 5
  max-dn 20
  ip source-address 208.8.8.11 port 2000
  exit
 voice service voip
  allow-connections h323 to sip
          
  allow-connections sip to h323
  allow-connections sip to sip
  supplementary-service h450.12
 sip
   bind control source-interface g1
   bind media source-interface g1
   registrar server expires max 600 min 60
 voice register global
  mode cme
  source-address 208.8.8.11 port 5060
  max-dn 12
  max-pool 12
  authenticate register
  create profile sync syncinfo.xml
  end
---



1. Make the www.rivan#$34T#.com accessible via port 8080

2. Make the VM Jumpserver, accessible via port 
-Well-known ports (0-1023) - reserved for known services
-Registered ports (1024-49151) - free to use
-Dynamic/Private ports (49152-65535) - ephemeral

!@EDGE
Block Classmates


!@UTM - L3 Firewall Rule
Block Porn
Block Schools

L4 Open Ports for your public IP


L1-4 Normal Firewall
L7 - NGFW

!@Meraki
Manage L7 Firewall Policy


---


CREATING A WEB PROXY OR HIDING BEHIND NAT:

@ping
www.sti.edu.ph:      vs     www.dlsu.edu.ph:

@BLDG-1
sudo su
ifconfig eth0 192.168.103.21 netmask 255.255.255.0 up
route add default gw 192.168.103.11
ping 192.168.103.11

@BLDG-2
sudo su
ifconfig eth0 192.168.103.22 netmask 255.255.255.0 up
route add default gw 192.168.103.11
ping 192.168.103.11

@BLDG-3
sudo su
ifconfig eth0 192.168.103.23 netmask 255.255.255.0 up
route add default gw 192.168.103.11


@UTM-PH
config t
 int gi 1
  ip nat OUTSIDE
 int gi 2
  ip nat INSIDE
 int gi 3
  ip nat INSIDE
 no access-list 8
 access-list 8 permit 192.168.102.0 0.0.0.255
 access-list 8 permit 192.168.103.0 0.0.0.255
 ip nat inside source list 8 interface Gi 1 overload
 end
show ip nat translations 

@BLDGS
ping   8.8.8.8    4.4.4.4     8.8.4.4


Access the web of BLDG first.
192.168.103.21
192.168.103.22
192.168.103.23


Now hide behind the firewall.


@UTM-PH
config t
IP Nat inside source static tcp 192.168.103.21 80 208.8.8.101 8080
end
show ip nat translation

Now open 208.8.8.101:8080 on browser

---



Rate Limitting (CoPP)

CONTROL PLANE POLICING: monitor all data entering
and leaving the control plane: ALL the interfaces:g1,g2,g3.
ExamTask: limit the ping to 8000 packets/sec! cm-pm-sp
config t
ip access-list extended ABUSEPING
 permit icmp any any
class-map STOPPING
 match access-group name ABUSEPING
no policy-map PINGSTOP
policy-map PINGSTOP
 class STOPPING
 police 8000 conform-action transmit exceed-action drop
 exit
 control-plane
  service-policy input PINGSTOP
end
sh policy-map control-plane

TASK2: limit, telnet, ssh, and https:  CM - PM  -SP
config t
ip access-list extended TELNET
 permit tcp any any eq 23
ip access-list extended SSH
 permit tcp any any eq 22
ip access-list extended HTTPS
 permit tcp any any eq 443
ip access-list extended ICMP
 permit icmp any any
Class-map match-all CMTELNET
 match access-group name TELNET
 exit
class-map match-all CMSSH
 match access-group name SSH
 exit
class-map match-all CMHTTPS
 match access-group name HTTPS
 exit
class-map match-all CMICMP
 match access-group name ICMP
 exit
Policy-map PMCOPP
class CMTELNET
 police 10000 conform-action drop exceed-action drop
class CMSSH
 police 800000  conform-action transmit exceed-action transmit
class CMHTTPS
 police 600000  conform-action transmit exceed-action transmit
class CMICMP
  police rate 4 pps conform-action transmit exceed-action drop
class class-default
 police 12000 conform-action transmit exceed-action transmit
 exit
control-plane
 service-policy input PMCOPP
END


Linux Rate Limitting


---

Incident Identification

Syslog
Severity levels

!@UTM
Terminal Monitor


System & Security Logs
- Application (NetFlow)
- Security (Audits : Event Manager)
- System (Task Manager)
- Setup (dir /ah)
- Forwarded Events


Port Mirroring
- Capture RTP


---

NetFlow

Flow Record
Flow Exporter
Flow Monitor

config t
flow record CCNP8-CUSTOM-OUT
 description Custom Flow Record for outbound traffic 
  match ipv4 destination address 
  match transport destination-port 
  collect counter bytes 
  collect counter packets 
  END
@@@CREATE A FLOW EXPORTER:
config t
 flow exporter CCNP8-COLLECTOR-HOST
 destination 192.168.104.1
 export-protocol  netflow-v9 
 transport UDP 9999 
 end
@@@ COMBINE flow monitor and flow record:
config t
flow monitor  CCNP8-INBOUND-MONITOR 
  record netflow ipv4 original-input
  cache timeout active 30
  exporter CCNP8-COLLECTOR-HOST
flow monitor  CCNP8-OUTBOUND-MONITOR
  record CCNP8-CUSTOM-OUT
  cache timeout active 30
  exporter CCNP8-COLLECTOR-HOST
  exit
@@@Define the Interface to Monitor: SiteA/ SiteB:
config t
Int Gi 3
 ip flow monitor CCNP8-INBOUND-MONITOR input 
 ip flow monitor CCNP8-INBOUND-MONITOR output
 end
 
 
