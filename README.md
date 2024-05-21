# Hurricane Labs Add-on for Windows PowerShell Transcript

## Support
- Splunk 7.3, 8.x, 9.x

## How This App Works
This app provides knowledge objects for working with Windows PowerShell transcript logs.  In addition to field extractions, a number of event types are included to support threat hunting use cases. 

## About Hurricane Labs Add-on for Windows PowerShell Transcript

| Author | Tom Kopchak, Hurricane Labs |
| --- | --- |
| App Version | 0.1.3 |
| Vendor Products | Windows PowerShell |
| Has index-time operations | true - line breaking |
| Create an index | false |
| Implements summarization | false |

##Installation:
This add-on should be installed on both search heads and indexers. 

On Universal Forwarders monitoring PowerShell transcript log files, the following configuration is recommended.
```
#Monitor PowerShell transcript logs
[monitor://C:\pstrans\*\*.txt]
sourcetype = powershell:transcript
index = powershell
disabled = 0
multiline_event_extra_waittime = true
time_before_close = 300
```

Additional information on the configuration of this app is available here: www.hurricanelabs.com/splunk-tutorials/splunk-tutorial-powershell-transcription-logging

# Support:
- This app is developer supported by Hurricane Labs. 
- You can send any inquiries / comments / bugs to splunk-app@hurricanelabs.com
- Response should be relatively fast if emails are sent between 9am-5pm (Eastern)


# Updates
0.1.3
-Fixed username and runas extractions, thanks Martin Mueller for contribution

0.1.2
-Fixed PSVersion and std_out extractions, thanks Jamie Wells for contribution

0.1.1
-Fixed typos in props.conf and tags.conf

0.1.0
-Initial release
