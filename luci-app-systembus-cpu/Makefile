include $(TOPDIR)/rules.mk

PKG_NAME:=luci-app-systembus-cpu
PKG_VERSION:=0.91
PKG_RELEASE:=$(AUTORELEASE)
PKG_MAINTAINER:=Oskari Rauta <oskari.rauta@gmail.com>

LUCI_TITLE:=Systembus cpu statistics
LUCI_DEPENDS:=+luci-mod-status +systembus
LUCI_PKGARCH:=all

include $(TOPDIR)/feeds/luci/luci.mk

define Package/luci-app-systembus-cpu/postinst
#!/bin/sh

TARGET_PATH="$${IPKG_INSTROOT}/usr/share/rpcd/acl.d"
FILENAME="luci-mod-status-index.json"
TMP_PATH=$$(mktemp -d -u)
TMP_FILE="$$TMP_PATH/$$FILENAME"

mkdir -p $${TMP_PATH}
rm -f $${TMP_FILE} $${TMP_FILE}.new
cp -f $${TARGET_PATH}/$${FILENAME} $${TMP_FILE}

[ -z "$$(grep -n -F $$'\t\"luci-mod-status-cpu\": {' $${TMP_FILE})" ] || {
	echo "already patched (luci-mod-status-cpu)"
	rm -rf $${TMP_PATH}
	exit 0
}

LINE_NO=$$(grep -n -F $$'\t"luci-mod-status-index-dhcp": {' $$TMP_FILE)
[ -z "$$LINE_NO" ] && {
	echo "patching failed: pattern not found"
	echo "searched for: luci-mod-status-index-dhcp"
	rm -rf $${TMP_PATH}
	exit 1
}

LINE_NO=$$(echo $$LINE_NO | head -n1 | cut -d ":" -f1)
[ $$LINE_NO -gt 0 ] || {
	echo "patching failed: invalid line number"
	echo "parsed result: \"$$LINE_NO\""
	rm -rf $${TMP_PATH}
	exit 1
}

REP="\t\\\"luci-mod-status-cpu\\\": {\n\t\t\\\"description\\\": \\\"Grant access to cpu load display\\\",\n\t\t\\\"read\\\": {\n\t\t\t\\\"ubus\\\": {\n\t\t\t\t\\\"system.cpu\\\": [ \\\"list\\\", \\\"temp\\\", \\\"temperature\\\" ]\n\t\t\t}\n\t\t}\n\t},\n"
PAT="{ if (NR == $${LINE_NO}) print \"$${REP}\"; print \$$0 }"

awk -e "$$PAT" $$TMP_FILE > $${TMP_FILE}.new

[ $$(wc -l $${TMP_FILE}.new | head -n1 | cut -d " " -f 1) -gt $$(($${LINE_NO} + 3)) ] || {
	echo "patching failed: patched output file malformed"
	rm -rf $${TMP_PATH}
	exit 1
}

rm -f $${TMP_FILE}
rm -f $${TARGET_PATH}/$${FILENAME}
mv $${TMP_FILE}.new $${TARGET_PATH}/$${FILENAME}
rm -rf $${TMP_PATH}

echo "patch succeeded"
exit 0
endef

# call BuildPackage - OpenWrt buildroot signature
