# zabbix-borg
Zabbix Plugin for monitoring Borg Backup

## Items
There are 10 Items:

![Zabbix Borg Items](pics/items.png)

## Triggers
There are 3 Triggers:

![Zabbix Borg Triggers](pics/triggers.png)

## Install
### Zabbix 
- Import template into zabbix server and configure hosts to use it.
- Deploy zabbix_agentd.conf.d/borg.conf to every machine, you want to use the template.

If you want to test it, use `zabbix_get -s IP -p 10050 -k borg.status`.

### Borg Backup
To let the template work, you must redirect borg output to `/var/log/borg.log`. Second build `/var/log/borg.status` like this:
```
borg create ... > /var/log/borg.log 2>&1

grep 'Archive name:' /var/log/borg.log > /dev/null 2>&1
if [ $? -ne 0 ]; then
  echo "$borg backup failed! Exiting"
  echo "$(date +%Y-%m-%d-%H:%M:%S): FAILED" > /var/log/borg.status
  exit 1
fi

ARCHIVESIZE=$(grep 'This archive' /var/log/borg.log |awk '{print $7}')
BC=$(echo $ARCHIVESIZE'>'0| bc -l)
if [ "$BC" -eq 1 ]; then
  echo "OK" > /var/log/borg.status
else
  echo "$(date +%Y-%m-%d-%H:%M:%S): FAILED" > /var/log/borg.status
  exit 1
fi
```

Output must be like this:
```
root@mail:~# cat /var/log/borg.status 
OK
```

```
root@mail:~# cat /var/log/borg.log 
Creating archive at "ssh://test@test.de/mnt/borg::mail-2020-03-03-20:02:54"
------------------------------------------------------------------------------
Archive name: mail-2020-03-03-20:02:54
Archive fingerprint: 2067bd6dd2e7bc2a27d373ef293eda59022d84d4f0f8ce860221a5123f77ba7a
Time (start): Tue, 2020-03-03 20:02:58
Time (end):   Tue, 2020-03-03 20:03:36
Duration: 37.84 seconds
Number of files: 277055
Utilization of max. archive size: 0%
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
This archive:               40.32 GB             31.61 GB             11.25 MB
All archives:                5.68 TB              4.48 TB             30.36 GB

                       Unique chunks         Total chunks
Chunk index:                  338444             39507784
------------------------------------------------------------------------------
```

