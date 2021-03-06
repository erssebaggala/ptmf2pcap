## Converting Huawei PTMF trace files to Wireshark PCAP files
[![Build Status](https://travis-ci.org/fran-ovia/ptmf2pcap.svg?branch=master)](https://travis-ci.org/fran-ovia/ptmf2pcap)

1. It is a simple application to convert Huawei PTMF files to Wireshark PCAP files.
2. No special requirement! Just Java Runtime Environment installed in your PC.
3. There are many types of PTMF files, but just some of them (those carrying SIP, RTP, Diameter or DNS) are supported by ptmf2pcap so far (I implemented ptmf2pcap because of the trace files generated by Huawei SBCs I had to deal with).

## How to use ptmf2pcap?

First o all, go to this repository's releases section (https://github.com/fran-ovia/ptmf2pcap/releases) and download ptmf2pcap.jar and (optionally) ptmf2pcap.bat too.

There are two ways of using ptmf2pcap:

**A. Just double-clicking on ptmf2pcap.jar file:**

The app will then look for PTMF files in the same directory and convert them to PCAP files (same name but with ".pcap" extension).
A command-line-like window will be opened to summarize the results of the process.

**B. Invoking ptmf2pcap.bat script from command line:**

Options are explained in the help:

```
#> ptmf2pcap.bat -h
ptmf2pcap.v0.9.4.build20160819:

Usage 1 (converts the input PTMF file into the output PCAP file):

    ptmf2pcap -f <input_file> <output_file>

Usage 2 (converts the PTMF files from the input directory into PCAP files in the
 output directory):

    ptmf2pcap -d <input_directory> <output_directory>
```

Note that you might need to edit ptmf2pcap.bat script to customize the locations of your java.exe executable (if it's not already included in your PATH variable) and ptmf2pcap.jar file (if you don't want to store it in the same directory as the ptmf2pcap.bat script).

Note that I'm not including an .sh script equivalent to the .bat script, since implementing it is so straightforward and would probably need customization anyway.

## Why to convert a PTMF file to a PCAP file?

1. So once we have the network traffic in PCAP format we can then analyse it with Wireshark instead of doing it with Huawei Trace Viewer. Wireshark provides many more features, and the filters it implements are more flexible and agile to use.
2. So we can then merge network traces gathered at Huawei nodes with network traces gathered at some other vendor's equipment (and therefore not stored in PTMF format).
3. So we can share network traces with other people that do not have Huawei's PTMF viewer installed in their PCs (for instance, when you are troubleshooting a Huawei-to-Ericsson interworking scenario and you are working together with an Ericsson engineer).

## PTMF file types supported by ptmf2pcap

| PTMF File Type | Comments on the PTMF to PCAP frame conversion process |
| -------------  |-------------|
| SIP            | Transport protocol (UDP, TCP or SCTP) is worked out by analysing the SIP message (Via header) |
| Diameter       | Transport protocol is assumed to be SCTP |
| UserInterface  | Frames with content different from SIP, DNS, Diameter or RTP are treated as system log frames and embedded into Syslog messages with source and destination IPs set to 0.0.0.0. Comments for SIP and Diameter PTMF File Types apply to UserInterface frames with SIP and Diameter content too |
| IP             | Transparent conversion (since the IP PTMF frames contain the whole Ethernet packet) |

The information in next section is very useful to understad the comments from the table above.

## Some explanations on the PTMF files and ptmf2pcap application:

1. First of all, note that I have found no specification of the PTMF file format. All my knowledge on such format comes from analysing the byte content of some PTMF files I got from Huawei SBCs and comparing it with the information displayed when opening the same PTMF files with a Huawei PTMF viewer.
2. A PTMF file contains a file header and a list of PTMF frames
3. The information in the file header is useful to determine the PTMF file type. So far ptmf2pcap recognizes only a limited amount of file types (unrecognized file types are not converted to PCAP).
4. The format of the PTMF frames is not always the same, but it depends on the PTMF file type
5. Although PTMF files are generally considered equivalent to a PCAP file but in Huawei format, it is easy to believe that they contain the same type of information, but this is not always true, and it is better understood with an example for the case of network traces corresponding to SBC signaling (SIP, DNS, Diameter):
   * When such signaling is stored as frames in a PCAP file, each frame contains an Ethernet Layer, An IP layer, a transport layer (UDP, TCP or SCTP) and an application layer (SIP, DNS or Diameter).
   * When such signaling is stored as frames in a PTMF file of type SIP, Diameter or UserInterface, each frame does not contain all the network layers, but just the application layer (SIP, DNS or Diameter) and some fields of the other layers (IPs and ports). In other words, there is a part of the information from each network packet that is not stored in the PTMF file. This is probably the reason why Huawei does not offer (as far as I know) PTMF to PCAP conversion for these file types, and was the reason for this ptmf2pcap being created.
   * The frames in PTMF files can also contain system information (such as the system resource processing each signaling/media message), and PTMF files of UserInterface type can even include some frames that contain system logs only.
6. Thus, when ptmf2pcap converts a PTMF frame to a PCAP frame:
   * It might happen that not all the information from Ethernet, IP and transport layers can be extracted from the PTMF file (the IPs and ports can always be extracted, but that is not always the case for other fields such as MAC addreses and checksums, which would then need to be filled with default values). When doing the latter, ptmf2pcap fills with zeroes so it gets noticeable (so when generated PCAP files contain frames with 00:00:00:00:00:00 MAC addresses and 0x0000 checksums you can guess the reason why).
   * It might also happen that some PTMF frames do not contain any valid network message, but just system logs, so the ptmf2pcap just embeds such information into Syslog network frames in the output PCAP file (those Syslog network frames have source and destination IPs set to 0.0.0.0).
