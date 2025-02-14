# Deploying vodle via [F-Droid](https://f-droid.org)

Status: trying to find out how this works by reading 
- https://f-droid.org/en/docs/Inclusion_How-To/ : Proposal by Metadata Merge Request
- https://f-droid.org/en/docs/Submitting_to_F-Droid_Quick_Start_Guide/
- https://f-droid.org/en/docs/Installing_the_Server_and_Repo_Tools/
- https://f-droid.org/en/docs/Setup_an_F-Droid_App_Repo/
- https://f-droid.org/en/docs/Build_Metadata_Reference/
- https://f-droid.org/en/docs/All_About_Descriptions_Graphics_and_Screenshots/
- https://f-droid.org/en/docs/Signing_Process/

## TODO:

- add license header to all source code files?
- how to credit node_modules and add license information for them?
- licenses for artwork?
- adjust AndroidManifest.xml and build.gradle as in https://capacitorjs.com/docs/android/configuration:
    - permissions: https://developer.android.com/reference/android/Manifest.permission
        - internet
        - check network status: ACCESS_NETWORK_STATE
        - share: none?
        - notifications: POST_NOTIFICATIONS ?
        - deep links
        - open outside links
- fix icon (in AndroidManifest.xml it says `android:icon="@mipmap/ic_launcher"`, how to change this? https://developer.android.com/guide/topics/manifest/manifest-intro)

## Directories
- ~/git/vodle: clone of the app's source repo https://github.com/pik-gane/vodle
- ~/git/fdroidserver: clone of https://gitlab.com/fdroid/fdroidserver
    - used to build the unsigned apk
- ~/git/fdroiddata: fork of the fdroid catalogue https://gitlab.com/fdroid/fdroiddata
    - used as a kind of working directory by fdroidserver
    - contains the unsigned apk after building  
- ~/fdroid: local "fdroid repo"
    - contains PRIVATE information (keystore.p12 and config.yml)
    - used for signing the apk (and publishing it?) 

## Setting up build environment

### https://f-droid.org/en/docs/Submitting_to_F-Droid_Quick_Start_Guide/
- forked fdroiddata, switched to branche it.vodle
- added `metadata/it.vodle.yml` and edited it, starting with versionCode 1
- on console:
```
sudo docker run --rm -itu vagrant --entrypoint /bin/bash \
-v /usr/bin/apksigner:/usr/bin/apksigner:ro \
-v ~/git/fdroiddata:/build:z \
-v ~/git/fdroidserver:/home/vagrant/fdroidserver:Z \
registry.gitlab.com/fdroid/fdroidserver:buildserver
```
    - in container (takes about 10 minutes):
```
. /etc/profile
export PATH="$fdroidserver:$PATH" PYTHONPATH="$fdroidserver"
cd /build
fdroid readmeta
#fdroid rewritemeta it.vodle
fdroid checkupdates it.vodle
fdroid lint it.vodle
fdroid build --on-server it.vodle
```
- finally this succeeded and produced an unsigned APK.

### https://f-droid.org/en/docs/Installing_the_Server_and_Repo_Tools/
- installed servertools
- mkdir ~/fdroid
- cd ~/fdroid
- fdroid init --repo-keyalias vodle.it
- edited config.yml to say `keydname: CN=vodle.it, OU=F-Droid`
- backed up config.yml and keystore.p12
- touch metadata/com.example.app.yml
- `cp ~/git/fdroiddata/unsigned/it.vodle_1.apk ~/fdroid/unsigned/`
- fdroid publish --verbose
- this moved the apk from unsigned to repo
- fdroid update --verbose
- fdroid gpgsign

### https://f-droid.org/en/docs/Setup_an_F-Droid_App_Repo/#real-world-setup
- ?

## Building new version

- in android/app/build.gradle, android/app/release/output-metadata.json, and package-lock.json, increase versionName and increment versionCode
- commit and push these changes, note the commit id
- in it.vodle.yml:
    - add above versionName, versionCode and commit id as new entry under Builds
    - adjust CurrentVersion, CurrentVersionCode to those values
- commit 
- proceed with sudo docker run as stated above:
```
sudo docker run --rm -itu vagrant --entrypoint /bin/bash \
-v /usr/bin/apksigner:/usr/bin/apksigner:ro \
-v ~/fdroiddata:/build:z \
-v ~/fdroidserver:/home/vagrant/fdroidserver:Z \
registry.gitlab.com/fdroid/fdroidserver:buildserver
```
- in container (takes about 10 minutes):
```
. /etc/profile
export PATH="$fdroidserver:$PATH" PYTHONPATH="$fdroidserver"
cd /build
fdroid readmeta
#fdroid rewritemeta it.vodle
fdroid checkupdates it.vodle
fdroid lint it.vodle
fdroid build --verbose --on-server it.vodle
```
- then sign it:
```
cd ~/fdroid
cp ~/fdroiddata/unsigned/it.vodle_xxx.apk ~/fdroid/unsigned/
fdroid publish --verbose
fdroid update --verbose
fdroid gpgsign
```
- WHAT THEN? just commit fdroiddata?

## Necessary meta-data

- applicationId: it.vodle

## Meeting F-Droid Prerequisites

- Licenses:
    - MIT mostly, some Apache 2.0, a few ISC, one "unlicense"

### Only use acceptably licensed components
- see https://forum.f-droid.org/t/nativescript-apache-cordova-ionic-framework-apps/9442



 