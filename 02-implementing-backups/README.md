# 02 – Implementing Backups

This document records the complete procedure and screenshots for preparing backup media, performing a one-time backup, restoring a deleted file, and using Volume Shadow Copy Service (VSS) to roll a file back to a previous version. All branding/lab references have been removed; everything else is preserved.

### Objectives
- Prepare backup media; run a one-time backup; prove restore.  
- Enable VSS and roll a file back to a previous version. 

### Tools & Techniques
DiskPart, Windows Server Backup (Backup Once/Recover), Explorer/Recycle Bin, VSS/Previous Versions, `wmic shadowcopy`

### Sample Outputs
diskpart> format fs=ntfs label="Backup01" quick
Windows Server Backup: Backup completed successfully.
wmic: Method execution successful.
Recovery: Operation completed successfully.

### Key Takeaways
- Backups recover **deletions/loss**; VSS rolls back **changes**.  
- Always test restore (delete → recover → verify).  
- Keep backup targets separate; follow 3-2-1 where possible.

## Table of Contents
- [1. Prepare Backup Media](#1-prepare-backup-media)
- [2. One-Time Backup with Windows Server Backup](#2-one-time-backup-with-windows-server-backup)
- [3. Delete a File and Restore from Backup](#3-delete-a-file-and-restore-from-backup)
- [4. Enable VSS and Restore Previous Version](#4-enable-vss-and-restore-previous-version)

---

# 1. Prepare Backup Media 

**Environment**  
Windows Server 2019 client VM (`PC10`). You will prepare a second disk as **Backup01 (F:)**, take a one-time backup of `C:\Users`, test a restore of a deleted file, then enable VSS and roll a file back.

**Open an elevated Command Prompt**  
Use the Start menu or search to open **Command Prompt** as Administrator.
  
![Open Command Prompt](./images/0.png)

**Initialize and format Disk 1 as Backup01 (F:)**  
Use DiskPart to bring Disk 1 online, clear read-only, create a primary partition, quick-format NTFS with label **Backup01**, and assign the letter **F**.
```text
diskpart
select disk 1
online disk
attribute disk clear readonly
create partition primary
format fs=ntfs label="Backup01" quick
assign letter=f
exit
```
![DiskPart – prepare F:](./images/1.png)

**Stage sample files for backup/restore testing**  
Create three files under Jaime’s Documents and one in Public for later restore and VSS tests.
```cmd
copy C:\SETUP\NetworkMiner\Fingerprints\oui.txt C:\Users\Jaime\Documents\document01.txt
copy C:\SETUP\NetworkMiner\Fingerprints\oui.txt C:\Users\Jaime\Documents\document02.txt
copy C:\SETUP\NetworkMiner\Fingerprints\oui.txt C:\Users\Jaime\Documents\document03.txt
copy C:\SETUP\NetworkMiner\Fingerprints\oui.txt C:\Users\Public\document04.txt
```
![Copy sample files](./images/2.png)

**Open Windows Server Backup**  
Launch Windows Server Backup (via Start or `wbadmin.msc`).  
![Open Windows Server Backup](./images/3.png)

**Start “Backup Once” (Different options → Custom)**  
Select **Local Backup** → **Backup Once** → **Different options** → **Custom** to choose specific items.  
![Backup Once wizard](./images/4.png)

**Select items to include**  
Choose **Add Items** → expand **Local Disk (C:)** → check **Users** → **OK** → **Next**.  
![Destination: F:](./images/5.png)

**Choose destination and run backup**  
Select **Local drives**, choose **Backup01 (F:)** as the destination, then start the backup.  
![Backup completed](./images/6.png)

**Delete `document01.txt` and empty Recycle Bin**  
Use File Explorer: delete `C:\Users\Jaime\Documents\document01.txt`, then **Empty Recycle Bin** to ensure only the backup can restore it.  
![Delete and empty Recycle Bin](./images/7.png)

**Start Recovery wizard**  
In Windows Server Backup, select **Recover** → **This server (PC10)** → choose the backup date/time you just created.  
![Recover wizard start](./images/8.png)

**Select Recovery Type and target item**  
Choose **Files and folders** → browse `PC10 → Local Disk (C:) → Users → jaime → Documents`, select **document01.txt**.  
![Select Items to Recover](./images/9.png)

**Restore to original location and verify**  
Pick **Original location** → **Recover**. After success, open `document01.txt` to confirm the content.  
![Recovery complete](./images/10.png)

**Enable Shadow Copies (VSS) on C:**  
Disk Management → right-click **(C:)** → **Properties** → **Shadow Copies** tab → select **C:\** → **Enable** → confirm.  
![Enable VSS on C:](./images/11.png)

**Force a VSS snapshot from an elevated Command Prompt**  
This ensures there’s a shadow copy to restore from.

wmic shadowcopy call create Volume=c:
![Force VSS shadow](./images/12.png)

**Modify `document04.txt` (simulate change)**  
Open `C:\Users\Public\document04.txt`, add a new first line such as `changed`, and save the file.  
![Edit document04.txt](./images/13.png)

**Restore a previous version of `document04.txt`**  
Right-click the file → **Restore previous versions** → choose a suitable version → **Restore** → verify that the `changed` line is gone.  
![Restore previous version](./images/14.png)

---

# 2. One-Time Backup with Windows Server Backup

The following screenshots show the **wizard pages** and **status views** for the one-time backup you configured above. They confirm the choices and the successful run.

**Open backup console and confirm “Local Backup” view**  
![15](./images/15.png)

**Launch “Backup Once” from the Actions pane**  
![16](./images/16.png)

**Use Different options (custom backup)**  
![17](./images/17.png)

**Select Custom configuration**  
![18](./images/18.png)

**Add Items → choose `Users` on C:**  
![19](./images/19.png)

**Items selected (Users)**  
![20](./images/20.png)

**Destination type: Local drives**  
![21](./images/21.png)

**Choose Backup01 (F:) as destination**  
![22](./images/22.png)

**Confirmation page (pre-run)**  
![23](./images/23.png)

**Backup progress**  
![24](./images/24.png)

**Operation details**  
![25](./images/25.png)

**Completion message**  
![26](./images/26.png)

**Close wizard and return to console**  
![27](./images/27.png)

**Local Backup dashboard after successful run**  
![28](./images/28.png)

---

# 3. Delete a File and Restore from Backup

These screenshots document the **recovery workflow** for `document01.txt` after it was deleted and the Recycle Bin emptied.

**Open the Recovery wizard**  
![29](./images/29.png)

**Choose “This server (PC10)”**  
![30](./images/30.png)

**Select the appropriate backup date/time**  
![31](./images/31.png)

**Recovery type: Files and folders**  
![32](./images/32.png)

**Browse to `C:\Users\Jaime\Documents`**  
![33](./images/33.png)

**Select `document01.txt`**  
![34](./images/34.png)

**Specify recovery options (Original location)**  
![35](./images/35.png)

**Confirm the recovery**  
![36](./images/36.png)

**Recovery progress**  
![37](./images/37.png)

**Recovery completed successfully**  
![38](./images/38.png)

**Open and verify restored file**  
![39](./images/39.png)

**Return to backup console post-recovery**  
![40](./images/40.png)

---

# 4. Enable VSS and Restore Previous Version

This sequence shows **enabling Shadow Copies**, **forcing a snapshot**, **editing `document04.txt`**, and **restoring a previous version**.

**Open Disk Management**  
![41](./images/41.png)

**C: Properties → Shadow Copies tab**  
![42](./images/42.png)

**Enable Shadow Copies for C:**  
![43](./images/43.png)

**Confirm enablement**  
![44](./images/44.png)

**Shadow Copies now shows a next run time / snapshot entries**  
![45](./images/45.png)

**Open elevated Command Prompt**  
![46](./images/46.png)

**Force a VSS snapshot**  
Create a snapshot explicitly so there’s a restorable version even if automatic timing hasn’t run.

```cmd
wmic shadowcopy call create Volume=c:\\
```
![47](./images/47.png)

**Open `C:\Users\Public\document04.txt`**  
![48](./images/48.png)

**Add a new first line (e.g., `changed`) and save**  
![49](./images/49.png)

**Right-click the file → Restore previous versions**  
![50](./images/50.png)

**Select the most recent pre-change version**  
![51](./images/51.png)

**Confirm restoration**  
![52](./images/52.png)

**Success notification**  
![53](./images/53.png)

**Open the file; confirm the change is rolled back**  
![54](./images/54.png)

**Close editors and return to File Explorer**  
![55](./images/55.png)

**Final state after restoration**  
![56](./images/56.png)

> Note: VSS retains previous versions but is **not a full backup**. Deleted files are not recoverable via VSS alone; use backups or File History for that.
