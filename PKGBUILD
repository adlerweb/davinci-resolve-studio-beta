# Maintainer: Alex S. <shantanna_at_hotmail_dot_com>
# Contributor: Jonathon Fernyhough <jonathon_at_manjaro_dot_org>

# You'll need to download the package archive from
# https://www.blackmagicdesign.com/support/

# Hardware support is limited. Nvidia cards should work fine.
# If you're running a hybrid setup, try with primusrun/optirun.

pkgname=davinci-resolve-studio-beta
_pkgname=resolve
resolve_app_name=com.blackmagicdesign.resolve
pkgver=16.0b1
pkgrel=2
arch=('any')
url="https://www.blackmagicdesign.com/support/family/davinci-resolve-and-fusion"
license=('Commercial')
depends=('glu' 'gtk2' 'gstreamer' 'libpng12' 'lib32-libpng12' 'ocl-icd' 'openssl-1.0'
         'opencl-driver' 'qt4' 'qt5-base' 'qt5-svg' 'qt5-webkit' 'qt5-webengine' 'qt5-websockets')
makedepends=('libarchive' 'xdg-user-dirs')
options=('!strip')
provides=('davinci-resolve')
install=davinci-resolve.install

if [ ${pkgname} == "davinci-resolve-studio-beta" ]; then
# Variables for STUDIO edition
	pkgdesc='Professional A/V post-production software suite from Blackmagic Design. Studio edition, requires license key or license dongle.'
	_archive_name=DaVinci_Resolve_Studio_${pkgver}_Linux
	sha256sums=('29c9a2d5fb94168e4bcbeee6b7c7f192ab471f3e04bf4c75631932d6b3907165')
	conflicts=('davinci-resolve-beta' 'davinci-resolve' 'davinci-resolve-studio')
	
else
# Variables for FREE edition
	pkgdesc='Professional A/V post-production software suite from Blackmagic Design'
	_archive_name=DaVinci_Resolve_${pkgver}_Linux
	sha256sums=('7f4517cc08dde962e515ec1bdfbc65b0f1e55368f8d82d31987590e62a6be9c0')
	conflicts=('davinci-resolve' 'davinci-resolve-studio' 'davinci-resolve-studio-beta')
	
fi


_archive=${_archive_name}.zip
_installer_binary=${_archive_name}.run

# Trying to make the user's life easier ;o)
msg2 "Trying to fetch the archive file if available..."
DOWNLOADS_DIR=`xdg-user-dir DOWNLOAD`

if [ ! -f ${PWD}/${_archive} ]; then
	if [ -f $DOWNLOADS_DIR/${_archive} ]; then
		ln -sfn $DOWNLOADS_DIR/${_archive} ${PWD}
	else
		msg2 ""
		msg2 "The package can be downloaded here: https://www.blackmagicdesign.com/support/family/davinci-resolve-and-fusion"
		msg2 "Please remember to put a downloaded package ${_archive}into the build directory ${PWD} or $DOWNLOADS_DIR"
		msg2 ""
	fi
fi

source=("local://${_archive}")


prepare()
{
	# Remove udev rules (The official installer does not remove these files. This leads to the conflict "exists in the file system".)
	confiles=$(find /usr/lib/udev/rules.d -name 75-davincipanel.rules -o -name 75-sdx.rules 2> /dev/null | awk -F/ '{print $NF}'
               find /opt/resolve/configs -name log-conf.xml -o -name config.dat 2> /dev/null | awk -F/ '{print $NF}'
               )
	if [ "${confiles}" ]; then
		msg2 "The file(s) $(echo ${confiles} | xargs | sed 's/ /, /g') already exist in your filesystem."
		msg2 "This can lead to a conflict and the installation will fail."
		msg2 "Please restart the installation with the --overwrite option."
	fi
}

package()
{
	msg2 "Creating missing folders..."
	mkdir -p -m 0775 "${pkgdir}/opt/${_pkgname}/"{configs,DolbyVision,easyDCP,Fairlight,GPUCache,logs,Media,"Resolve Disk Database",.crashreport,.license,.LUT}
	mkdir -p "${pkgdir}/usr/share/"{applications,desktop-directories,icons/hicolor,mime/packages}
#	mkdir -p "${pkgdir}/tmp/${_pkgname}/"{logs,GPUCache}
	mkdir -p "${pkgdir}/usr/lib/udev/rules.d"
	mkdir -p "${pkgdir}/etc/xdg/menus"

	msg2 "Extracting from bundle..."
	msg "Please wait, this take a while..."
	cd "${srcdir}" || exit
	bsdtar x -f ${_installer_binary} -C "${pkgdir}/opt/${_pkgname}"

	msg2 "Add lib symlinks..."
	cd "${pkgdir}/opt/${_pkgname}/" || exit
	ln -s /usr/lib/libcrypto.so.1.0.0 libs/libcrypto.so.10
	ln -s /usr/lib/libssl.so.1.0.0 libs/libssl.so.10

	msg2 "Install launchers and configs..."
	cd "${pkgdir}/opt/${_pkgname}/" || exit
	install -Dm666 share/default-config.dat "${pkgdir}/opt/${_pkgname}/configs/config.dat"
	install -Dm666 share/log-conf.xml "${pkgdir}/opt/${_pkgname}/configs/log-conf.xml"
	install -Dm666 share/default_cm_config.bin "${pkgdir}/opt/${_pkgname}/DolbyVision/config.bin"
	install -Dm644 share/DaVinciResolve.desktop "${pkgdir}/usr/share/applications/${resolve_app_name}.desktop"
	install -Dm644 share/DaVinciResolvePanelSetup.desktop "${pkgdir}/usr/share/applications/${resolve_app_name}-Panels.desktop"
	install -Dm644 share/DaVinciResolveInstaller.desktop "${pkgdir}/usr/share/applications/${resolve_app_name}-Installer.desktop"
	install -Dm644 share/DaVinciResolveCaptureLogs.desktop "${pkgdir}/usr/share/applications/${resolve_app_name}-CaptureLogs.desktop"
	install -Dm644 share/DaVinciResolve.directory "${pkgdir}/usr/share/desktop-directories/${resolve_app_name}.directory"
	install -Dm644 share/DaVinciResolve.menu "${pkgdir}/etc/xdg/menus/${resolve_app_name}.menu"

	for _file in $(find ${pkgdir}/usr/share ${pkgdir}/etc -type f -name *.desktop -o -name *.directory -o -name *.menu | xargs)
	do
		sed -i "s|RESOLVE_INSTALL_LOCATION|/opt/${_pkgname}|g" $_file
	done

	# This will help adding the app to favorites and prevent glitches on many desktops.
	echo "StartupWMClass=resolve" >> "${pkgdir}/usr/share/applications/${resolve_app_name}.desktop"

	msg2 "Creating and installing udev rules..."
	echo 'SUBSYSTEM=="usb", ENV{DEVTYPE}=="usb_device", ATTRS{idVendor}=="096e", MODE="0666"' > "${pkgdir}/usr/lib/udev/rules.d/75-davincipanel.rules"
	echo 'SUBSYSTEM=="usb", ENV{DEVTYPE}=="usb_device", ATTRS{idVendor}=="1edb", MODE="0666"' > "${pkgdir}/usr/lib/udev/rules.d/75-sdx.rules"
	chmod 644 "${pkgdir}/usr/lib/udev/rules.d/"{75-davincipanel.rules,75-sdx.rules}

#	Not sure we need it
#	msg2 "Any final tweaks..."
#	ln -s "/tmp/${_pkgname}/logs" "${pkgdir}/opt/${_pkgname}/logs"
#	ln -s "/tmp/${_pkgname}/GPUCache" "${pkgdir}/opt/${_pkgname}/GPUCache"

	msg2 "Installing Application icons..."
	# Obviously not working without root rights.
#	XDG_DATA_DIRS="${pkgdir}/usr/share/icons/hicolor" xdg-icon-resource install --size 64 "${pkgdir}/opt/${_pkgname}/graphics/DV_Resolve.png" DaVinci-Resolve 2>&1 >> /dev/null
#	XDG_DATA_DIRS="${pkgdir}/usr/share/icons/hicolor" xdg-icon-resource install --size 64 "${pkgdir}/opt/${_pkgname}/graphics/DV_ResolveProj.png" DaVinci-ResolveProj 2>&1 >> /dev/null
#	XDG_DATA_DIRS="${pkgdir}/usr/share/icons/hicolor" xdg-icon-resource install --size 64 --context mimetypes "${pkgdir}/opt/${_pkgname}/graphics/DV_ResolveProj.png" application-x-resolveproj 2>&1  >> /dev/null
#	XDG_DATA_DIRS="${pkgdir}/usr/share/mime/packages" xdg-mime install --novendor "${pkgdir}/opt/${_pkgname}/share/resolve.xml" 2>&1  >> /dev/null
	install -D -m644 graphics/DV_Resolve.png "${pkgdir}/usr/share/icons/hicolor/64x64/apps/DV_Resolve.png"
	install -D -m644 graphics/DV_ResolveProj.png "${pkgdir}/usr/share/icons/hicolor/64x64/apps/DV_ResolveProj.png"

	install -D -m644 share/resolve.xml "${pkgdir}/usr/share/mime/packages/resolve.xml"
	
	msg2 "Setting the right permissions..."

	if [ ! "$(logname 2>&1 >/dev/null)" ]; then
		_user=$(logname)
		_group=$(id -g -n ${_user})
	else
		_user=root
		_group=root
	fi

	chown -R ${_user}:${_group} "${pkgdir}/opt/${_pkgname}/"{*,.*}
	chown -R ${_user}:root "${pkgdir}/opt/${_pkgname}/"{configs,DolbyVision,easyDCP,Fairlight,logs,Media,'Resolve Disk Database',.crashreport,.license,.LUT}

	msg2 "Done!"
}

# vim: fileencoding=utf-8 sts=4 sw=4 noet
