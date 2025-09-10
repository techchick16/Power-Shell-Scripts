# Power-Shell-Scripts
Contains Power Shell scripts

# Extract-CertMetadata.ps1 file 
Extracts metadata from X.509 certificate files in a directory or single file.

Rename the certificate file(s) (usually sent to you as a .txt file) to end with .pem, .cer, or .crt. 
Create a new NotePad file and copy and paste the code into it. 
Rename the C:\Path\To\Your\Script to a valid directory on your computer and where you plan to save out your Notepad file to.
Rename the C:\Path\To\Your\Script\MetaDataOutput.csv to a valid directory on your computer you want to save the meta data information out to in a .csv (comma delimited file).  
You need to have Excel installed on your computer to open the .csv file.
Save the file under C:\Path\To\Your\Script\Extract-CertMetadata.ps1.
Click on Windows button plus X at the same time and select Windows PowerShell.
This will take you to the command prompt, like an MS Dos command prompt. 
Type Cd "C:\Path\To\Your\Script" to go to the directory you put the Extract-CertMetadata.ps1 file in.
Type this command at the command prompt to run it: .\Extract-CertMetadata.ps1
This will produce a .csv file that should contain one row for each certificate found in the C:\Path\To\Your\Script directory.
