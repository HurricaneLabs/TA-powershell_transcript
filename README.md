# Hurricane Labs Add-on for Windows PowerShell Transcript

## Support
- Splunk 7.3, 8.0

## How This App Works
This app provides knowledge objects for working with Windows PowerShell transript logs.  

## About Hurricane Labs Add-on for Windows PowerShell Transcript

| Author | Tom Kopchak, Hurricane Labs |
| --- | --- |
| App Version | 0.1.0 |
| Vendor Products | Windows PowerShell |
| Has index-time operations | true - line breaking |
| Create an index | false |
| Implements summarization | false |

##Installation:
This add-on should be installed on both search heads and indexers. 

On Universal Forwarders monitoring PowerShell transcript log files, the following configuration is recommended.
```#Monitor PowerShell transcript logs
[monitor://C:\pstrans\*\*.txt]
sourcetype = powershell:transcript
index = powershell
disabled = 0
multiline_event_extra_waittime = true
time_before_close = 300```

Additional information on the configuration of this app is available here: https://www.hurricanelabs.com/splunk-tutorials/XXXXXXX

# Support:
- This app is developer supported by Hurricane Labs. 
- You can send any inquiries / comments / bugs to splunk-app@hurricanelabs.com
- Response should be relatively fast if emails are sent between 9am-5pm (Eastern)


# Updates
0.1.0
-Initial release
