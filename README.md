
# hostapd-wpe-openwrt
Updated: 20/12/2020 

Hostapd-wpe (Wireless Pwnage Edition) packages for OpenWRT 19.07. Some effort was done to make the build as small as possible and reuse OpenWrt components where possible. The build uses openssl and libubox as only dependencies

The WPE patch was taken from the [aircrack-ng repository](https://github.com/aircrack-ng/aircrack-ng/blob/master/patches/wpe/hostapd-wpe/hostapd-wpe.patch) and applied to the hostapd in OpenWrt. This build includes Cupid attack (for HeartBleed), Karma mode and Mschapv2 password fixes.

## Build this package for your architecture
Clone this repository.

`git clone https://github.com/jekkos/hostapd-wpe-openwrt`

Next edit `feeds.conf.default` in OpenWrt root folder and add following line to the end of the file. 

`src-link hostapd-wpe /path/to/hostapd-wpe-openwrt`

in OpenWrt folder update the feeds and install them

`./scripts/feeds update -a`
`./scripts/feeds install -a`

Then compile the package

`make package/network/services/hostapd-wpe/compile V=s`

.ipk files can be found in `bin/packages/<arch>/base/` (in this case arch=mips)

* hostapd-wpe_git-2_mips_24kc.ipk
* hostapd-common_2019-08-08-ca8c2bd2-4_mips_24kc.ipk
* libubox20191228_2020-05-25-66195aee-1_mips_24kc.ipk (you probably already have this installed)
* libopenssl1.1_1.1.1i-1_mips_24kc.ipk (you probably already have this installed)

## Download and install .ipk directly for mips
The package is here precompiled for mips24kc (tplink wr703n). It only has openssl as a dependency. Download the .ipk files in the download section here, copy to `/tmp` and then install as follows

`opkg install /tmp/hostapd-common_2019-08-08-ca8c2bd2-4_mips_24kc.ipk`
`opkg install /tmp/hostapd-wpe_git-2_mips_24kc.ipk`

## Run on OpenWrt
Install coreutils-nohup to keep the process up after you detach SSH

`opkg update && opkg install coreutils-nohup`
`nohup hostapd-wpe -i wlan1 -k -s /etc/hostapd-wpe/hostapd-wpe.conf &` to start the process. A config file can be found in `/etc/hostapd-wpe/hostapd-wpe.conf`

The process will write it's logging to `~/hostapd-wpe.log` after you send it a `SIGTERM` signal.
