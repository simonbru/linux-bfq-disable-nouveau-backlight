# Maintainer: Piotr Gorski <lucjan.lucjanov@gmail.com>
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

### BUILD OPTIONS
# Set these variables to ANYTHING that is not null to enable them

# Tweak kernel options prior to a build via nconfig
_makenconfig=

# Tweak kernel options prior to a build via menuconfig
_makemenuconfig=

# Tweak kernel options prior to a build via xconfig
_makexconfig=

# Tweak kernel options prior to a build via gconfig
_makegconfig=

# Running with a 1000 HZ tick rate 
_1k_HZ_ticks=

# NUMA is optimized for multi-socket motherboards.
# A single multi-core CPU actually runs slower with NUMA enabled.
# See, https://bugs.archlinux.org/task/31187
_NUMAdisable=y

# Compile ONLY probed modules
# As of mainline 2.6.32, running with this option will only build the modules
# that you currently have probed in your system VASTLY reducing the number of
# modules built and the build time to do it.
#
# WARNING - ALL modules must be probed BEFORE you begin making the pkg!
#
# To keep track of which modules are needed for your specific system/hardware,
# give module_db script a try: https://aur.archlinux.org/packages/modprobed-db
# This PKGBUILD will call it directly to probe all the modules you have logged!
#
# More at this wiki page ---> https://wiki.archlinux.org/index.php/Modprobed_db
_localmodcfg=

# Use the current kernel's .config file
# Enabling this option will use the .config of the RUNNING kernel rather than
# the ARCH defaults. Useful when the package gets updated and you already went
# through the trouble of customizing your config options.  NOT recommended when
# a new kernel is released, but again, convenient for package bumps.
_use_current=

### Do not edit below this line unless you know what you're doing

pkgbase=linux-bfq
# pkgname=('linux-bfq' 'linux-bfq-headers' 'linux-bfq-docs')
_srcname=linux-4.8
pkgver=4.8.10
pkgrel=2
arch=('i686' 'x86_64')
url="http://algo.ing.unimo.it"
license=('GPL2')
options=('!strip')
makedepends=('kmod' 'inetutils' 'bc')
_bfqrel=v7r11
_bfqver=v8r4
_bfqpath="http://algo.ing.unimo.it/people/paolo/disk_sched/patches/4.8.0-${_bfqver}"
#_bfqpath="https://pf.natalenko.name/mirrors/bfq/4.8.0-${_bfqver}"
_gcc_patch="enable_additional_cpu_optimizations_for_gcc_v4.9+_kernel_v3.15+.patch"

source=("http://www.kernel.org/pub/linux/kernel/v4.x/${_srcname}.tar.xz"
        "https://www.kernel.org/pub/linux/kernel/v4.x/${_srcname}.tar.sign"
        "http://www.kernel.org/pub/linux/kernel/v4.x/patch-${pkgver}.xz"
        "https://www.kernel.org/pub/linux/kernel/v4.x/patch-${pkgver}.sign"
        "${_bfqpath}/0001-block-cgroups-kconfig-build-bits-for-BFQ-${_bfqrel}-4.8.0.patch"
        "${_bfqpath}/0002-block-introduce-the-BFQ-${_bfqrel}-I-O-sched-to-be-ported.patch"
        "${_bfqpath}/0003-block-bfq-add-Early-Queue-Merge-EQM-to-BFQ-${_bfqrel}-to-.patch"
        "${_bfqpath}/0004-Turn-BFQ-${_bfqrel}-into-BFQ-${_bfqver}-for-4.8.0.patch"
        "http://repo-ck.com/source/gcc_patch/${_gcc_patch}.gz"
        'change-default-console-loglevel.patch'
         # the main kernel config files
        'config' 'config.x86_64'
         # pacman hook for initramfs regeneration
        '99-linux.hook'
         # standard config files for mkinitcpio ramdisk
        'linux.preset'
        # patches from https://github.com/linusw/linux-bfq/commits/bfq-v8
        '0005-BFQ-Fix.patch'
        '0006-BFQ-Fix.patch')

_kernelname=${pkgbase#linux} 

prepare() {
    cd "${srcdir}/${_srcname}"

    ### Add upstream patch
        msg "Add upstream patch"
        patch -Np1 -i "${srcdir}/patch-${pkgver}" 
         
    ### set DEFAULT_CONSOLE_LOGLEVEL to 4 (same value as the 'quiet' kernel param)
    # remove this when a Kconfig knob is made available by upstream
    # (relevant patch sent upstream: https://lkml.org/lkml/2011/7/26/227)
        msg "Patching set DEFAULT_CONSOLE_LOGLEVEL to 4"
        patch -p1 -i "${srcdir}/change-default-console-loglevel.patch"

    ### Patch source with BFQ
        msg "Patching source with BFQ patches"
        for p in "${srcdir}"/000{1,2,3,4,5,6}-*BFQ*.patch; do
        msg " $p"
        patch -Np1 -i "$p"
        done

    ### Patch source to enable more gcc CPU optimizatons via the make nconfig
        msg "Patching source with gcc patch to enable more cpus types"
	patch -Np1 -i "${srcdir}/${_gcc_patch}"

	
    ### Clean tree and copy ARCH config over
	msg "Running make mrproper to clean source tree"
	make mrproper

	if [ "${CARCH}" = "x86_64" ]; then
		cat "${srcdir}/config.x86_64" > ./.config
	else
		cat "${srcdir}/config" > ./.config
	fi
      
    ### Optionally set tickrate to 1000 
	if [ -n "$_1k_HZ_ticks" ]; then
		msg "Setting tick rate to 1k..."
		sed -i -e 's/^CONFIG_HZ_300=y/# CONFIG_HZ_300 is not set/' \
			-i -e 's/^# CONFIG_HZ_1000 is not set/CONFIG_HZ_1000=y/' \
			-i -e 's/^CONFIG_HZ=300/CONFIG_HZ=1000/' .config
	fi

	### Optionally use running kernel's config
	# code originally by nous; http://aur.archlinux.org/packages.php?ID=40191
	if [ -n "$_use_current" ]; then
		if [[ -s /proc/config.gz ]]; then
			msg "Extracting config from /proc/config.gz..."
			# modprobe configs
			zcat /proc/config.gz > ./.config
		else
			warning "You kernel was not compiled with IKCONFIG_PROC!"
			warning "You cannot read the current config!"
			warning "Aborting!"
			exit
		fi
	fi

	if [ "${_kernelname}" != "" ]; then
		sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"${_kernelname}\"|g" ./.config
		sed -i "s|CONFIG_LOCALVERSION_AUTO=.*|CONFIG_LOCALVERSION_AUTO=n|" ./.config
	fi

	### Optionally disable NUMA since >99% of users have mono-socket systems.
	# For more, see: https://bugs.archlinux.org/task/31187
	if [ -n "$_NUMAdisable" ]; then
		if [ "${CARCH}" = "x86_64" ]; then
			msg "Disabling NUMA from kernel config..."
			sed -i -e 's/CONFIG_NUMA=y/# CONFIG_NUMA is not set/' \
				-i -e '/CONFIG_AMD_NUMA=y/d' \
				-i -e '/CONFIG_X86_64_ACPI_NUMA=y/d' \
				-i -e '/CONFIG_NODES_SPAN_OTHER_NODES=y/d' \
				-i -e '/# CONFIG_NUMA_EMU is not set/d' \
				-i -e '/CONFIG_NODES_SHIFT=6/d' \
				-i -e '/CONFIG_NEED_MULTIPLE_NODES=y/d' \
				-i -e '/# CONFIG_MOVABLE_NODE is not set/d' \
				-i -e '/CONFIG_USE_PERCPU_NUMA_NODE_ID=y/d' \
				-i -e '/CONFIG_ACPI_NUMA=y/d' ./.config
		fi
	fi

	# set extraversion to pkgrel
	sed -ri "s|^(EXTRAVERSION =).*|\1 -${pkgrel}|" Makefile

	# don't run depmod on 'make install'. We'll do this ourselves in packaging
	sed -i '2iexit 0' scripts/depmod.sh

	# get kernel version
	msg "Running make prepare for you to enable patched options of your choosing"
	make prepare

	### Optionally load needed modules for the make localmodconfig
	# See https://aur.archlinux.org/packages/modprobed-db
		if [ -n "$_localmodcfg" ]; then
		msg "If you have modprobe-db installed, running it in recall mode now"
		if [ -e /usr/bin/modprobed-db ]; then
			[[ ! -x /usr/bin/sudo ]] && echo "Cannot call modprobe with sudo.  Install via pacman -S sudo and configure to work with this user." && exit 1
			sudo /usr/bin/modprobed-db recall
		fi
		msg "Running Steven Rostedt's make localmodconfig now"
		make localmodconfig
	fi

	if [ -n "$_makenconfig" ]; then
		msg "Running make nconfig"
		make nconfig
	fi
	
	if [ -n "$_makemenuconfig" ]; then
		msg "Running make menuconfig"
		make menuconfig
	fi
	
	if [ -n "$_makexconfig" ]; then
		msg "Running make xconfig"
		make xconfig
	fi
	
	if [ -n "$_makegconfig" ]; then
		msg "Running make gconfig"
		make gconfig
	fi

	# rewrite configuration
	yes "" | make config >/dev/null

	# save configuration for later reuse
	if [ "${CARCH}" = "x86_64" ]; then
		cat .config > "${startdir}/config.x86_64.last"
	else
		cat .config > "${startdir}/config.last"
	fi
}

build() {
  cd "${srcdir}/${_srcname}"

  make ${MAKEFLAGS} LOCALVERSION= bzImage modules
}

_package() {
    pkgdesc='Linux Kernel and modules with the  BFQ scheduler.'
    depends=('coreutils' 'linux-firmware' 'mkinitcpio>=0.7')
    optdepends=('crda: to set the correct wireless channels of your country' 'nvidia-bfq: nVidia drivers for linux-bfq' 'modprobed-db: Keeps track of EVERY kernel module that has ever been probed - useful for those of us who make localmodconfig')
    backup=("etc/mkinitcpio.d/${pkgbase}.preset")
    install=linux.install

    cd "${srcdir}/${_srcname}"

    KARCH=x86

   # get kernel version
   _kernver="$(make LOCALVERSION= kernelrelease)"
   _basekernel=${_kernver%%-*}
   _basekernel=${_basekernel%.*}

    mkdir -p "${pkgdir}"/{lib/modules,lib/firmware,boot}
    make LOCALVERSION= INSTALL_MOD_PATH="${pkgdir}" modules_install
    cp arch/$KARCH/boot/bzImage "${pkgdir}/boot/vmlinuz-${pkgbase}"

    # set correct depmod command for install
    sed -e "s|%PKGBASE%|${pkgbase}|g;s|%KERNVER%|${_kernver}|g" \
    "${startdir}/${install}" > "${startdir}/${install}.pkg"
    true && install=${install}.pkg

    # install mkinitcpio preset file for kernel
    sed "s|%PKGBASE%|${pkgbase}|g" "${srcdir}/linux.preset" |
    install -D -m644 /dev/stdin "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

    # install pacman hook for initramfs regeneration
    sed "s|%PKGBASE%|${pkgbase}|g" "${srcdir}/99-linux.hook" |
    install -D -m644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/99-${pkgbase}.hook"

    # remove build and source links
    rm -f "${pkgdir}"/lib/modules/${_kernver}/{source,build}
    # remove the firmware
    rm -rf "${pkgdir}/lib/firmware"
    # make room for external modules
    ln -s "../extramodules-${_basekernel}${_kernelname:--ARCH}" "${pkgdir}/lib/modules/${_kernver}/extramodules"
    # add real version for building modules and running depmod from post_install/upgrade
    mkdir -p "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--ARCH}"
    echo "${_kernver}" > "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--ARCH}/version"


    # Now we call depmod...
    depmod -b "${pkgdir}" -F System.map "${_kernver}"

    # move module tree /lib -> /usr/lib
    mkdir -p "${pkgdir}/usr"
    mv "${pkgdir}/lib" "${pkgdir}/usr/"

    # add vmlinux
    install -D -m644 vmlinux "${pkgdir}/usr/lib/modules/${_kernver}/build/vmlinux"
}

_package-headers() {
    pkgdesc='Header files and scripts to build modules for linux-bfq.'
    depends=('linux-bfq')

    install -dm755 "${pkgdir}/usr/lib/modules/${_kernver}"

    cd "${srcdir}/${_srcname}"
    
  
        install -D -m644 Makefile \
		"${pkgdir}/usr/lib/modules/${_kernver}/build/Makefile"
	install -D -m644 kernel/Makefile \
		"${pkgdir}/usr/lib/modules/${_kernver}/build/kernel/Makefile"
	install -D -m644 .config \
		"${pkgdir}/usr/lib/modules/${_kernver}/build/.config"

	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include"

	for i in acpi asm-generic config crypto drm generated keys linux math-emu \
		media net pcmcia scsi soc sound trace uapi video xen; do
	cp -a include/${i} "${pkgdir}/usr/lib/modules/${_kernver}/build/include/"
	done

	# copy arch includes for external modules
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/x86"
	cp -a arch/x86/include "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/x86/"

	# copy files necessary for later builds, like nvidia and vmware
	cp Module.symvers "${pkgdir}/usr/lib/modules/${_kernver}/build"
	cp -a scripts "${pkgdir}/usr/lib/modules/${_kernver}/build"

	# fix permissions on scripts dir
	chmod og-w -R "${pkgdir}/usr/lib/modules/${_kernver}/build/scripts"
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/.tmp_versions"

	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/kernel"

	cp arch/${KARCH}/Makefile "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/"

	if [ "${CARCH}" = "i686" ]; then
		cp arch/${KARCH}/Makefile_32.cpu "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/"
	fi

	cp arch/${KARCH}/kernel/asm-offsets.s "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/kernel/"

	# add docbook makefile
	install -D -m644 Documentation/DocBook/Makefile \
	"${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/DocBook/Makefile"

	# add dm headers
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/md"
	cp drivers/md/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/md"

	# add inotify.h
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include/linux"
	cp include/linux/inotify.h "${pkgdir}/usr/lib/modules/${_kernver}/build/include/linux/"

	# add wireless headers
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/net/mac80211/"
	cp net/mac80211/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/net/mac80211/"

	# add dvb headers for external modules
	# in reference to:
	# http://bugs.archlinux.org/task/9912
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-core"
	cp drivers/media/dvb-core/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-core/"
	# and...
	# http://bugs.archlinux.org/task/11194
	###
	### DO NOT MERGE OUT THIS IF STATEMENT
	### IT AFFECTS USERS WHO STRIP OUT THE DVB STUFF SO THE OFFICIAL ARCH CODE HAS A CP
	### LINE THAT CAUSES MAKEPKG TO END IN AN ERROR
	###
	if [ -d include/config/dvb/ ]; then
		mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include/config/dvb/"
		cp include/config/dvb/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/include/config/dvb/"
	fi

	# add dvb headers for http://mcentral.de/hg/~mrec/em28xx-new
	# in reference to:
	# http://bugs.archlinux.org/task/13146
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
	cp drivers/media/dvb-frontends/lgdt330x.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/i2c/"
	cp drivers/media/i2c/msp3400-driver.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/i2c/"

	# add dvb headers
	# in reference to:
	# http://bugs.archlinux.org/task/20402
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/usb/dvb-usb"
	cp drivers/media/usb/dvb-usb/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/usb/dvb-usb/"
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends"
	cp drivers/media/dvb-frontends/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/tuners"
	cp drivers/media/tuners/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/tuners/"

	# add xfs and shmem for aufs building
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/fs/xfs"
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/mm"
	# removed in 3.17 series
	#cp fs/xfs/xfs_sb.h "${pkgdir}/usr/lib/modules/${_kernver}/build/fs/xfs/xfs_sb.h"

	# copy in Kconfig files
	for i in $(find . -name "Kconfig*"); do
		mkdir -p "${pkgdir}"/usr/lib/modules/${_kernver}/build/`echo ${i} | sed 's|/Kconfig.*||'`
		cp ${i} "${pkgdir}/usr/lib/modules/${_kernver}/build/${i}"
	done
	
	# add objtool for external module building and enabled VALIDATION_STACK option
	if [ -f tools/objtool/objtool ];  then
        mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/tools/objtool"
        cp -a tools/objtool/objtool ${pkgdir}/usr/lib/modules/${_kernver}/build/tools/objtool/ 
        fi

	chown -R root.root "${pkgdir}/usr/lib/modules/${_kernver}/build"
	find "${pkgdir}/usr/lib/modules/${_kernver}/build" -type d -exec chmod 755 {} \;

	# strip scripts directory
	find "${pkgdir}/usr/lib/modules/${_kernver}/build/scripts" -type f -perm -u+w 2>/dev/null | while read binary ; do
	case "$(file -bi "${binary}")" in
	*application/x-sharedlib*) # Libraries (.so)
		/usr/bin/strip ${STRIP_SHARED} "${binary}";;
	*application/x-archive*) # Libraries (.a)
		/usr/bin/strip ${STRIP_STATIC} "${binary}";;
	*application/x-executable*) # Binaries
		/usr/bin/strip ${STRIP_BINARIES} "${binary}";;
	esac
	done
	
        # remove a files already in linux-bfq-docs package
        rm -f "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/kbuild/Kconfig.recursion-issue-01"
        rm -f "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/kbuild/Kconfig.recursion-issue-02"
        rm -f "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/kbuild/Kconfig.select-break"

    # remove unneeded architectures
    rm -rf "${pkgdir}"/usr/lib/modules/${_kernver}/build/arch/{alpha,arm,arm26,arm64,avr32,blackfin,c6x,cris,frv,h8300,hexagon,ia64,m32r,m68k,m68knommu,mips,microblaze,mn10300,openrisc,parisc,powerpc,ppc,s390,score,sh,sh64,sparc,sparc64,tile,unicore32,um,v850,xtensa}
}

_package-docs() {
    pkgdesc="Kernel hackers manual - HTML documentation that comes with the linux-bfq kernel"
    depends=('linux-bfq')
  
    cd "${srcdir}/${_srcname}"

    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build"
    cp -al Documentation "${pkgdir}/usr/lib/modules/${_kernver}/build"
    find "${pkgdir}" -type f -exec chmod 444 {} \;
    find "${pkgdir}" -type d -exec chmod 755 {} \;

    # remove a file already in linux package
    rm -f "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/DocBook/Makefile"
}

pkgname=("${pkgbase}" "${pkgbase}-headers" "${pkgbase}-docs")
for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    $(declare -f "_package${_p#${pkgbase}}")
    _package${_p#${pkgbase}}
  }"
done

sha512sums=('a48a065f21e1c7c4de4cf8ca47b8b8d9a70f86b64e7cfa6e01be490f78895745b9c8790734b1d22182cf1f930fb87eaaa84e62ec8cc1f64ac4be9b949e7c0358'
            'SKIP'
            'c8ad31fe9d6a555e5b219cb4e751f633916a66e7d2d4b70a169dc55fe786d9b25000c29950fe39342c6a8a958ad036b4c642fab7e16ece6e6d76eafe794d9a92'
            'SKIP'
            '95a7b9dc5a6c378b19e199285b5c1c397ca0ca0cf03c42d185b57da68329e59d59294d1879998f4020a0dee10d36c550acf30f28970c82adb2e7604c86424178'
            'dc0649dfe2a5ce8e8879a62df29a4a1959eb1a84e5d896a9cb119d6a85a9bad1b17135371799e0be96532e17c66043d298e4a14b18fad3078a1f425108f888c9'
            '135afcffee2439e2260a71202658dce9ec5f555de3291b2d49a5cffdd953c6d7851b8a90e961576895555142a618e52220a7e4a46521ca4ba19d37df71887581'
            '87ae76889ab84ced46c237374c124786551982f8ff7325102af9153aed1ab7be72bec4db1f7516426c5ee34c5ce17221679eb2f129c5cbdeec05c0d3cb7f785d'
            '8e8f407262f3f573654e7f0fceec6075fd2da0258a384206d28e5f59698168609e575f3d855adb8bb476b0684da409dab22f7ddc0f00e9c10c67debe4409c30b'
            'd9d28e02e964704ea96645a5107f8b65cae5f4fb4f537e224e5e3d087fd296cb770c29ac76e0ce95d173bc420ea87fb8f187d616672a60a0cae618b0ef15b8c8'
            '643cad54fc54f4790e1d28dbc60f775f117fa3e25e15aff418b0406cc7a9b49db5d67fc02ff7056a951a2d64d5bbc58306c40fa0d588460664c2921ad613745c'
            '94fbb7e67b77bb8911b69cdb23bbb74317e75a4ed786e4c6486bf2927042f9374e932ea879d1b250837e42c628abf7082fd6f895a2257c5d3ad652f6a520306a'
            'd6faa67f3ef40052152254ae43fee031365d0b1524aa0718b659eb75afc21a3f79ea8d62d66ea311a800109bed545bc8f79e8752319cd378eef2cbd3a09aba22'
            '2dc6b0ba8f7dbf19d2446c5c5f1823587de89f4e28e9595937dd51a87755099656f2acec50e3e2546ea633ad1bfd1c722e0c2b91eef1d609103d8abdc0a7cbaf'
            '3889679e288d51f6fecc7ed6581ccde34acbf1e4861f5c9ca237a1ad13158502757d3fc457d7b6caf1c8c99c9dba842265004154a71cffb8ec14e1030e49e312'
            '3c3f3b6407d9a1a63cd91c2b5c35e6c932afa5bf33f1b2e8a530dbd9eacda8c8f956616c4cc9228657624da58e354ba5714252237d84ff3386fd65cf44f06ddc')
            
validpgpkeys=(
              'ABAF11C65A2970B130ABE3C479BE3E4300411886' # Linus Torvalds
              '647F28654894E3BD457199BE38DBBDC86092693E' # Greg Kroah-Hartman
             )
