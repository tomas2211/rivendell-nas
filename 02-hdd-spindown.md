# HDD spindown

Setting spindown timeout through `hdparm -S1 /dev/sda` did not work on my WD Gold. I decided to use `hd-idle` daemon for that.

1. Install from apt (don't bother building from sources) `sudo apt install hd-idle`
2. Set timeout to 30 mins:
```bash
tomas@rivendell:~$ cat /etc/default/hd-idle
# defaults file for hd-idle

# start hd-idle automatically?
START_HD_IDLE=true

#HD_IDLE_OPTS="-h"

HD_IDLE_OPTS="-a sda -i 1800 -l /var/log/hd-idle.log"
```

3. Enable in systemd:
```bash
sudo systemctl enable hd-idle
sudo systemctl start hd-idle
```


## Monitor disk usage

```bash
sudo dstat -tdD /dev/sda --top-io
```

```bash
sudo iotop
```
