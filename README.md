# Jellyfin Tizen app Devcontainer

This project provides a devcontainer work environment for building a installing the Jellyfin Tizen app for Samsung Smart TVs

The main process is from the following sources
* https://www.reddit.com/r/jellyfin/comments/s0438d/build_and_deploy_jellyfin_app_to_samsung_tizen/
* https://github.com/babagreensheep/jellyfin-tizen-docker


# Tizen SDK
The Tizen SDK is included as part of the repository devcontainer, and a volume has been created for the `/home/node/tizen-studio-data` path so that profile and certificate data is persisted across restarts.

```bash
tizen certificate -a TizenCert -p P@ssw0rd! -c CountryCode -ct MyCity -o "My Company" -n "My Name" -e my@email.com -f tizencert

tizen security-proviles add -n MyProfileName -a /home/node/tizen-studio-data/keystore/author/tizencert.p12 -p P@ssw0rd!

# Replace your certificate password in profiles.xml
sed -i 's/\/home\/node\/tizen-studio-data\/keystore\/author\/tizencert.pwd/P@ssword!/' /home/node/tizen-studio-data/profile/profiles.xml

# Default password for the Distribution certificate is: tizenpkcs12passfordsigner
sed -i 's/\/home\/node\/tizen-studio-data\/tools\/certificate-generator\/certificates\/distributor\/tizen-distributor-signer.pwd/tizenpkcs12passfordsigner/' /home/node/tizen-studio-data/profile/profiles.xml
```

**_NOTE:_** If your password is weak, the contents of the `profiles.xml` file will be overwritten each time you perform a `tizen package` build (so use a decent password).

# Build Jellyfin Tizen

Now you can follow the instructions of the [JellyFin-Tizen](https://github.com/jellyfin/jellyfin-tizen) using this configured devcontainer work environment.

1. Open this repository in VSCode in a Dev Container.
1. Open a terminal in VSCode
1. Follow the commandline instructions outlined in the [JellyFin-Tizen Readme](https://github.com/jellyfin/jellyfin-tizen/blob/master/README.md)

```
git clone https://github.com/jellyfin/jellyfin-web.git
git clone https://github.com/jellyfin/jellyfin-tizen.git

cd jellyfin-web/
git fetch
git checkout -b release-10.9.z origin/release-10.9.z
SKIP_PREPARE=1 npm ci --no-audit
USE_SYSTEM_FONTS=1 npm run build:production

cd ..
cd jellyfin-tizen/
JELLYFIN_WEB_DIR=../jellyfin-web/dist npm ci --no-audit
tizen build-web -e ".*" -e gulpfile.js -e README.md -e "node_modules/*" -e "package*.json" -e "yarn.lock"
tizen package -t wgt -o . -s MyProfileName -- .buildResult

sdb connect MY.TV.IP.ADDR
sdb devices
tizen install -n Jellyfin.wgt -t <DEVICE_NAME>
```
