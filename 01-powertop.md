# Powertop setup

Utility powertop (2.15) built and installed from sources https://github.com/fenrus75/powertop.

See `~/repos/powertop`.

Auto-tune, more specifically setting "SATA link power management", brought down idle power consumption to ~10W (from ca. 15W) and allowed reaching C10 states (only C6 without).

## Auto-tune on startup

The settings were not preserved between reboots. Powertop auto-tune is therefore run each statup using `systemd`.

Add service file:
```conf
tomas@rivendell:~$ cat /etc/systemd/system/powertop.service
[Unit]
Description=PowerTOP auto tune

[Service]
Type=oneshot
Environment="TERM=dumb"
RemainAfterExit=true
ExecStart=/usr/local/sbin/powertop --auto-tune

[Install]
WantedBy=multi-user.target
```

Enable service:
```
systemctl daemon-reload
systemctl enable powertop.service
```


Source: https://askubuntu.com/questions/112705/how-do-i-make-powertop-changes-permanent

## Auto-tune dump

```bash
## auto-tune-dump commands BEGIN


## NMI watchdog should be turned off
echo '0' > '/proc/sys/kernel/nmi_watchdog';

## Enable SATA link power management for host0
echo 'med_power_with_dipm' > '/sys/class/scsi_host/host0/link_power_management_policy';

## Enable SATA link power management for host2
echo 'med_power_with_dipm' > '/sys/class/scsi_host/host2/link_power_management_policy';

## Enable SATA link power management for host4
echo 'med_power_with_dipm' > '/sys/class/scsi_host/host4/link_power_management_policy';

## Enable SATA link power management for host6
echo 'med_power_with_dipm' > '/sys/class/scsi_host/host6/link_power_management_policy';

## Enable SATA link power management for host1
echo 'med_power_with_dipm' > '/sys/class/scsi_host/host1/link_power_management_policy';

## Enable SATA link power management for host3
echo 'med_power_with_dipm' > '/sys/class/scsi_host/host3/link_power_management_policy';

## Enable SATA link power management for host5
echo 'med_power_with_dipm' > '/sys/class/scsi_host/host5/link_power_management_policy';

## Enable SATA link power management for host7
echo 'med_power_with_dipm' > '/sys/class/scsi_host/host7/link_power_management_policy';

## VM writeback timeout
echo '1500' > '/proc/sys/vm/dirty_writeback_centisecs';

## auto-tune-dump commands END
```
