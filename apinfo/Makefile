include $(TOPDIR)/rules.mk

PKG_NAME:=apinfo
PKG_VERSION:=20260111
PKG_RELEASE:=1

include $(INCLUDE_DIR)/package.mk

define Package/apinfo
	SECTION:=admin
	CATEGORY:=Administration
	TITLE:=AP Controller
	DEPENDS:=+sshpass
	MAINTAINER:=Cezary Jackiewicz <cezary@eko.one.pl>
	PKGARCH:=all
endef

define Package/apinfo/description
  Simple AP management system
endef

define Package/apinfo/conffiles
/etc/config/apinfo
endef

define Build/Compile
endef

define Package/apinfo/postinst
#!/bin/sh
if [ -z "$${IPKG_INSTROOT}" ]; then
	[ -e /etc/uci-defaults/apinfo ] && /etc/uci-defaults/apinfo
	/etc/init.d/rpcd restart
	/etc/init.d/cron restart
	/usr/bin/apinfo
fi
exit 0
endef

define Package/apinfo/postrm
#!/bin/sh
if [ -z "$${IPKG_INSTROOT}" ]; then
	/etc/init.d/rpcd restart
	sed -i '/apinfo/d' /etc/crontabs/root
	/etc/init.d/cron restart
fi
exit 0
endef

define Package/apinfo/install
	$(INSTALL_DIR) $(1)
	$(CP) ./files/* $(1)
endef

$(eval $(call BuildPackage,apinfo))
