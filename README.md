# openwrt-hotplug-button-reboot
Reboot button script for OpenWrt on Linksys WRT610N V1, hold down the WPS button for 10 seconds and release to reboot

Mileage may vary for other models but the syntax is pretty much the same

Repo is mostly just here as a personal reference for `ash` syntax

OpenWrt button operation source: https://openwrt.org/docs/guide-user/hardware/hardware.button#hotplug_buttons

## Install
```
opkg update
opkg install kmod-button-hotplug
mkdir -p /etc/hotplug.d/button

cat << "EOF" > /etc/hotplug.d/button/buttons
logger "the button was ${BUTTON} and the action was ${ACTION}"
if [ $BUTTON == "wps" ] ; 
    then
        if [ $ACTION == "pressed" ] ;
        then
            echo $(date +%s) > /tmp/button.txt
            logger "wps press time logged"
        elif [ $ACTION == "released" ] ;
            then
                TIMECHECK=$(cat /tmp/button.txt)
                DURATION=$(($(date +%s) - $TIMECHECK))
                logger "wps release time logged"
                logger "press duration was ${DURATION}"
                if [ $DURATION -ge 10 ] ;
                    then
                        logger "[!!] 10s button hold, rebooting! [!!]"
                        reboot
                fi
        fi
fi
EOF
```
Modify as desired: button may have a different internal name than `"wps"` (use `logread` in router shell to identify), button press duration can be adjusted (change number in `if [ $DURATION -ge 10 ] ;` to whatever you wish)
