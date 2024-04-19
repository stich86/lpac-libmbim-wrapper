# lpac-libmbim-wrapper
This is a wrapper for LPAC client to use `mbimcli` to manage **eUICC** on Linux client.

# Acknowledgments
I want to extend a big thanks to [@z3ntu](https://github.com/z3ntu/) for his original [work](https://github.com/z3ntu/lpac-libqmi-wrapper). I've just rewritten the parser to make it work with `mbimcli`.

# How to use
- Install or compile LPAC client, please refer to this [link](https://github.com/estkme-group/lpac) for instructions
- Download `wrapper.py`, edit it to point the right device using the variable `DEV` and make it executable (`chmod +x wrapper.py`)
- Use `./wrapper.py command`

## Some examples:

To display a list of installed profiles, simply type:

`./wrapper.py profile list`

Output:

```
INFO: Connect
INFO: Open channel with AID a0000005591010ffffffff8900000100
INFO: Received LPA data. Printing...
{'payload': {'code': 0,
             'data': [{'iccid': '89XXXXXXXXXXXXX',
                       'icon': None,
                       'iconType': None,
                       'isdpAid': 'a0000005591010ffffffff8900001100',
                       'profileClass': 'operational',
                       'profileName': 'Tata_EP3303v3_TEA02759_TEA02760_Firsty_20K',
                       'profileNickname': None,
                       'profileState': 'enabled',
                       'serviceProviderName': 'Firsty'}],
             'message': 'success'},
 'type': 'lpa'}
INFO: Close channel 01
INFO: Disconnect
Exit code: 0
```

All other commands are based on [LPAC](https://github.com/estkme-group/lpac?tab=readme-ov-file#usage) binary. 
Make sure to have access to the MBIM port, otherwise, run it as root.

# Test Done

Here is a list of tests that I've done, both with embedded and [physical](https://www.lenovo.com/it/it/p/accessories-and-software/mobile-broadband/4g-lte/4xc1l91362) eSIM:

| Modem Tested                     | AT | MBIM |
|----------------------------------|----|------|
| Foxconn T99W175 (Lenovo version) | ❌  | ✅    |
| Quectel RM502Q-GL                | ⚠️  | ✅    |

When using ***Foxconn T99W175*** select the correct slot based on type of eSIM you are using (slot **0**=Physical SIM, slot **1**=Embedded eSIM)
You can swap slot with `mbimcli` using this command:

`mbimcli -p -d /dev/wwan0mbim0 --ms-set-device-slot-mappings=1`

If you want to know which slot is active, just type:

`mbimcli -p -d /dev/wwan0mbim0 --ms-query-device-slot-mappings`

For AT provisioning, the modem needs these commands to interact with the eUICC:

- `AT+CCHO` to open logical channel
- `AT+CCHC` to close logical channel 
- `AT+CGLA` to use logical channel access

**T99W175** lacks AT commands, while Quectel seems to work only for these:
- Chip Info
- List Profile
- Enable/Disable Profile
- Delete Profile
- Notification List

`Download Profile` and `Notification Process` are not working!

Plese note if you use **AT** backend, this wrapper is not needed.

# Bugs & know issues

Delete profile doesn't contact SM-DS server automatically, so it **doesn't detach** the eSIM profile. 
You should run these commands to release it: 

Get all tasks issued on the eUICC:

`./wrapper notification list`

Output:

```
{'payload': {'code': 0,
             'data': [{'iccid': '8931XXXXXXXXXXXXXXX',
                       'notificationAddress': 'dp-plus-par07-01.oasis-smartsim.com',
                       'profileManagementOperation': 'install',
                       'seqNumber': 6},
                      {'iccid': '8939XXXXXXXXXXXXXXX',
                       'notificationAddress': 'frm.prod.ondemandconnectivity.com',
                       'profileManagementOperation': 'disable',
                       'seqNumber': 2},
                      {'iccid': '8939XXXXXXXXXXXXXXX',
                       'notificationAddress': 'frm.prod.ondemandconnectivity.com',
                       'profileManagementOperation': 'enable',
                       'seqNumber': 3},
                      {'iccid': '8939XXXXXXXXXXXXXXX',
                       'notificationAddress': 'frm.prod.ondemandconnectivity.com',
                       'profileManagementOperation': 'disable',
                       'seqNumber': 4},
                      {'iccid': '8939XXXXXXXXXXXXXXX',
                       'notificationAddress': 'frm.prod.ondemandconnectivity.com',
                       'profileManagementOperation': 'delete',
                       'seqNumber': 5},
                      {'iccid': '8931XXXXXXXXXXXXXXX',
                       'notificationAddress': 'dp-plus-par07-01.oasis-smartsim.com',
                       'profileManagementOperation': 'enable',
                       'seqNumber': 7},
                      {'iccid': '8931XXXXXXXXXXXXXXX',
                       'notificationAddress': 'dp-plus-par07-01.oasis-smartsim.com',
                       'profileManagementOperation': 'disable',
                       'seqNumber': 8},
                      {'iccid': '8931XXXXXXXXXXXXXXX',
                       'notificationAddress': 'dp-plus-par07-01.oasis-smartsim.com',
                       'profileManagementOperation': 'delete',
                       'seqNumber': 9}],
             'message': 'success'},
 'type': 'lpa'}
```

Look at **profileManagementOperation** type `delete` and take not of `seqNumber` value, then issue the notification command to release the eSIM profile:

`./wrapper notification process -r 5` <-- this will tell remote SM-DS server to release eSIM profile for ICCID `8939XXXXXXXXXXXXXXX`.

`./wrapper notification process -r 9` <-- this will tell remote SM-DS server to release eSIM profile for ICCID `8931XXXXXXXXXXXXXXX`.

# Others

I'm not a Python expert, so it's possible that there are some bugs present. Feel free to file an issue and/or submit a pull request to enhance the code :)
