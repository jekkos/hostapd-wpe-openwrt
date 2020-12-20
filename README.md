
# hostapd-wpe-openwrt
Updated: 20/12/2020 

Hostapd-wpe (Wireless Pwnage Edition) packages for OpenWRT 19.07. Some effort was done to make the build as small as possible and to reuse OpenWrt components where possible. The build uses openssl and libubox as only dependencies

The WPE patch was taken from the [aircrack-ng repository](https://github.com/aircrack-ng/aircrack-ng/blob/master/patches/wpe/hostapd-wpe/hostapd-wpe.patch) and applied to the hostapd in OpenWrt. This build includes Cupid attack (for HeartBleed), Karma mode and Mschapv2 password fixes.

## Add this repository to your build

Clone this repository.

`git clone https://github.com/jekkos/hostapd-wpe-openwrt`

Next edit `feeds.conf.default` in OpenWrt root folder and add following line to the end of the file. 

`src-link hostapd-wpe /path/to/hostapd-wpe-openwrt`

Next in OpenWrt root update the feeds and install the buildfiles

`./scripts/feeds update -a`
`./scripts/feeds install -a`

Then compile the package

`make package/network/services/hostapd-wpe/compile V=s`

.ipk files can then be fonud in `bin/packages/<arch>/base/` (in this case arch=mips)

* hostapd-wpe_git-2_mips_24kc.ipk
* hostapd-common_2019-08-08-ca8c2bd2-4_mips_24kc.ipk
* libubox20191228_2020-05-25-66195aee-1_mips_24kc.ipk (you probably already have this installed)
* libopenssl1.1_1.1.1i-1_mips_24kc.ipk (you probably already have this installed)

## Download and install .ipk directly
The package is here precompiled for mips24kc. It only has openssl as a dependency.

## Run on OpenWrt
`hostapd-wpe -i wlan1 -k /etc/hostapd-wpe/hostapd-wpe.conf` to start the process. A config file can be found in `/etc/hostapd-wpe/hostapd-wpe.conf`
