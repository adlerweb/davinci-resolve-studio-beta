# Maintainer: Alex S. <shantanna_at_hotmail_dot_com>
# Contributor: Jonathon Fernyhough <jonathon_at_manjaro_dot_org>
# Contributor: Florian Knodt <aur_at_adlerweb_dot_info>

# Hardware support is limited. Nvidia cards should work fine.
# If you're running a hybrid setup, try with primusrun/optirun.

# This was originally written by Daniel Bermond in blackmagic-decklink-sdk pkgbuild
# It is sufficient to just replace _downloadid and possibly _referid to correspond
# new release version. It can be obtained from Developer Tools -> Network.
# Look for an URL like https://www.blackmagicdesign.com/api/register/de/download/XXX
# where XXX is _downloadid and Referer containing _referid
_downloadid='9248e8caea9c434fa519bdb3c9ac4132'
_referid='d33ad0df1c04430bbeda60fe3eb6f897'
_siteurl="https://www.blackmagicdesign.com/api/register/us/download/${_downloadid}"

_useragent="User-Agent: Mozilla/5.0 (X11; Linux ${CARCH}) \
                        AppleWebKit/537.36 (KHTML, like Gecko) \
                        Chrome/77.0.3865.75 \
                        Safari/537.36"

_reqjson="{\"platform\":\"Linux\",\"policy\":true,\"product\":{\"name\":\"DaVinci Resolve Studio\"},\"country\":\"us\",\"downloadOnly\":true,\"origin\":\"www.blackmagicdesign.com\"}"

_reqjson="$(  printf '%s' "$_reqjson"   | sed 's/[[:space:]]\+/ /g')"
_useragent="$(printf '%s' "$_useragent" | sed 's/[[:space:]]\+/ /g')"
_useragent_escaped="${_useragent// /\\ }"

_srcurl="$(curl \
            -s \
            -H 'Host: www.blackmagicdesign.com' \
            -H 'Accept: application/json, text/plain, */*' \
            -H 'Origin: https://www.blackmagicdesign.com' \
            -H "$_useragent" \
            -H 'Content-Type: application/json;charset=UTF-8' \
            -H "Referer: https://www.blackmagicdesign.com/support/download/${_referid}/Linux" \
            -H 'Accept-Encoding: gzip, deflate, br' \
            -H 'Accept-Language: en-US,en;q=0.9' \
            -H 'Authority: www.blackmagicdesign.com' \
            -H 'Cookie: _ga=GA1.2.1849503966.1518103294; _gid=GA1.2.953840595.1518103294' \
            --data-ascii "$_reqjson" \
            --compressed \
            "$_siteurl")"

DLAGENTS=("https::/usr/bin/curl \
            -gqb '' -C - --retry 3 --retry-delay 3 \
            -H Host:\ swr.cloud.blackmagicdesign.com \
            -H Upgrade-Insecure-Requests:\ 1 \
            -H ${_useragent_escaped} \
            -H Accept:\ text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8 \
            -H Accept-Language:\ en-US,en;q=0.9 \
            -o %o \
            --compressed \
            %u")

pkgname=davinci-resolve-studio-beta
_pkgname=resolve
resolve_app_name=com.blackmagicdesign.resolve
pkgver=20.0b1
pkgrel=1
arch=('x86_64')
url="https://www.blackmagicdesign.com/support/family/davinci-resolve-and-fusion"
license=('Commercial')
depends=('glu' 'gtk2' 'libpng12' 'fuse2' 'opencl-driver' 'qt5-x11extras' 'qt5-svg' 'qt5-webengine'
         'qt5-websockets' 'qt5-quickcontrols2' 'qt5-multimedia' 'libxcrypt-compat' 'xmlsec'
         'java-runtime' 'ffmpeg4.4' 'gst-plugins-bad-libs' 'python-numpy' 
         'tbb' 'apr-util' 'luajit' 'libc++' 'libc++abi')
makedepends=('libarchive' 'xdg-user-dirs' 'patchelf')
options=('!strip')

# Variables for STUDIO edition
pkgdesc='Professional A/V post-production software suite from Blackmagic Design. Studio edition, requires license key or license dongle.'
sha256sums=('ced2f69025e2145a28ba16d714b27553ac22b3a59de787ab36c5dec13b3f4ffb')
conflicts=('davinci-resolve-beta' 'davinci-resolve' 'davinci-resolve-studio')
_archive_name=DaVinci_Resolve_Studio_${pkgver}_Linux
_archive=${_archive_name}.zip
source=("${_archive}"::"$_srcurl")

prepare() {
  chmod u+x "./DaVinci_Resolve_Studio_${pkgver}_Linux.run"
  "./DaVinci_Resolve_Studio_${pkgver}_Linux.run" --appimage-extract

  # Fix permission to all files and dirs
  chmod -R u+rwX,go+rX,go-w "squashfs-root"

  pushd "squashfs-root/share/panels"
  tar -zxf dvpanel-framework-linux-x86_64.tgz
  chmod -R u+rwX,go+rX,go-w "lib"
  mv *.so "${srcdir}/squashfs-root/libs"
  mv lib/* "${srcdir}/squashfs-root/libs"
  popd

  rm -rf squashfs-root/installer squashfs-root/installer* squashfs-root/AppRun squashfs-root/AppRun*

  while IFS= read -r -d '' _file; do
    chmod 0755 "${_file}"
  done < <(find "squashfs-root" -type d -print0)

  while IFS= read -r -d '' _file; do
    [[ -f "${_file}" && $(od -t x1 -N 4 "${_file}") == *"7f 45 4c 46"* ]] || continue
    chmod 0755 "${_file}"
  done < <(find "squashfs-root" -type f -print0)

  # Prepare list of paths for patchelf
  _patchelf_paths=("libs"
                   "libs/plugins/sqldrivers"
                   "libs/plugins/xcbglintegrations"
                   "libs/plugins/imageformats"
                   "libs/plugins/platforms"
                   "libs/Fusion"
                   "plugins"
                   "bin"
                   "BlackmagicRAWSpeedTest/BlackmagicRawAPI"
                   "BlackmagicRAWSpeedTest/plugins/platforms"
                   "BlackmagicRAWSpeedTest/plugins/imageformats"
                   "BlackmagicRAWSpeedTest/plugins/mediaservice"
                   "BlackmagicRAWSpeedTest/plugins/audio"
                   "BlackmagicRAWSpeedTest/plugins/xcbglintegrations"
                   "BlackmagicRAWSpeedTest/plugins/bearer"
                   "BlackmagicRAWPlayer/BlackmagicRawAPI"
                   "BlackmagicRAWPlayer/plugins/mediaservice"
                   "BlackmagicRAWPlayer/plugins/imageformats"
                   "BlackmagicRAWPlayer/plugins/audio"
                   "BlackmagicRAWPlayer/plugins/platforms"
                   "BlackmagicRAWPlayer/plugins/xcbglintegrations"
                   "BlackmagicRAWPlayer/plugins/bearer"
                   "Onboarding/plugins/xcbglintegrations"
                   "Onboarding/plugins/qtwebengine"
                   "Onboarding/plugins/platforms"
                   "Onboarding/plugins/imageformats"
                   "DaVinci Control Panels Setup/plugins/platforms"
                   "DaVinci Control Panels Setup/plugins/imageformats"
                   "DaVinci Control Panels Setup/plugins/bearer"
                   "DaVinci Control Panels Setup/AdminUtility/PlugIns/DaVinciKeyboards"
                   "DaVinci Control Panels Setup/AdminUtility/PlugIns/DaVinciPanels")
  for _index in "${!_patchelf_paths[@]}"
  do
    _patchelf_paths[${_index}]="/opt/${_pkgname}/${_patchelf_paths[${_index}]}"
  done
  while IFS= read -r -d '' _file; do
    [[ -f "${_file}" && $(od -t x1 -N 4 "${_file}") == *"7f 45 4c 46"* ]] || continue
    patchelf --set-rpath "$(IFS=":"; echo "${_patchelf_paths[*]}:\$ORIGIN")" "${_file}"
  done < <(find "squashfs-root" -type f -size -32M -print0)

  while IFS= read -r -d '' _file; do
    sed -i "s|RESOLVE_INSTALL_LOCATION|/opt/${_pkgname}|g" "${_file}"
  done < <(find . -type f '(' -name "*.desktop" -o -name "*.directory" -o -name "*.directory" -o -name "*.menu" ')' -print0)

  rm "squashfs-root/libs/libc++.so.1" "squashfs-root/libs/libglib-2.0.so.0" "squashfs-root/libs/libgio-2.0.so.0" "squashfs-root/libs/libgmodule-2.0.so.0"
  ln -s "../BlackmagicRAWPlayer/BlackmagicRawAPI" "squashfs-root/bin/"
  ln -s /usr/lib/libc++.so.1.0 "squashfs-root/libs/libc++.so.1"
  ln -s /usr/lib/libglib-2.0.so.0 "squashfs-root/libs/libglib-2.0.so.0"
  ln -s /usr/lib/libgio-2.0.so.0 "squashfs-root/libs/libgio-2.0.so.0"
  ln -s /usr/lib/libgmodule-2.0.so.0 "squashfs-root/libs/libgmodule-2.0.so.0"
  ln -s /usr/lib/libgdk_pixbuf-2.0.so.0 "squashfs-root/libs/libgdk_pixbuf-2.0.so.0"

  echo "StartupWMClass=resolve" >> "squashfs-root/share/DaVinciResolve.desktop"

  echo 'SUBSYSTEM=="usb", ENV{DEVTYPE}=="usb_device", ATTRS{idVendor}=="096e", MODE="0666"' > "squashfs-root/share/etc/udev/rules.d/99-DavinciPanel.rules"
}

package() {
  install -d -m 0755 "${pkgdir}/opt/${_pkgname}"
  # Install the squashfs-root
  cp -rf squashfs-root/* "${pkgdir}/opt/${_pkgname}"

  # Distribute files into other directories
  pushd "${pkgdir}/opt/${_pkgname}"
  install -D -m 0644 -t "${pkgdir}/opt/${_pkgname}/configs" "share/default-config.dat" "share/log-conf.xml"
  install -D -m 0644 -t "${pkgdir}/opt/${_pkgname}/DolbyVision" "share/default_cm_config.bin"
  install -d -m 0755 "${pkgdir}/opt/${_pkgname}/.license"
  install -D -m 0644 -t "${pkgdir}/usr/share/applications" "share/DaVinciResolve.desktop" "share/DaVinciControlPanelsSetup.desktop" "share/DaVinciResolveInstaller.desktop" \
    "share/DaVinciResolveCaptureLogs.desktop" "share/blackmagicraw-player.desktop" "share/blackmagicraw-speedtest.desktop"
  install -D -m 0644 -t "${pkgdir}/usr/share/desktop-directories" "share/DaVinciResolve.directory"
  install -D -m 0644 -t "${pkgdir}/etc/xdg/menus" "share/DaVinciResolve.menu"
  install -D -m 0644 -t "${pkgdir}/usr/share/icons/hicolor/64x64/apps" "graphics/DV_Resolve.png" "graphics/DV_ResolveProj.png"
  install -D -m 0644 -t "${pkgdir}/usr/share/mime/packages" "share/resolve.xml"
  install -D -m 0644 -t "${pkgdir}/usr/lib/udev/rules.d" "share/etc/udev/rules.d"/{99-BlackmagicDevices.rules,99-ResolveKeyboardHID.rules,99-DavinciPanel.rules}
  popd
}

# vim: fileencoding=utf-8 sts=4 sw=4 noet
