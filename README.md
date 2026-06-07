# Customizing the LG WebOS 7 / WebOS22 Homescreen

How to replace the background image, remove unwanted UI elements, and change system text on your LG TV — no permanent changes to the read-only filesystem.

> **Tested on:** LG WebOS 7.5.2 (Rockhopper), VN region  
> **Requirements:** Root access, active SSH connection, Homebrew Channel with `webosbrew` init.d support

---

# Before / After
## Before
<img width="1080" height="616" alt="1000034656" src="https://github.com/user-attachments/assets/dc7205e8-3881-4097-b991-31a3b30354eb" />

## After
<img width="1080" height="699" alt="1000034690" src="https://github.com/user-attachments/assets/a2360f31-5b9e-4b54-9b81-2ac24fd3b362" />



## Background

On webOS 22, the Home app has been completely rewritten using QML. The layout and user interface are defined entirely within QML files

```
/usr/palm/applications/com.webos.app.home/qml/UserInterfaceLayer/Containers/MainView.qml
```

This directory is on a read-only, cryptographically signed filesystem, so we can't edit files directly. Instead, we use a **bind mount overlay**: copy the assets to a writable location in `/tmp`, apply our modifications, and mount that over the original directory. The original files are never touched.


## Implementation steps
Run each of the following commands.
First
```
mkdir -p /media/developer/apps/usr/palm/applications/tld.my.customhome/UserInterfaceLayer/Containers
```
```
mkdir -p /media/developer/apps/usr/palm/applications/tld.my.customhome/assets-custom
```
```
cp /usr/palm/applications/com.webos.app.home/qml/UserInterfaceLayer/Containers/MainView.qml /media/developer/apps/usr/palm/applications/tld.my.customhome/UserInterfaceLayer/Containers/MainView.qml
```
```
cat > /media/developer/apps/usr/palm/applications/tld.my.customhome/apply.sh << 'EOF'
#!/bin/sh
set -e -o pipefail -x
APP_DIR=/usr/palm/applications/com.webos.app.home/qml
OVERRIDE_BASEPATH=/media/developer/apps/usr/palm/applications/tld.my.customhome
rm -rf /tmp/weboshome-merged
cp -R "$APP_DIR" /tmp/weboshome-merged
cp -R "$OVERRIDE_BASEPATH/UserInterfaceLayer" /tmp/weboshome-merged/
cp -R "$OVERRIDE_BASEPATH/assets-custom" /tmp/weboshome-merged/
mount --bind /tmp/weboshome-merged "$APP_DIR"
pkill -f com.webos.app.home
EOF
```
```
chmod +x /media/developer/apps/usr/palm/applications/tld.my.customhome/apply.sh
```
```
ln -sf /media/developer/apps/usr/palm/applications/tld.my.customhome/apply.sh /var/lib/webosbrew/init.d/49-custom-homescreen
```

Second.
```
sed -i 's/visible: (!systemProperties.isTopShelfGuide && systemProperties.isNoAdvertisement && !uiController.isEditMode) === true && uiController.isFullHome && (shelfListContainer.state === "app" || interfaces.application.isDefaultShelfList)/visible: false/g' /media/developer/apps/usr/palm/applications/tld.my.customhome/UserInterfaceLayer/Containers/MainView.qml
```
```
sed -i '22a\
\
    Image {\
        id: customBackground\
        anchors.fill: parent\
        source: "/media/developer/apps/usr/palm/applications/tld.my.customhome/assets-custom/background.jpg"\
        fillMode: Image.PreserveAspectCrop\
        z: 0\
    }' /media/developer/apps/usr/palm/applications/tld.my.customhome/UserInterfaceLayer/Containers/MainView.qml
```
```
 sed -i '291s/active: (systemProperties.isTopShelfGuide && !uiController.isEditMode) === true && uiController.isFullHome/active: false/' /media/developer/apps/usr/palm/applications/tld.my.customhome/UserInterfaceLayer/Containers/MainView.qml
```
```
sed -i '321s/active: (!systemProperties.isTopShelfGuide && systemProperties.isNoAdvertisement && !uiController.isEditMode) === true && uiController.isFullHome && (shelfListContainer.state === "app" || interfaces.application.isDefaultShelfList)/active: false/' /media/developer/apps/usr/palm/applications/tld.my.customhome/UserInterfaceLayer/Containers/MainView.qml
```

```
sed -i '501a\            visible: false' /media/developer/apps/usr/palm/applications/tld.my.customhome/UserInterfaceLayer/Containers/MainView.qml
```

```
sed -n '499,505p' /media/developer/apps/usr/palm/applications/tld.my.customhome/UserInterfaceLayer/Containers/MainView.qml
```
```
umount /usr/palm/applications/com.webos.app.home/qml ; rm -rf /tmp/weboshome-merged && sh /media/developer/apps/usr/palm/applications/tld.my.customhome/apply.sh
```





## Notes & Limitations
- The image file must be saved in the /media/developer/apps/usr/palm/applications/tld.my.customhome/assets-custom/You upload here.jpg
- The image resolution should be set to 3840×2160.
- The QML element names and file structure may differ between TV models, regions, and WebOS versions. Always inspect the original files on your specific TV first.
- The overlay is applied in `/tmp` and is lost on reboot — that's intentional and makes this approach safe to experiment with.

## Credits
This article is adapted from https://github.com/mareklarek/webos10-homescreen-customization/tree/main
