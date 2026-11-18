pkgname=hyprfm-bin
_appname=HyprFM
pkgver=0.7.1
pkgrel=1
pkgdesc="A lightweight Qt6/QML file manager for Hyprland (release AppImage, bundled Qt)"
arch=('x86_64')
url="https://github.com/soyeb-jim285/hyprfm"
license=('MIT')
# Host libs the AppImage leaves unbundled (readelf NEEDED minus usr/lib).
depends=('dbus' 'e2fsprogs' 'fontconfig' 'freetype2' 'gcc-libs' 'glib2' 'glibc' 'gvfs'
         'libglvnd' 'libx11' 'libxcb' 'libxkbcommon' 'wayland' 'xdg-utils' 'zlib')
makedepends=('7zip')
optdepends=(
    'wl-clipboard: clipboard support via wl-copy and wl-paste'
    'fd: fast recursive search (falls back to a built-in walker)'
    'bat: syntax-highlighted text previews'
    'md4c: rendered Markdown previews (via md2html)'
    'gvfs-smb: SMB/CIFS remote browsing support'
    'gvfs-mtp: Android phones (MTP) in the sidebar'
    'ffmpeg: video thumbnails and audio/video metadata (via ffprobe)'
    'poppler: PDF thumbnails, previews, and metadata (via pdftoppm/pdfinfo)'
    'perl-image-exiftool: EXIF metadata for images (via exiftool)'
    'udisks2: mount/unmount devices from sidebar'
)
provides=('hyprfm')
conflicts=('hyprfm' 'hyprfm-git')
options=('!strip' '!debug')
source=(
    "${_appname}-v${pkgver}-x86_64.AppImage::${url}/releases/download/v${pkgver}/${_appname}-v${pkgver}-x86_64.AppImage"
    "LICENSE-${pkgver}::https://raw.githubusercontent.com/soyeb-jim285/hyprfm/v${pkgver}/LICENSE"
)
sha256sums=('e27ad1aa12ba6e6c3b0e850081592dfdea895ec0beeef07e2f854a1b23e3e9df'
            '8f8b292b6ebbda3e2a3ec0fc0f7c1146916e2821e671bcb3c92cde2afab60fab')
noextract=("${_appname}-v${pkgver}-x86_64.AppImage")

prepare() {
    # Unpack the squashfs after the ELF runtime without running the AppImage.
    local img="${_appname}-v${pkgver}-x86_64.AppImage" shoff shentsize shnum
    shoff=$(od -An -t u8 -j 40 -N 8 "$img")
    shentsize=$(od -An -t u2 -j 58 -N 2 "$img")
    shnum=$(od -An -t u2 -j 60 -N 2 "$img")
    tail -c +$((shoff + shentsize * shnum + 1)) "$img" > appimage.sqfs
    rm -rf AppDir
    7z x -y -oAppDir appimage.sqfs >/dev/null
}

package() {
    install -d "${pkgdir}/opt/hyprfm"
    cp -a AppDir/. "${pkgdir}/opt/hyprfm/"

    # AppRun resolves its dir before following symlinks, so wrap it instead.
    install -Dm755 /dev/stdin "${pkgdir}/usr/bin/hyprfm" <<'EOF'
#!/bin/sh
exec /opt/hyprfm/AppRun "$@"
EOF
    # Portal binary finds libs (RUNPATH) and the picker via its real path.
    ln -s /opt/hyprfm/usr/bin/xdg-desktop-portal-hyprfm "${pkgdir}/usr/bin/xdg-desktop-portal-hyprfm"

    install -d "${pkgdir}/usr/share"
    cp -a AppDir/usr/share/{applications,icons,metainfo,dbus-1,systemd,xdg-desktop-portal} "${pkgdir}/usr/share/"
    # The binary has /usr/share/hyprfm (themes, QML fallback) compiled in.
    ln -s /opt/hyprfm/usr/share/hyprfm "${pkgdir}/usr/share/hyprfm"

    install -Dm644 "LICENSE-${pkgver}" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
