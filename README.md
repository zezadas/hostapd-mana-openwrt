this is a fork from adde88/hostapd-mana-openwrt with some fixes that I run on openwrt-x86 KVM VM with UEFI

Includes:
 - hostapd-mana ported to openwrt by adde88
 - wpa-sycophant
 - berate_ap
 - fire-lamb running on python3 - ported by zezadas (me) 

Quick How to compile
 - Download openwrt SDK for your device
 - cp hostapd-mana-openwrt/libs <SDK-DIR>/package/ -r 
 - cp hostapd-mana-openwrt/net/ <SDK-DIR>/package/ -r
 - cd <SDK-DIR>
 - ./scripts/feeds update -a  && make package/symlinks
 - make package/net/hostapd-mana/compile

 - after compiling copy te needed ipk files to your openwrt device and install them.


TODO:
 - have better README

