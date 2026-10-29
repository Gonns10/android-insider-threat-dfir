# Android Insider Threat: DFIR Proof of Concept

Welcome to my first Digital Forensics and Incident Response (DFIR) portfolio project. 

This repository contains a practical **Proof of Concept (PoC)** simulating a corporate insider threat investigation. The objective was to perform a logical acquisition of an Android device (OPPO A72) and analyze the artifacts to detect unauthorized data exfiltration, bypassing automated tools to focus on manual analysis and problem-solving.

## Tools & Technologies Used
*   **Android Debug Bridge (ADB):** for logical acquisition (`adb pull` and `adb backup -nosystem`).
*   **Autopsy (Sleuth Kit):** for data ingestion, timeline analysis, and artifact recovery.
*   **WinRAR:** for manual backup extraction and decompression.

## Project Structure
*    **[Read the Full Forensic Report](Forensic_Report.md)**: detailed breakdown of the methodology, technical challenges (troubleshooting ADB/ZLIB errors), timeline analysis, and final evidence discovery.
*    **img/**: sanitized screenshots of the Autopsy analysis environment.

## Ethical & Privacy Disclaimer
*This project was conducted strictly for educational and portfolio purposes on a personally owned test device. All data, images, and logs have been generalized and completely sanitized (redacted) to ensure personal privacy. No real corporate or sensitive personal data is exposed in this repository.*
