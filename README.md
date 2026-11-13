# Hurricane Labs Add-on for Windows PowerShell Transcript

## Support
- Splunk Enterprise 9.x and 10.x (tested on 9.4.3, 10.4.1 and 10.6.1)
- Windows PowerShell 5.1 and PowerShell 7.x transcripts written on Windows, including translated headers. Tested with English, German and French (5.1 and 7.6) and Japanese, Traditional Chinese and Russian (7.6); Windows PowerShell 5.1 in other languages is untested.

## How This App Works
This app provides knowledge objects for working with Windows PowerShell transcript logs.  In addition to field extractions, a number of event types are included to support threat hunting use cases. 

Each event is one PowerShell session: the transcript header (user, machine, launch command line, process ID, PowerShell version) plus the commands and their output (`std_out`). Events are timestamped with the session's `Start time`.

The add-on also sets the field names of the CIM Endpoint data model's Processes dataset: `dest`, `user`, `process`, `process_id`, `process_path`, `process_name` and `action`. It does not tag events into the data model by default; see [Adding the events to the CIM Endpoint data model](#adding-the-events-to-the-cim-endpoint-data-model).

## About Hurricane Labs Add-on for Windows PowerShell Transcript

| Author | Tom Kopchak, Hurricane Labs |
| --- | --- |
| App Version | 0.2.0 |
| Vendor Products | Windows PowerShell 5.1, PowerShell 7 |
| Has index-time operations | true - line breaking, timestamps, dropping transcript end blocks |
| Create an index | false |
| Implements summarization | false |

## Installation
Install this add-on on search heads, and on whichever tier parses the data: indexers, or heavy forwarders if they sit in front of the indexers. 

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

Transcript timestamps are the endpoint's local time with no time zone. Splunk applies the time zone of the forwarder that reads the file, which is right when each computer's own forwarder reads its own transcripts, as above.

If computers write their transcripts to one network share and a single forwarder reads them all, transcripts from computers in other time zones will be dated hours off. Set the host from the file name, then set the time zone per host in props.conf on the parsing tier:
```
# inputs.conf on the forwarder reading the share
# (file names are PowerShell_transcript.<COMPUTER>.<random>.<timestamp>.txt)
[monitor://\\fileserver\pstrans\*\*.txt]
sourcetype = powershell:transcript
host_regex = PowerShell_transcript\.([^.\\]+)\.

# props.conf on indexers or heavy forwarders, one stanza per group of hosts
[host::NYC-*]
TZ = America/New_York
```

### Turning on transcription for PowerShell 7
Windows PowerShell 5.1 and PowerShell 7 read separate transcription policies:

- Windows PowerShell 5.1: `HKLM\Software\Policies\Microsoft\Windows\PowerShell\Transcription`
- PowerShell 7: `HKLM\Software\Policies\Microsoft\PowerShellCore\Transcription`

Setting only the Windows PowerShell policy does not turn on transcription in PowerShell 7. Either set the PowerShell 7 policy too, or set it with `UseWindowsPowerShellPolicySetting = 1` so it follows the Windows PowerShell one. Both take `EnableTranscripting`, `OutputDirectory` and `EnableInvocationHeader` (the per-command "Command start time" blocks).

Additional information on the configuration of this app is available here: https://www.hurricanelabs.com/blog/how-to-use-powershell-transcription-logs-in-splunk

### Adding the events to the CIM Endpoint data model
To have transcripts count as process launches in the Endpoint data model (Processes dataset), for Splunk Enterprise Security and Splunk Security Content searches:

1. Add the tags in `local/tags.conf` of this add-on on the search heads:
   ```
   [eventtype=powershell_transcript_session]
   process = enabled
   report = enabled
   ```
   Tag `powershell_transcript_session`, not `powershell_event`. It only matches events with a full header. When a session goes idle longer than the forwarder's `time_before_close`, the rest of it arrives as a separate event with no header, and that event is not a process launch.
2. Add your transcript index to the CIM add-on's `cim_Endpoint_indexes` macro.

If the same hosts also send Sysmon or Windows event 4688 process data, each PowerShell launch will then appear twice in the data model.

PowerShell 7 names its host `pwsh.dll` in the transcript; the add-on reports `process_name` as `pwsh.exe`, the process that is actually running, and keeps the original in `process_path` and `process`.

### Limitations
- A launch command line that spans several lines is cut at the first line in `process` and `app`.
- PowerShell 7 on Linux or macOS also reports `pwsh.dll`, which becomes `process_name = pwsh.exe` there too.
- A session that goes idle longer than `time_before_close` is split. The later part has no header fields. It is dated by its first line shaped like `label: yyyyMMddHHmmss`, which is usually a "Command start time" but can be command output; with no such line, Splunk falls back to its own time.
- Command output that reproduces a whole transcript header, for example `Get-Content` of another transcript, splits the session there. The commands after it get the printed header's user, computer, process and start time. Anyone running commands in a session can do this on purpose, so header fields and `_time` are not tamper-proof.
- A session is one event, and Splunk cuts events at 256 KB (`TRUNCATE` in props.conf) and discards the rest. Commands after a long stretch of output, such as a large directory listing, are not indexed. Splunk logs each cut, with the file in `data_source`: `index=_internal sourcetype=splunkd "Truncating line" "data_sourcetype=powershell:transcript"`.
- A `-UseMinimalHeader` session in which no command was run is dropped: it has the same shape as a transcript end block.
- `powershell_encoded_command` doesn't flag an encoded command of only one or two characters (such as `ls`): its base64 is too short to tell from other text.
- `powershell_command` checks that PowerShell is named earlier on the line, not that PowerShell is the program being run. So `python -c` typed at a prompt in a `WindowsPowerShell` folder still matches, and so does a script's own `-c` argument after `-File`.
- The threat-hunting eventtypes search the whole session text, so a command that is only printed in output (or a matching file name) can match too.

# Support:
- This app is developer supported by Hurricane Labs. 
- You can send any inquiries / comments / bugs to splunk-app@hurricanelabs.com
- Response should be relatively fast if emails are sent between 9am-5pm (Eastern)


# Updates
0.2.0
- Fixed event timestamps: events now get the session's `Start time`. Before, they got the transcript file's last-modified time.
- Added PowerShell 7 support: sessions are split and end blocks dropped for the "PowerShell transcript" banner as well as "Windows PowerShell transcript", in CRLF or LF files.
- Fixed `user`, `domain`, `username`, `runas_user` and `runas_domain` for names with dots, spaces or `$`, and for `NT AUTHORITY\SYSTEM`.
- `std_out` now starts after the header for both PowerShell versions.
- Added support for transcripts in other languages: Windows PowerShell 5.1 on a non-English Windows and PowerShell 7.6.6 and later translate the banner and header labels. Parsing now goes by the header's layout instead of English words.
- Fixed `computer`/`dest` taking several lines when the machine name is blank.
- The header-less tail of a split session is now dated by its first `label: yyyyMMddHHmmss` line, usually a "Command start time", instead of the file time.
- Threat-hunting eventtype changes:
  - `powershell_encoded_command` now matches every abbreviation of `-EncodedCommand` (`-e`, `-encod` and so on) and the `/` forms such as `/e` and `/enc`, but only when the payload that follows is shaped like base64 of UTF-16LE text, the only thing `-EncodedCommand` accepts. That also covers `Start-Process -ArgumentList '-enc','<base64>'` and renamed or obfuscated launches. `-Encoding` abbreviations (`Out-File -enc utf8NoBOM`) and other tools' `-e` (`docker run -e HOSTNAME`, `git grep -e`, `sed -e`) no longer match. It adds an `encoded_command_switch` field.
  - `powershell_command` now needs the switch on a line that runs `powershell` or `pwsh`. It no longer matches other parameters starting with "com" (`-ComputerName`, `-ComObject`), cmdlet names ending in "-Command" (`Invoke-Command`, `Get-Command`), or other tools' `-c` (`python -c`, `bash -c`) unless that line also names PowerShell (see Limitations). It adds a `command_switch` field.
  - `powershell_nslookup_txt` now also matches `-type=txt` and `-querytype=txt`.
  - `powershell_registry` now also matches `Get-ItemProperty -Path HK...` and `Set-Location HK...`.
- Added CIM field names: `dest`, `process`, `process_id`, `process_path`, `process_name`, `action`, and the `powershell_transcript_session` eventtype for opting in to the CIM Endpoint data model.
- Added the `[id]` stanza to app.conf and `sc_admin` write access for Splunk Cloud.

Upgrade notes:
- The timestamp, line-breaking and end-block changes apply only to data indexed after the upgrade. PowerShell 7 and translated sessions indexed earlier stay merged.
- The eventtype changes apply at search time, so they also change what saved searches and alerts built on these eventtypes or their tags match in existing data.
- Events are now dated when the session started, which can be well before Splunk receives the transcript (after the session ends, or after it goes quiet for `time_before_close`). A scheduled search that looks back only a few minutes by event time can miss long sessions. Search by arrival time instead, for example `_index_earliest=-20m`.
- Transcript times carry no time zone, so a single forwarder reading a shared folder from computers in several time zones needs a per-host `TZ` (see Installation). The old file-modified-time timestamps didn't have this problem.

0.1.3
-Fixed username and runas extractions, thanks Martin Mueller for contribution

0.1.2
-Fixed PSVersion and std_out extractions, thanks Jamie Wells for contribution

0.1.1
-Fixed typos in props.conf and tags.conf

0.1.0
-Initial release
