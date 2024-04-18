# lpac-libmbim-wrapper
This is a wrapper for LPAC client to use `mbimcli` to manage **eUICC** on Linux client.

# Acknowledgments
I want to extend a big thanks to [@z3ntu](https://github.com/z3ntu/) for his original [work](https://github.com/z3ntu/lpac-libqmi-wrapper). I've just rewritten the parser to make it work with `mbimcli`.

# How to use
Download `wrapper.py` and make sure to have LPAC client installed. Refer to this [link](https://github.com/estkme-group/lpac) for instructions on how to do so.

## Some examples:

To display a list of installed profiles, simply type:

`./wrapper.py profile list`

Example output:

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

All other commands are similar to LPAC. Just use wrapper instead of lpac as the first argument. Make sure to have access to the MBIM port, otherwise, run it as root.

It has been tested on a *Foxconn T99W175* module in a *Lenovo ThinkPad X1C-G9*. Make sure to have selected the correct slot (slot 0=Physical SIM, slot 1=eSIM).
You can swap slot with `mbimcli` using this command:

`mbimcli -p -d /dev/wwan0mbim0 --ms-set-device-slot-mappings=1`

If you want to know which slot is active, just type:

`mbimcli -p -d /dev/wwan0mbim0 --ms-query-device-slot-mappings`

I've planned to test using a physical eSIM over MBIM, similar to [this](https://www.lenovo.com/it/it/p/accessories-and-software/mobile-broadband/4g-lte/4xc1l91362), in case your modem doesn't support eUICC AT commands.

Just for reference, here are the required AT commands to interact with the eUICC:
- `AT+CCHO` to open logical channel
- `AT+CCHC` to close logical channel 
- `AT+CGLA` to use logical channel access

## Bugs & know issues

Delete profile doesn't seem to contact SM-DS server, so it **doesn't release** the eSIM profile. I'm tryting to understandy why.
I'm not a Python expert, so it's possible that there are some bugs present. Feel free to file an issue and/or submit a pull request to enhance the code :)
