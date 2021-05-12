# Howto make license request for Neuromag Software on ubuntu 20

## 1. Switch link /bin/sh to bash, becouse by default it point to dash

```bash
ln -sf /bin/bash /bin/sh
```

## 2. Necessary rename active network device to old naming scheme eth* 

In modern linux simple way is using netplan manifest.

There is need add option for rename device which matched by it macaddr. E.g. as described below

```bash
cat /etc/netplan/99-network.yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 192.168.1.15/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 192.168.1.1
        search:
          - host.localnet
      match:
        macaddress: 00:25:90:c9:37:34
      set-name: eth0
```

For apply it need reboot host.

## 3. License probe utility use 32-bit code.

It need system wide fixing

```bash
sudo dpkg --add-architecture i386
sudo apt-get update
sudo apt-get install libc6:i386 libncurses5:i386 libstdc++6:i386 -y
```

## 4. The usual way of placing neuromag software is /opt/neuromag

So it should be done


## 5. Generation script is very ancient

At first need to install depricated utility

```bash
sudo apt-get install net-utils -y
```

Patch file need for fixes old and incorrect statements.

```bash
diff -Naur license_info.orig license_info
--- license_info.orig	2021-04-26 09:54:07.000000000 +0300
+++ license_info	2021-05-12 16:22:27.117134625 +0300
@@ -17,7 +17,8 @@
 #================================================================================
 if [ `uname` = "Linux" ] ; then
     ifbasename=eth
-    ifconfig="/sbin/ifconfig -a"
+    ## lol: ifconfig="/sbin/ifconfig -a"
+    ifconfig="$(which ifconfig) -a"
     awk_extract_lanpars='{sub("addr:","",$2); sub("Bcast:","",$3); sub("Mask:","",$4); print "ipaddr[" no "]=" $2 "\nnetmask[" no "]=" $3 "\nbroadcast[" no "]=" $4}'

     # Line: sed '$!N;s/\n/ /'` concatenates the lines on a single line (i.e. removes line changes)
@@ -91,8 +92,10 @@
     interface=$1
     lanno=`number_part $interface`
     if [ `uname` = 'Linux' ] ; then
-	$ifconfig | grep -e "$interface.*HWaddr" |
-	    awk '{apu=$1; apu=substr(apu,match(apu,"[0-9]")); $0=substr($0,match($0,"HWaddr")); gsub(":","",$2); print "macaddr[" apu "]=0x" $2}'
+	## ancient: $ifconfig | grep -e "$interface.*HWaddr" |
+        curr_mc=$($ifconfig | sed "0,/^${interface}/d" | grep '\s*ether\s' | head -n 1 | awk '{print "0x"toupper($2)}' | tr -d ':') ##'
+	## lol:    awk '{apu=$1; apu=substr(apu,match(apu,"[0-9]")); $0=substr($0,match($0,"HWaddr")); gsub(":","",$2); print "macaddr[" apu "]=0x" $2}'
+        echo macaddr[$lanno]=$curr_mc
     else
 	printf "macaddr[$lanno]="
 	echo `/usr/sbin/lanscan | grep $interface | awk '{ print $2 }'`
```


Then need apply patch to probe script like follow

```bash
patch license_info patch.txt
```

## 6. Finally, we can make a license request

```bash
/opt/neuromag/bin/admin/license_info
#
# Workstation spec for swift
#
# Generated: Wed May 12 15:57:08 MSK 2021
#
hostname=swift
domainname=configuredconfigured
host.localnet
uname="Linux swift 5.4.0-72-generic #80-Ubuntu SMP Mon Apr 12 17:35:00 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux"
model="Intel(R) Xeon(R) CPU E5-2680 v2 @ 2.80GHz (x40)"
# Configured interfaces: eth0
macaddr[0]=0x001E67CD0384
ipaddr[0]=192.168.1.17
netmask[0]=netmask
broadcast[0]=255.255.255.0
defroute=192.168.1.1
nisdomain=neuromag
timezone=MSK
probe="
++ HOST swift ++++++++++++++++++++++++++++++++++++++++++++++++++
040************************881
522************************50F
----------------------------------------------------------------
"
#
products="dana-x.y.z maxfilter-q.w.e"
```
