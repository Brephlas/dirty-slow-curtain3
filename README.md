# dirty-slow-curtain3
*Dirty solution to have both fast and slow speed available for Switchbot Curtain 3*


This solution requires you to replace `/homeassistant/deps/lib/python3.12/site-packages/switchbot/devices/curtain.py` with the modified file. If you need the original one back, you can get it from the original [repository](https://github.com/Danielhiversen/pySwitchbot/tree/master). 

I used the file `/config/custom_components/.slow` as a binary identification between the modes. You can modify the path in the file `curtain.py` by changing the global variable `FILE_SPEED_IDENTIFIER`.

This is a pretty dirty and hacky solution but it is working fine for me and since I am not into integration development I was not able to develop a better solution quickly.

## Steps to apply this to your installation

1) Replace the `curtain.py` file inside the docker container: `docker cp /homeassistant/curtain.py homeassistant:/usr/local/lib/python3.12/site-packages/switchbot/devices/curtain.py`
   
   Alternatively, you can add the `pyswitchbot`-repo using [Custom deps deployment](https://github.com/home-assistant/addons-development?tab=readme-ov-file) and replace the `curtain.py` file there.
   The path is:
   - before HA 2024.2: `/homeassistant/deps/lib/python3.11/site-packages/switchbot/devices/curtain.py`
   - after HA 2024.2: `/homeassistant/deps/lib/python3.12/site-packages/switchbot/devices/curtain.py`
     
   If it was installed with `python3.11` you can just rename the folder to `python3.12`
4)  Adjust `configuration.yaml` and add the directory where the identifier `.slow` file should be saved to the `allowlist_external_dirs`
    ```
    allowlist_external_dirs:
        - "/config/custom_components"
    ```

5) Create directory `/config/custom_components` and file `.slow` inside
6) Write 0 or 1 into the file
7) Add shell_command to enable or disable slow mode for the curtains
    ```
    switchbot_curtain_enable_slow: 'echo "1" > /config/custom_components/.slow'
    switchbot_curtain_disable_slow: 'echo "0" > /config/custom_components/.slow'
    ```
8) Reboot Home Assistant
9) Have fun and use it by calling either one of the services before calling the `cover`-services:
    ```
    shell_command.switchbot_curtain_enable_slow
    shell_command.switchbot_curtain_disable_slow
    ```

## What does this modification do?

Basically, this modification only checks if `0` or `1` is in the file `.slow` and use this as binary identification to either go slow or fast. The following excerpt shows the file read:
```py
with open(FILE_SPEED_IDENTIFIER, "r") as fr:
    r = int(fr.read().strip())
    if r == 1:
        self.slow = 1
    else:
        self.slow = 255
```

If you for example call the `cover.open_cover` service, the speed will be set before sending the command to the switchbot:
```py
speed = self.readState()
```

# Credits

- Thanks to the work by @the-ress who did the work to identify the necessary data for the slow mode: https://github.com/the-ress/pySwitchbot/tree/curtain-speed 
- Also thanks to the original repository from Danielhiversen: https://github.com/Danielhiversen/pySwitchbot
