include $(TOPDIR)/rules.mk

PKG_NAME:=n2n-v3
PKG_VERSION:=3.1.1
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/ntop/n2n.git
PKG_SOURCE_VERSION:=dev
PKG_MIRROR_HASH:=skip

PKG_LICENSE:=GPL-3.0-only
PKG_LICENSE_FILES:=LICENSE
PKG_USE_MIPS16:=0
PKG_BUILD_PARALLEL:=1

include $(INCLUDE_DIR)/package.mk

define Package/n2n-v3/Default
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=VPN
  URL:=https://github.com/ntop/n2n
endef

define Package/n2n-edge
  $(call Package/n2n-v3/Default)
  TITLE:=n2n v3 VPN edge node
  DEPENDS:=+kmod-tun +libpthread +libopenssl
endef

define Package/n2n-edge/description
  n2n v3 is a layer-two peer-to-peer virtual private network (VPN).
  This package contains the edge node daemon.
endef

define Package/n2n-edge/conffiles
/etc/n2n/edge.conf
endef

define Package/n2n-supernode
  $(call Package/n2n-v3/Default)
  TITLE:=n2n v3 VPN supernode
  DEPENDS:=+libpthread +libopenssl
endef

define Package/n2n-supernode/description
  n2n v3 is a layer-two peer-to-peer virtual private network (VPN).
  This package contains the supernode daemon.
endef

define Package/n2n-supernode/conffiles
/etc/n2n/supernode.conf
endef

CONFIGURE_ARGS += \
	--with-openssl

# 禁用非核心可选依赖，避免 19.07 与 25.12 宿主机/feed 环境差异导致编译失败
# n2n v3 已内置 LZO 压缩与全套加密算法，若需 zstd 可改为 DEPENDS 增加 +libzstd 并移除下面对应项
CONFIGURE_VARS += \
	ac_cv_lib_zstd_ZSTD_compress=no \
	ac_cv_lib_cap_cap_get_proc=no

define Build/Configure
	(cd $(PKG_BUILD_DIR) && autoheader && autoconf)
	$(call Build/Configure/Default)
endef

define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR) edge supernode
endef

define Package/n2n-edge/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/edge $(1)/usr/sbin/edge
	$(INSTALL_DIR) $(1)/etc/n2n
	$(INSTALL_CONF) ./files/edge.conf $(1)/etc/n2n/edge.conf
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/n2n-edge.init $(1)/etc/init.d/n2n-edge
endef

define Package/n2n-supernode/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/supernode $(1)/usr/sbin/supernode
	$(INSTALL_DIR) $(1)/etc/n2n
	$(INSTALL_CONF) ./files/supernode.conf $(1)/etc/n2n/supernode.conf
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/n2n-supernode.init $(1)/etc/init.d/n2n-supernode
endef

$(eval $(call BuildPackage,n2n-edge))
$(eval $(call BuildPackage,n2n-supernode))