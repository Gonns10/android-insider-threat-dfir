# Insider Threat Investigation: 

Android Logical Acquisition & Analysis

## 1. Executive Summary

This project simulates a corporate digital forensics investigation concerning a suspected Insider Threat (data exfiltration by a departing employee). The objective was to acquire, process, and analyze the contents of a corporate-issued Android device using open-source tools (ADB, Python, Autopsy) to identify potential intellectual property theft, while documenting strict adherence to digital forensics methodologies and technical limitations.

## 2. Case Background & Rules of Engagement

* **Scenario:** an employee suspected of transferring confidential corporate files to personal storage resigned. 
* **Device Status:** following the company's HR offboarding protocol and MDM policies, the device was surrendered in an unlocked state (PIN provided by the user).
* **Objective:** extract communications, downloaded documents, and metadata to establish a timeline of unauthorized data exfiltration.

## 3. Device Identification
* **Model:** OPPO A72 (CPH2067)
* **OS:** Android (ColorOS)
* **Encryption State:** File-Based Encryption (FBE) active.

## 4. Acquisition Methodology
A physical extraction was evaluated but deliberately discarded. Unlocking the bootloader on the OPPO CPH2067 to achieve root access would trigger an automatic Factory Reset, permanently destroying the evidence. Consequently, a **Logical Acquisition** via Android Debug Bridge (ADB) was executed to preserve data integrity.

[Link to the ADB Github page](https://github.com/nelenkov/android-backup-extractor)


![700](Pasted%20image%2020261026193146.png)

We can see in the picture how we needed to accept the access with the phone unlocked.
### 4.1. Public Storage Extraction

Targeted extraction of the emulated SD card to capture downloads, media, and external app storage:

```bash
adb pull /sdcard/ evidence_oppo_sdcard/
````

![](Pasted%20image%2020261026201457.png)

We can see in the picture how we had two different problems: 

- **Long file path limitations:** Windows threw `Not a directory` errors due to path length limits in cached files (e.g., `com.heytap.music`). 
	_Mitigation:_ These cache files were verified as non-essential to the investigation and were successfully bypassed to complete the extraction.
- **MAC times alteration (Procedural deviation):** the `-a` flag was omitted during the `adb pull` execution, which modified the external OS timestamps (Creation/Modification dates) of the extracted files to the time of acquisition. 
	_Mitigation:_ the temporal analysis was strictly pivoted to rely on internal metadata (EXIF data, document properties, and SQLite internal timestamps) which remained completely intact.
### 4.2. Private Application Data Extraction

At first, we tried the next command but we had **ZLIB compression errors** due to conflicts with protected OS processes. 

```
adb backup -all -apk -shared -system -f evidence_insider.ab
```

![](Pasted%20image%2020261026201516.png)

That's why the `-nosystem` flag was appended to exclude native bloatware, successfully extracting the critical user-installed application data (WhatsApp, Browsers, etc.) without data loss.

```
adb backup -all -apk -shared -nosystem -f evidence_insider.ab
```

We need to take into account also **ColorOS "Private Safe" Encryption:** 
OPPO's native secure folder operates with hardware-backed encryption. Since ADB logical backups explicitly ignore `allowBackup="false"` flags, contents within this vault could not be acquired without destructive brute-force methods, marking a recognized limitation in the acquisition scope.

After getting the file.ab we need to unpack it with the corresponding abe.jar we found in [the ADB github page](https://github.com/nelenkov/android-backup-extractor/releases).

![](Pasted%20image%2020261029101026.png)

```
java -jar ./abe-38fd634.jar unpack evidence_insider.ab evidence_insider.tar
```


## 5. Evidence Processing

### 5.1. Autopsy Ingestion

First, the logical folder and the `.tar` archive were ingested into Autopsy. To expand the `.tar` contents, the **Embedded File Extractor** module was executed. 

![](Pasted%20image%2020261027101242.png)

But it didn't work as expected. The `.tar` was not extracted as we can se in the LogicalFileSet1(2)  in the next picture:

![](Pasted%20image%2020261028102704.png)
(It says "clean" because it was the `.tar` without the system data as commented previously in 4.2)

So we unzipped the evidence_insider_clean.tar, got the `/apps` directory and then added it to Autopsy without any problems.

### 5.2. Autopsy Analysis

Subsequent analysis relied on the following modules:

- **Android Analyzer:** parsing SQLite databases.
	This one failed because it expects another input type and with the `/system` data that we could not get.
    
- **Picture Analyzer (EXIF):** extracting geolocation from images of potentially snapped screens.
    
- **Extension Mismatch & File Type ID:** detecting obfuscated confidential files.    
    
- **Keyword Search:** indexing strings for corporate project names.

After several hours of analysis, we get this Tree:

![](Pasted%20image%2020261029111001.png)

#### 5.2.1. Temporal Analysis & Timeline Reconstruction

Using Autopsy's Timeline feature, we mapped out the exact events leading up to the data leak. We first had to filter out a bunch of system noise—like default OS files weirdly dating back to 1970, 1980, or 2000 from before the phone synced its clock—to confirm they were just normal system quirks and not someone tampering with the timestamps. 

By switching to a logarithmic view and adjusting the slider to focus strictly on the final weeks before the employee left, we cut through years of background clutter. This zoomed-in focus made it easy to connect the dots between app usage and file modifications, clearly exposing the unauthorized data access right before the phone was handed in.

  
![](Pasted%20image%2020261029110253.png)

### 5.2.2. Evidence Analysis & CamScanner Artifacts

During the document review in Autopsy, multiple PDF files generated via **CamScanner** were identified. In an insider threat scenario, these files indicate the digital capture of physical corporate documents using the mobile device for unauthorized exfiltration, bypassing standard company channels.

![](Pasted%20image%2020261029112717.png)

### 6. Conclusion

This project served as a hands-on proof of concept (PoC) to explore mobile digital forensics methodologies and master the Autopsy platform through practical troubleshooting. Rather than relying on automated tools, working through logical acquisitions (`adb`), unpacking compressed backups, and manually parsing file types provided a deep understanding of Android's internal architecture and artifact recovery.

Emphasizing a strict ethical approach, personal privacy was fully protected by sanitizing data and focusing strictly on generalized technical indicators. Ultimately, this exercise successfully demonstrated how analytical resilience and structured investigation techniques can be applied to uncover actionable digital evidence.