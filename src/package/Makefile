#
# Copyright (C) 2006-2014 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=hostapd-wpe
PKG_VERSION:=git
PKG_RELEASE:=1

PKG_SOURCE_URL:=http://w1.fi/hostap.git
PKG_SOURCE_PROTO:=git
PKG_SOURCE_DATE:=2019-08-08
PKG_SOURCE_VERSION:=ca8c2bd28ad53f431d6ee60ef754e98cfdb4c17b
PKG_MIRROR_HASH:=9d9f1c60afa5324ee17219bd3ec61c1a6fa4043b4187da9bb44e59025d3ed31d

PKG_MAINTAINER:=Jeroen Peelaerts <jeroen.peelaerts@gmail.com>, Original Openwrt package: Mallo - Tarlogic Security <oscar.mallo@tarlogic.com>, Original hostapd package: Felix Fietkau <nbd@openwrt.org>

PKG_BUILD_PARALLEL:=1

PKG_CONFIG_DEPENDS:= \
	CONFIG_PACKAGE_kmod-ath9k \
	CONFIG_PACKAGE_kmod-mac80211 \
	CONFIG_WPA_RFKILL_SUPPORT \
	CONFIG_DRIVER_WEXT_SUPPORT \
	CONFIG_DRIVER_11N_SUPPORT \
	CONFIG_DRIVER_11AC_SUPPORT \

EAPOL_TEST_PROVIDERS:=

SUPPLICANT_PROVIDERS:=
HOSTAPD_PROVIDERS:=

LOCAL_TYPE=$(strip \
		$(if $(findstring wpad,$(BUILD_VARIANT)),wpad, \
		$(if $(findstring supplicant,$(BUILD_VARIANT)),supplicant, \
		hostapd \
		)))

LOCAL_AND_LIB_VARIANT=$(patsubst hostapd-%,%,\
		      $(patsubst wpad-%,%,\
		      $(patsubst supplicant-%,%,\
		      $(BUILD_VARIANT)\
		      )))

LOCAL_VARIANT=$(patsubst %-internal,%,\
	      $(patsubst %-openssl,%,\
	      $(patsubst %-wolfssl,%,\
	      $(LOCAL_AND_LIB_VARIANT)\
	      )))

SSL_VARIANT=$(strip \
		$(if $(findstring openssl,$(LOCAL_AND_LIB_VARIANT)),openssl,\
		$(if $(findstring wolfssl,$(LOCAL_AND_LIB_VARIANT)),wolfssl,\
		internal\
		)))

CONFIG_VARIANT:=$(LOCAL_VARIANT)
ifeq ($(LOCAL_VARIANT),mesh)
  CONFIG_VARIANT:=full
endif

include $(INCLUDE_DIR)/package.mk

STAMP_CONFIGURED:=$(STAMP_CONFIGURED)_$(CONFIG_WPA_MSG_MIN_PRIORITY)


ifneq ($(CONFIG_DRIVER_11N_SUPPORT),)
  HOSTAPD_IEEE80211N:=y
endif

ifneq ($(CONFIG_DRIVER_11AC_SUPPORT),)
  HOSTAPD_IEEE80211AC:=y
endif

DRIVER_MAKEOPTS= \
	CONFIG_ACS=$(CONFIG_PACKAGE_kmod-cfg80211) \
	CONFIG_DRIVER_NL80211=$(CONFIG_PACKAGE_kmod-cfg80211) \
	CONFIG_IEEE80211N=$(HOSTAPD_IEEE80211N) \
	CONFIG_IEEE80211AC=$(HOSTAPD_IEEE80211AC) \
	CONFIG_DRIVER_WEXT=$(CONFIG_DRIVER_WEXT_SUPPORT) \

ifneq ($(LOCAL_VARIANT),mini)
  DRIVER_MAKEOPTS += CONFIG_IEEE80211W=$(CONFIG_DRIVER_11W_SUPPORT)
endif

ifeq ($(LOCAL_VARIANT),full)
  ifeq ($(SSL_VARIANT),openssl)
    DRIVER_MAKEOPTS += CONFIG_TLS=openssl CONFIG_SAE=y CONFIG_OWE=y CONFIG_SUITEB192=y CONFIG_AP=y CONFIG_MESH=y
    TARGET_LDFLAGS += -lcrypto -lssl
  endif
  ifeq ($(SSL_VARIANT),wolfssl)
    DRIVER_MAKEOPTS += CONFIG_TLS=wolfssl CONFIG_WPS_NFC=1 CONFIG_SAE=y CONFIG_OWE=y CONFIG_SUITEB192=y CONFIG_AP=y CONFIG_MESH=y
    TARGET_LDFLAGS += -lwolfssl
  endif
endif

ifneq ($(LOCAL_TYPE),hostapd)
  ifeq ($(LOCAL_VARIANT),mesh)
    ifeq ($(SSL_VARIANT),openssl)
      DRIVER_MAKEOPTS += CONFIG_TLS=openssl CONFIG_AP=y CONFIG_SAE=y CONFIG_MESH=y
      TARGET_LDFLAGS += -lcrypto -lssl
    endif
    ifeq ($(SSL_VARIANT),wolfssl)
      DRIVER_MAKEOPTS += CONFIG_TLS=wolfssl CONFIG_WPS_NFC=1 CONFIG_AP=y CONFIG_SAE=y CONFIG_MESH=y
      TARGET_LDFLAGS += -lwolfssl
    endif
  endif

  ifdef CONFIG_WPA_RFKILL_SUPPORT
    DRIVER_MAKEOPTS += NEED_RFKILL=y
  endif
endif

ifdef CONFIG_USE_EGLIBC
  TARGET_LDFLAGS += -lrt
  TARGET_LDFLAGS_C += -lrt
endif

DRV_DEPENDS:=+PACKAGE_kmod-cfg80211:libnl-tiny

define Package/hostapd-wpe/Default
  SECTION:=net
  CATEGORY:=Network
  TITLE:=IEEE 802.1x Authenticator with WPE support
  DEPENDS:=$(DRV_DEPENDS) +hostapd-common +libubus
  CONFLICTS:=$(HOSTAPD_PROVIDERS)
  HOSTAPD_PROVIDERS+=$(1)
endef

define Package/hostapd-wpe
$(call Package/hostapd-wpe/Default,$(1))
  TITLE+= (full)
  VARIANT:=full-internal
endef

define Package/hostapd-wpe/description
 This package contains a full featured IEEE 802.1x/WPA/EAP/RADIUS Authenticator with WPE patch support.
endef

ifneq ($(wildcard $(PKG_BUILD_DIR)/.config_*),$(subst .configured_,.config_,$(STAMP_CONFIGURED)))
  define Build/Configure/rebuild
	$(FIND) $(PKG_BUILD_DIR) -name \*.o -or -name \*.a | $(XARGS) rm -f
	rm -f $(PKG_BUILD_DIR)/hostapd/hostapd-wpe
	rm -f $(PKG_BUILD_DIR)/wpa_supplicant/wpa_supplicant
	rm -f $(PKG_BUILD_DIR)/.config_*
	touch $(subst .configured_,.config_,$(STAMP_CONFIGURED))
  endef
endif

define Build/Configure
	$(Build/Configure/rebuild)
	$(if $(wildcard ./files/hostapd-$(LOCAL_VARIANT).config), \
		$(CP) ./files/hostapd-$(LOCAL_VARIANT).config $(PKG_BUILD_DIR)/hostapd/.config \
	)
	$(CP) ./files/wpa_supplicant-$(LOCAL_VARIANT).config $(PKG_BUILD_DIR)/wpa_supplicant/.config
endef

TARGET_CPPFLAGS := \
	-I$(STAGING_DIR)/usr/include/libnl-tiny \
	-I$(PKG_BUILD_DIR)/src/crypto \
	$(TARGET_CPPFLAGS) \
	-DCONFIG_LIBNL20 \
	-D_GNU_SOURCE \
	$(if $(CONFIG_WPA_MSG_MIN_PRIORITY),-DCONFIG_MSG_MIN_PRIORITY=$(CONFIG_WPA_MSG_MIN_PRIORITY))

TARGET_CFLAGS += -ffunction-sections -fdata-sections -flto
TARGET_LDFLAGS += -Wl,--gc-sections -flto=jobserver -fuse-linker-plugin -lubox -lubus

ifdef CONFIG_PACKAGE_kmod-cfg80211
  TARGET_LDFLAGS += -lm -lnl-tiny
endif

define Build/RunMake
	CFLAGS="$(TARGET_CPPFLAGS) $(TARGET_CFLAGS)" \
	$(MAKE) $(PKG_JOBS) -C $(PKG_BUILD_DIR)/$(1) \
		$(TARGET_CONFIGURE_OPTS) \
		$(DRIVER_MAKEOPTS) \
		LIBS="$(TARGET_LDFLAGS)" \
		LIBS_c="$(TARGET_LDFLAGS_C)" \
		AR="$(TARGET_CROSS)gcc-ar" \
		BCHECK= \
		$(2)
endef

define Build/Compile/hostapd-wpe
	$(call Build/RunMake,hostapd, \
		hostapd-wpe hostapd-wpe_cli \
	)
endef

define Build/Compile/hostapd-wpe-certs
	$(call Build/RunMake,hostapd/certs)
endef

define Build/Compile
	echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> LOCAL TYPE = $(LOCAL_TYPE)"
	echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> BUILD VARIANT = $(BUILD_VARIANT)"
	$(Build/Compile/hostapd-wpe)
	$(Build/Compile/hostapd-wpe-certs)
	$(Build/Compile/full)
endef

config_files=hostapd-wpe.conf hostapd-wpe.eap_user
cert_config_files=ca.der ca.key ca.pem dh server.key server.pem 

define Install/hostapd-wpe
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_DIR) $(1)/usr/local/etc/hostapd-wpe
	$(INSTALL_DIR) $(1)/usr/local/etc/hostapd-wpe/certs
endef

define Package/hostapd-wpe/install
	$(call Install/hostapd-wpe,$(1))
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/hostapd/hostapd-wpe $(1)/usr/sbin/
	$(INSTALL_DATA) $(addprefix $(PKG_BUILD_DIR)/hostapd/,$(config_files)) $(1)/usr/local/etc/hostapd-wpe/
	$(INSTALL_DATA) $(addprefix $(PKG_BUILD_DIR)/hostapd/certs/,$(cert_config_files)) $(1)/usr/local/etc/hostapd-wpe/certs/
endef

$(eval $(call BuildPackage,hostapd-wpe))
