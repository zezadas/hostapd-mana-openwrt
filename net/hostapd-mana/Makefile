#
# Copyright (C) 2019 Andreas Nilsen <adde88@gmail.com> https://github.com/adde88
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=hostapd-mana
PKG_VERSION:=2.6.5
PKG_RELEASE:=18

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)

PKG_MAINTAINER:=Andreas Nilsen <adde88@gmail.com>

PKG_LICENSE:=GPL-2.0
PKG_LICENSE_FILES:=COPYING

PKG_BUILD_PARALLEL:=1

PKG_CONFIG_DEPENDS:= \
	CONFIG_PACKAGE_kmod-ath9k \
	CONFIG_PACKAGE_kmod-cfg80211 \
	CONFIG_PACKAGE_hostapd \
	CONFIG_PACKAGE_hostapd-mini \
	CONFIG_PACKAGE_kmod-hostap \
	CONFIG_DRIVER_WEXT_SUPPORT \
	CONFIG_DRIVER_11N_SUPPORT \
	CONFIG_DRIVER_11AC_SUPPORT \
	CONFIG_DRIVER_TAXONOMY_SUPPORT

include $(INCLUDE_DIR)/package.mk

STAMP_CONFIGURED:=$(STAMP_CONFIGURED)

ifneq ($(CONFIG_DRIVER_11N_SUPPORT),)
	HOSTAPD_IEEE80211N:=y
endif

ifneq ($(CONFIG_DRIVER_11AC_SUPPORT),)
	HOSTAPD_IEEE80211AC:=y
endif

ifneq ($(CONFIG_DRIVER_TAXONOMY),)
	CONFIG_DRIVER_TAXONOMY:=y
endif

DRIVER_MAKEOPTS= \
	CONFIG_ACS=$(CONFIG_PACKAGE_kmod-cfg80211) \
	CONFIG_DRIVER_NL80211=$(CONFIG_PACKAGE_kmod-cfg80211) \
	CONFIG_IEEE80211N=$(HOSTAPD_IEEE80211N) \
	CONFIG_IEEE80211AC=$(HOSTAPD_IEEE80211AC) \
	CONFIG_IEEE80211W=$(CONFIG_DRIVER_11W_SUPPORT) \
	CONFIG_DRIVER_WEXT=$(CONFIG_DRIVER_WEXT_SUPPORT) \
	CONFIG_DRIVER_TAXONOMY=$(CONFIG_DRIVER_TAXONOMY) \

space :=
space +=

ifdef CONFIG_USE_GLIBC
	TARGET_LDFLAGS += -lrt
	TARGET_LDFLAGS_C += -lrt
endif

DRV_DEPENDS:=+PACKAGE_kmod-cfg80211:libnl

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	
	if [ ! -d "./src/hostapd-mana/" ] ; then git clone https://github.com/adde88/hostapd-mana.git ; cp -r hostapd-mana/ ./src/hostapd-mana;  fi
	
	if [ ! -d "./src/wpa_sycophant" ] ; then git clone https://github.com/sensepost/wpa_sycophant.git; cp -r wpa_sycophant/ ./src/wpa_sycophant; fi #wpa_supplicant
	
	if [ ! -d "./src/berate_ap" ] ; then git clone https://github.com/sensepost/berate_ap.git; cp -r berate_ap/ ./src/berate_ap; fi
	
	if [ ! -d "./src/net-creds" ] ; then git clone https://github.com/zezadas/net-creds-python3.git net-creds; cp -r net-creds/ ./src/net-creds; fi
	
	#Firelamb? https://github.com/sensepost/mana/tree/master/firelamb - not feeling it. feel free to send a pulll request.
	
	$(CP) -rf ./src/* $(PKG_BUILD_DIR)/
endef

ifneq ($(wildcard $(PKG_BUILD_DIR)/.config_*),$(subst .configured_,.config_,$(STAMP_CONFIGURED)))
  define Build/Configure/rebuild
	$(FIND) $(PKG_BUILD_DIR) -name \*.o -or -name \*.a | $(XARGS) rm -f
	rm -f $(PKG_BUILD_DIR)/hostapd-mana/hostapd/hostapd
	rm -f $(PKG_BUILD_DIR)/wpa_sycophant/wpa_supplicant/wpa_supplicant
	rm -f $(PKG_BUILD_DIR)/.config_*
	touch $(subst .configured_,.config_,$(STAMP_CONFIGURED))
  endef
endif

define Build/Configure
	$(Build/Configure/rebuild)
	$(if $(wildcard ./hostapd-mana/files/hostapd-mana.config), \
		$(CP) ./hostapd-mana/files/hostapd-mana.config $(PKG_BUILD_DIR)/hostapd-mana/hostapd/.config \
	)
endef

TARGET_CPPFLAGS := \
	-I$(STAGING_DIR)/usr/include/libnl3 \
	-I$(PKG_BUILD_DIR)/src/crypto \
	$(TARGET_CPPFLAGS) \
	-DCONFIG_LIBNL30 \
	-D_GNU_SOURCE \

TARGET_CFLAGS += -ffunction-sections -fdata-sections -flto
TARGET_LDFLAGS += -Wl,--gc-sections -flto=jobserver -fuse-linker-plugin -lubox -lubus -lm -lnl-3 -lnl-genl-3 -lnl-route-3 -ldl 
TARGET_LDFLAGS += -lcrypto -lssl #flags for compiling wpa_sycophant

define Build/Compile
	echo "LOCAL TYPE: $(LOCAL_TYPE)"
	echo "BUILD VARIANT: $(BUILD_VARIANT)"
	$(Build/Compile/hostapd-mana)
	$(Build/Compile/wpa_sycophant)
	$(Build/Compile/berate_ap)
	$(Build/Compile/full)
endef

define Build/RunMake
	CFLAGS="$(TARGET_CPPFLAGS) $(TARGET_CFLAGS)" \
	$(MAKE) $(PKG_JOBS) -C $(PKG_BUILD_DIR)/$(1) \
		CC="$(TARGET_CC)" \
	$(TARGET_CONFIGURE_OPTS) \
	$(DRIVER_MAKEOPTS) \
	LIBS="$(TARGET_LDFLAGS)" \
	LIBS_c="$(TARGET_LDFLAGS_C)" \
	AR="$(TARGET_CROSS)gcc-ar" \
	BCHECK= \
	$(2)
endef








define Package/hostapd-mana/config
	source "$(SOURCE)/Config.in"
endef

define Package/hostapd-mana
	SECTION:=net
	CATEGORY:=Network
	TITLE:=Hostapd with support for the MANA and Karma attacks.
	DEPENDS:=$(DRV_DEPENDS) +libubus +libopenssl +libpthread +libnl
	URL:=https://github.com/sensepost/hostapd-mana
	TITLE:=hostapd-mana
endef

define Package/hostapd-mana/description
	This package contains hostapd with support for the MANA/KARMA attacks.
endef

define Build/Compile/hostapd-mana
	+$(call Build/RunMake,hostapd-mana/hostapd, \
		hostapd hostapd_cli \
	)
endef


define Install/hostapd-mana
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_DIR) $(1)/etc/hostapd-mana/
	$(INSTALL_DIR) $(1)/etc/hostapd-mana/cert
endef


define Package/hostapd-mana/install
	$(call Install/hostapd-mana,$(1))
	$(INSTALL_BIN)  $(PKG_BUILD_DIR)/hostapd-mana/hostapd/hostapd			$(1)/usr/sbin/hostapd-mana
	$(INSTALL_BIN)  $(PKG_BUILD_DIR)/hostapd-mana/hostapd/hostapd_cli		$(1)/usr/sbin/mana_cli
	$(INSTALL_BIN)	$(PKG_BUILD_DIR)/hostapd-mana/files/crackapd/crackapd.py	$(1)/usr/sbin/crackapd
	$(INSTALL_DATA)	$(PKG_BUILD_DIR)/hostapd-mana/files/crackapd/crackapd.conf	$(1)/etc/hostapd-mana/
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/hostapd-mana/files/conf/*			$(1)/etc/hostapd-mana/
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/hostapd-mana/files/cert/*			$(1)/etc/hostapd-mana/cert/
endef














define Package/wpa_sycophant
	SECTION:=net
	CATEGORY:=Network
	TITLE:=WPA Supplicant
	URL:=https://github.com/sensepost/wpa_sycophant
	DEPENDS:=$(DRV_DEPENDS) +libubus +libopenssl +libpthread +libnl  #+hostapd-common
	PROVIDES:=wpa_sycophant
endef


define Package/wpa_sycophant/description
	This package contains hostapd with support for the MANA/KARMA attacks.
endef

define Package/wpa_sycophant/config
	source "$(SOURCE)/Config.in"
endef

define Build/Compile/wpa_sycophant
	+$(call Build/RunMake,wpa_sycophant/wpa_supplicant, \
		wpa_cli wpa_supplicant \
	)
	sed 's/supplicant=.*/supplicant=\/usr\/sbin\/wpa_sycophant/g' -i $(PKG_BUILD_DIR)/wpa_sycophant/wpa_sycophant.sh
endef

define Install/wpa_sycophant
	$(INSTALL_DIR) $(1)/usr/sbin
	#$(INSTALL_DIR) $(1)/etc/hostapd-mana/
	#$(INSTALL_DIR) $(1)/etc/hostapd-mana/cert
endef

define Package/wpa_sycophant/install
	$(call Install/wpa_sycophant,$(1))
	$(INSTALL_BIN)  $(PKG_BUILD_DIR)/wpa_sycophant/wpa_supplicant/wpa_supplicant	$(1)/usr/sbin/wpa_sycophant
	$(INSTALL_BIN)  $(PKG_BUILD_DIR)/wpa_sycophant/wpa_sycophant.sh			$(1)/usr/sbin/wpa_sycophant.sh
	#$(INSTALL_BIN)  $(PKG_BUILD_DIR)/wpa_sycophant/wpa_supplicant/wpa_cli		$(1)/usr/sbin/wpa_cli
endef








define Package/berate_ap
	SECTION:=net
	CATEGORY:=Network
	TITLE:=Berate_ap
	URL:=https://github.com/sensepost/berate_ap
	DEPENDS:=$(DRV_DEPENDS) +bash +getopt
	PROVIDES:=berate_ap
endef


define Package/berate_ap/description
	Create_ap has been modified to run EAP networks and hostapd-mana with multiple options.
endef

define Package/berate_ap/config
	source "$(SOURCE)/Config.in"
endef

define Build/Compile/berate_ap
	sed  -i "s/cp -n/cp -u/g" $(PKG_BUILD_DIR)/berate_ap/berate_ap
endef

define Install/berate_ap
	$(INSTALL_DIR) $(1)/usr/sbin
endef

define Package/berate_ap/install
	$(call Install/berate_ap,$(1))
	$(INSTALL_BIN)	$(PKG_BUILD_DIR)/berate_ap/berate_ap			$(1)/usr/sbin/berate_ap
endef




define Package/net-creds
	SECTION:=net
	CATEGORY:=Network
	TITLE:=net-creds
	URL:=https://github.com/DanMcInerney/net-creds
	DEPENDS:=$(DRV_DEPENDS) +scapy 
	PROVIDES:=net-creds
endef


define Package/net-creds/description
	Thoroughly sniff passwords and hashes from an interface or pcap file. Concatenates fragmented packets and does not rely on ports for service identification.
endef

define Package/net-creds/config
	source "$(SOURCE)/Config.in"
endef

define Build/Compile/net-creds
	true
endef

define Install/net-creds
	$(INSTALL_DIR) $(1)/usr/sbin
endef

define Package/net-creds/install
	$(call Install/net-creds,$(1))
	$(INSTALL_BIN)	$(PKG_BUILD_DIR)/net-creds/net-creds.py			$(1)/usr/sbin/net-creds
endef



$(eval $(call BuildPackage,hostapd-mana))
$(eval $(call BuildPackage,wpa_sycophant))
$(eval $(call BuildPackage,berate_ap))
$(eval $(call BuildPackage,net-creds))
