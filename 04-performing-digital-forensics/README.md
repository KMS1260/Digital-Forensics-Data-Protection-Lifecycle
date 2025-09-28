# 04 –Performing Digital Forensics
This guide documents the complete workflow for performing post-incident forensic analysis across three scenarios: discovering a hidden partition, recovering deleted files, and performing file carving. It is written to be uploaded directly to GitHub and references screenshots stored in a sibling folder.

> **Platform**: Kali Linux VM (host name: **KALI**), working as **root** with password **Password**.

---

## Objectives

- Analyze a forensic drive image to **discover a hidden partition**.
- Recover **deleted files** from a forensic drive image.
- Perform **file carving** to extract inaccessible files from a damaged image.


## Table of Contents
- [1) Hidden Partition Discovery](#1-hidden-partition-discovery)
- [2) Recover Deleted Files (NTFS)](#2-recover-deleted-files-ntfs)
- [3) File Carving from a Damaged Image](#3-file-carving-from-a-damaged-image)
- [Appendix: Tips for Working with Evidence](#appendix-tips-for-working-with-evidence)

---

## Environment & Resources

- VM: **KALI** (Kali Linux), sign in as **root** / **Pa$$w0rd**.
- Media image: **Student-Resources-L25.ISO** containing test images.
- Tools used: `fdisk`, `testdisk`, `fiwalk`, `fsstat`, `mmls`, `fls`, `istat`, `losetup`, `mount`, `tsk_recover`, `xdg-open`.
- Reference repository (read-only in a real case): Digital Forensics Tool Testing (DFTT) images.


---

## 1) Hidden Partition Discovery

Forensic evaluation of a drive image

In this lab We will take over an investigation by the IRT (incident response team) of a recent intrusion has stalled because of the lack of evidence. As a security professional, we would perform our own evaluation of the known targetted system. At this point, the original system has already been reconstituted. However, a raw image of that system's drive is available for analysis. In this exercise, we will attempt to find additional evidence of the intrusion by inspecting this drive image.

From a local computer, view the Extended Partition Test page from http://dftt.sourceforge.net/. 

On local computer, open another tab in current browser or open a new browser. 

In your local browser's address bar on the new tab, enter http://dftt.sourceforge.net/. 

Select Extended Partition Test.

![](./Performing%20Digital%20Forensics/0.png)

Look over the contents of the Extended DOS Partition Test page. You may need to return to this page later.

![](./Performing%20Digital%20Forensics/1.jpg)

Leave the local browser tab open that is focused on dftt.sourceforge.net

The site dftt.sourceforge.net is the Digital Forensics Tool Testing image repository. This site contains 14 forensic images which can be used to test forensic analysis tools. These images are also useful in practicing and developing skills in using forensic tools and techniques before working on actual case collected evidence. This is just one of many similar repositories of forensic image testing files.

Connect to A KALI virtual machine and, sign in as root.

Make sure you downloaded file called 1-extend-part-zip from http://dftt.sourceforge.net/. (**Extended Partition Test**)

other files to download (Basic Data Carving Test #1 "**11-carve-fat.zip**") - (NTFS Undelete (and leap year) Test "**7-undel-ntfs.zip**")

Open a Terminal window by selecting the Terminal Emulator from the Kali Linux toolbar

The Terminal window should already be elevated to use root privileges. Maximize the Terminal window.

- Enter the following command to view the contents of the DVD Drive: 
```bash 
ls /media/cdrom0/
```
> ⚠️ **Note** for you it could be in a folder or in the downloads file so replace the /cdrom0 to whichever folder you have put the downloaded zip files we going to use in this project. also if you don't know how to create a DVD Drive in linux VM watch this Youtube video https://www.youtube.com/watch?v=muJpfNgtQ_U or search in youtube how to create dvd drive in linux vm

![](./Performing%20Digital%20Forensics/3.png)

- Enter the following command to copy the forensic test image files to the /root/Downloads directory:

**Note** you can skip this step if you already have those files in the Downloads file

```bash
cp /media/cdrom0/* /root/Downloads/
```
![](./Performing%20Digital%20Forensics/4.png)

- Enter **Unzip the 1-extend-part.zip** file in the /root/Downloads directory, change into the resulting directory, then view a long list of its contents.
```bash
 cd /root/Downloads.
```

- Enter to view the contents of the directory
```bash
 ls -l
```

- Enter **unzip 1-extend-part.zip** to extract the contents of the zip archive into its default subfolders.
```bash
unzip 1-extend-part.zip
```

- Enter **cd 1-extend-part** to enter the file
```bash
cd 1-extend-part
```

- Enter **ls -l** to view the contents of the directory
```bash
ls -l
```

![](./Performing%20Digital%20Forensics/5.png)

The ext-part-test-2.dd is the forensic test image file that we will be evaluating in this exercise.

Now that we have access to the drive image from the compromised system, we will start the analysis.

Using fdisk to display the partition details of the drive image.

- Enter the following to display the partition details of the drive image.
```bash
fdisk -l ext-part-test-2.dd
```
![](./Performing%20Digital%20Forensics/6.png)

In the presentation of partition details from fdisk, we notice that the 4th line is listed with a Type of Extended. Therefore, this is not a formattable volume, but it is the 4th primary partition which has been converted into an extended partition and then further subdivided.

we also notice the size of the extended partition. It is 155232 sectors. The next two items in the fdisk list are logical drives/partitions of size 52353 and 50337 sectors.

MBR-based drives are limited to 4 primary partitions. However, the 4th primary partition can be converted into an extended partition. This conversion allows the 4th partition to be subdivided into numerous additional formattable volumes called logical drives (which are often called partitions). This extended partition concept enables drives to support more than four (4) formatted volumes for use by an operating system.

**Quick questions**  

How many formattable partitions are displayed by fdisk for this drive image?
- <details>
  <summary>Answer</summary>
  
  ```
  5
  ```
</details>

What are possible explanations of the unaccounted for sectors from the extended partition? 
- <details>
  <summary>Answer</summary>
  
  ```
  A hidden logical drive
  ```

  ```
  Unused space not allocated to a logical drive
  ```
    
</details>


we decide to use a different drive image analysis tool to see if we can discover more information. we will use testdisk to display the partition details of the drive image.

- Enter the following to display the partition details of the drive image.
```bash
testdisk -l ext-part-test-2.dd
```

![](./Performing%20Digital%20Forensics/7.png)

We notice that this tool shows a total of 7 numbered entries. One more than what fdisk was able to discover.

> ⚠️**Note** If the output of testdisk is not well organzied by columns, enter reset, then run the testdiskcommand again.


Use fiwalk to analyze the drive image.

- Enter the following to view the output of this drive image analysis tool through the less viewer.
```bash
fiwalk ext-part-test-2.dd | less
```

![](./Performing%20Digital%20Forensics/8.png)

When using the less file viewing utility, press the **spacebar** to view the next page. You can return to a previous page using **b** or scroll one line **up** or **down** utilizing the arrow keys. When you are finished looking over the results, type **q** to exit the less viewer.

This forensic test image file is crafted with a text file in each partition named after the partition. We can view these filenames for each partition by scrolling through the output and watching for the change in the partition number.

Use **fsstat** to extract more information about the partitions, especially the hidden partition we have discovered. We will also need to use **mmls** (a TSK tool) to determine the offset of the hidden partition.

- Enter the following:
```bash
fsstat ext-part-test-2.dd
```

![](./Performing%20Digital%20Forensics/9.png)

You should see an error stating that the file system type was not determined. You remember that the fiwalk command displayed the file system type as "fat16".

- Enter the following:
```bash
fsstat -f fat16 ext-part-test-2.dd
```

![](./Performing%20Digital%20Forensics/10.png)

You should see an error displayed, indicating that the magic value is invalid. This indicates that the initial partition table is corrupted and cannot be automatically interpreted by fsstat.

- Enter the following to display the layout details of the partitions.
```bash
mmls ext-part-test-2.dd
```

![](./Performing%20Digital%20Forensics/11.png)

The Start column displays the sector offset for each drive division. Also, notice that the hidden extended volume is line 14 in this tool's output display.

- Using the sector offset for the 6th partition, enter:
```bash
fsstat -f fat16 ext-part-test-2.dd -o 262143
```

![](./Performing%20Digital%20Forensics/12.png)

This tool displays more information than what we had access to previously, but nothing is very interesting at this point.

The fsstat (file system statistics) utility is one of the many tools from The Sleuth Kit (TSK). It can be used to display details of the file system(s) on a drive image. The -f parameter sets the file system type. There are several options (display a list by entering fsstat -f list), including auto-detection options for fat and ext systems. If you don't know the file system, you may need to experiment.

Use the TSK tool fls to pull file information from the hidden partition.

- Use another tool from the TSK to pull file information, enter:
```bash
fls -f fat16 ext-part-test-2.dd -o 262143
```

![](./Performing%20Digital%20Forensics/13.png)

You should see a file named second-3.txt and other items preceded by a dollar sign. These $named items are system-hidden volume management components.


Use another TSK tool, istat, to pull inode information from the hidden partition.

- Enter the following:
```bash
istat -f fat16 ext-part-test-2.dd -o 262143 1
```

![](./Performing%20Digital%20Forensics/14.png)

Notice there is an additional digit at the end of this command, which is the reference for the inode for istat to retrieve.

We should see an error claiming the Metadata address is too small for image.

- Try the next inode by entering:
```bash
istat -f fat16 ext-part-test-2.dd -o 262143 2
```

![](./Performing%20Digital%20Forensics/15.png)

This reveals information about the root directory.

- Try the next inode by entering:
```bash
istat -f fat16 ext-part-test-2.dd -o 262143 3
```

![](./Performing%20Digital%20Forensics/16.png)

This inode is for the file in the root of the drive named second-3.txt.

- Try the next inode by entering:
```bash
istat -f fat16 ext-part-test-2.dd -o 262143 4
```

![](./Performing%20Digital%20Forensics/17.png)

This inode is for another file in the root of the drive named SECOND-3.txt. But notice this file has timestamps while the other inode entries do not. This could have been a file present on the drive prior to it being converted into a hidden partition, and thus it retained its original timestamps. However, this file was deleted or otherwise removed from this partition and was somehow corrupted, and is unable to be restored.

- Try the next inode by entering:
```bash
istat -f fat16 ext-part-test-2.dd -o 262143 5
```

![](./Performing%20Digital%20Forensics/18.png)

This result of an invalid metadata address indicates that we have viewed all of the inode details available.

An inode is a file system metadata structure that is used to store and organize file object information, such as file size, owner user, group IDs, permissions, and timestamps.
In this forensic image test file of a drive image, this file is a confirmation that we are viewing the contents of the hidden partition, which is the 3rd logical drive in the extended partition. With 3 primaries, this hidden partition is the 6th formattable volume from this drive image.

This is a very small drive with only enough content to prove the concept of a hidden partition. A real drive from a production system, even a hidden partition created by an attacker, would likely contain a significant number of inodes.

Mount the hidden partition to /mnt/p6 and view the contents.

we want to mount the hidden partition to perform more evaluation of the contents of that volume. 

we need to create a virtual device from the drive image file. 

- Enter the following:
```bash
losetup --partscan --find --show ext-part-test-2.dd
```
![](./Performing%20Digital%20Forensics/19.png)

The output of this command should be **/dev/loop0**. This indicates the virtual device has been created.

- Enter **mkdir /mnt/p6** to create a directory in /mnt to mount the hidden volume to.
```bash
mkdir /mnt/p6
```

- Enter mount **/dev/loop0p7 /mnt/p6** to mount the hidden partition.
```bash
mount /dev/loop0p7 /mnt/p6
```

![](./Performing%20Digital%20Forensics/20.png)

The use of **/dev/loop0p7** is to reference the 7th partition table item, which is the 6th partition. Remember that the 4th partition table item is the extended partition, which was originally the 4th primary partition. Its position must still be taken into account when performing mounting.

- Enter **cd /mnt/p6** to change into the mounted volume. 
```bash
cd /mnt/p6
```

- Enter **ls -l** to view the contents of the mounted volume.
```bash
ls -l
```

![](./Performing%20Digital%20Forensics/21.png)

You should see the file **second-3.txt** but not **SECOND-3.txt**. The SECOND-3.txt file may have been deleted and overwritten, as it is not recoverable from this forensic image test file.

Leave the Terminal window open.

From this point, in a real-world investigation, we would likely find interesting files and metadata on the hidden partition that we have recovered access to. Unfortunately, this forensic image test file does not contain anything other than a single filename to confirm the discovery of and recovery of access to the 3rd logical volume from the extended partition.

---

## 2) Recover Deleted Files (NTFS)

Another investigation by the IRT of a different security incident has also resulted in a lack of evidence. In this instance, there is a drive image of the storage device where it is believed evidence was destroyed by the perpetrator. As a security professional, you would like to perform your own evaluation of the drive image. In this exercise, we will attempt to recover evidence from the drive image.

From your local computer, view the NTFS Undelete (and leap year) Test #1 page from http://dftt.sourceforge.net/.

On your local computer, open another tab in your current browser or open a new browser.

In your local browser's address bar on the new tab, enter http://dftt.sourceforge.net/.

Select NTFS Undelete (and leap year) Test #1.

![](./Performing%20Digital%20Forensics/22.png)

Look over the contents of the NTFS Undelete (and leap year) Test #1 page. You may need to return to this page later.

Leave the local browser tab open that is focused on dftt.sourceforge.net.

Connect to the KALI virtual machine and, sign in as root.

Unzip the **7-undel-ntfs.zip** file in the **/root/Downloads** directory, change into the resulting directory, then view a list of its contents.

The Terminal window should still be open.

- Enter **cd /root/Downloads**. 
```bash
cd /root/Downloads
```

- Enter **ls -l** to view the contents of the directory. 
```bash
ls -l
```

- Enter **unzip 7-undel-ntfs.zip** to extract the contents of the zip archive into its default subfolders. 
```bash
unzip 7-undel-ntfs.zip
```

- Enter **cd 7-undel-ntfs**.
```bash
cd 7-undel-ntfs
```

- Enter **ls -l** to view the contents of the directory.
```bash
ls -l
```

![](./Performing%20Digital%20Forensics/23.png)

The 7-ntfs-undel.dd is the forensic test image file that we will be evaluating in this project.

The filename of the zip is 7-undel-ntfs.zip, the name of the extract directory is 7-undel-ntfs, but the name of the drive image file is 7-ntfs-undel.dd.

Now that we have access to the drive image from the suspect's system, we will start the analysis.

- Mount **7-ntfs-undel.dd** to **/mnt/temp7**, then view the contents of the image. 

- Enter **mkdir /mnt/temp7** to create a mount point. 
```bash
mkdir /mnt/temp7
```

- Enter **mount 7-ntfs-undel.dd /mnt/temp7** to mount the image to the mount point. 
```bash
 mount 7-ntfs-undel.dd /mnt/temp7
```

- Enter **ls -l /mnt/temp7** to view the contents of the drive image.
```bash
ls -l /mnt/temp7
```

![](./Performing%20Digital%20Forensics/24.png)

The results will show a System Volume Information directory but nothing else. This causes you to think that the suspect may have deleted files that you may be able to recover.

Use **tsk_recover** (a TSK tool) to attempt to automatically recover the deleted files from this drive image into a folder named output. Then, view a list of the recovered files.

- Enter **tsk_recover 7-ntfs-undel.dd output** to attempt to automatically recover the deleted files from this drive image into a folder named output.
```bash
tsk_recover 7-ntfs-undel.dd output
```

![](./Performing%20Digital%20Forensics/25.png)

You should see the statement: Files Recovered: 8.


- Enter **ls -l output** to view the recovered file information.
```bash
ls -l output
```

![](./Performing%20Digital%20Forensics/26.png)

Discover the recovered filenames that are not located in the root of the output recovery directory.

- Enter **ls -l output/dir1** to view the contents of the recovered dir1 directory. 
```bash
ls -l output/dir1
```

- Enter **ls -l output/dir1/dir2** to view the contents of the recovered dir1/dir2 directory.
```bash
ls -l output/dir1/dir2
```
![](./Performing%20Digital%20Forensics/27.png)

**Quick question** What are the names of the recovered files that are in sub-directories?
- <details>
  <summary>Answer</summary>
  
  ```
  mult2.dat
  ```
  ```
  frag3.dat
  ```
</details>

The undeleted files are now visible and accessible for further investigation. However, the files from this forensic test disk image do not have any visible or useful content to explore.

Leave the Terminal window open.

From this point, in a real-world investigation, you would analyse the timestamps and contents of the recovered files. This may lead to other information which could direct the investigation further.

---

## 3) File Carving from a Damaged Image

The IRT informs you that a second drive's image is available from the suspect's system, where data destruction is suspected. As a security professional, you are eager to attempt data recovery on this drive image. In this exercise, we will work through the tedious process of gaining access to a damaged drive image in hopes of recovering files through file carving. File carving is the forensic process of recovering access to files that are otherwise inaccessible due to corruption, partial data loss (especially headers), deletion, or partition structure damage.

From your local computer, view the Basic Data Carving Test #1 page from http://dftt.sourceforge.net/.

On your local computer, open another tab in your current browser or open a new browser.

Select Basic Data Carving Test #1.

![](./Performing%20Digital%20Forensics/28.png)

Look over the contents of the Basic Data Carving Test #1 page.

Connect to the KALI virtual machine and,  sign in as root.

Unzip the **11-carve-fat.zip** file in the **/root/Downloads** directory, change into the resulting directory, then view a list of its contents.

The Terminal window should still be open. 

- Enter **cd /root/Downloads**.
```bash
 cd /root/Downloads
```

- Enter **ls -l** to view the contents of the directory. 
```bash
ls -l
```

- Enter **unzip 11-carve-fat.zip** to extract the contents of the zip archive into its default subfolders. 
```bash
unzip 11-carve-fat.zip
```

- Enter **cd 11-carve-fat**.
```bash
cd 11-carve-fat
```

- Enter **ls -l** to view the contents of the directory.
```bash
ls -l
```

![](./Performing%20Digital%20Forensics/29.png)

The **11-carve-fat.dd** is the forensic test image file that you will be evaluating in this exercise. 

Now that we have access to the drive image from the suspect's system, we will start the analysis.

 Mount **11-carve-fat.dd** to **/mnt/temp11**.

- Enter **mkdir /mnt/temp11** to create a mount point.
  ```bash
  mkdir /mnt/temp11
  ```
- Enter **mount 11-carve-fat.dd /mnt/temp11** to mount the image to the mount point.
  ```bash
  mount 11-carve-fat.dd /mnt/temp11
  ```

![](./Performing%20Digital%20Forensics/30.png)

This should result in a mounting error. This indicates that something is corrupted in the image and cannot be mounted for direct file system analysis.

 Use **fdisk** to display the partition details of the drive image. 
 
- Enter **fdisk -l 11-carve-fat.dd** to display the partition details of the drive image.
  ```bash
  fdisk -l 11-carve-fat.dd
  ```

![](./Performing%20Digital%20Forensics/31.png)

This should result in a partial presentation of information. Notice that the Disk identifier: value is all zeros, and there is no partition table displayed. Something is wrong with this drive image.

Use **fiwalk** to perform a drive image analysis.

- Enter **fiwalk 11-carve-fat.dd** to perform a drive image analysis.
  ```bash
  fiwalk 11-carve-fat.dd
  ```

![](./Performing%20Digital%20Forensics/32.png)

Notice the results include numerous errors, most of which state Possible encryption detected.

Use **fsstat** to display file system statistics of the drive image. You may need to use **mmls** to determine the partition(s) offset(s).

- Enter **fsstat 11-carve-fat.dd** to display file system statistics of the drive image.
```bash
fsstat 11-carve-fat.dd
```

![](./Performing%20Digital%20Forensics/33.png)

This tool should also show an error of Possible encryption detected. However, previously, you needed to define the file system type for this tool to work. Try guessing the file system type. Start with fat16.

- Enter **fsstat -f fat16 11-carve-fat.dd** to display file system statistics of the drive image.
  ```bash
  fsstat -f fat16 11-carve-fat.dd
  ```

![](./Performing%20Digital%20Forensics/34.png)

The error presented now is that there is an Invalid magic value. This indicates that the initial partition table is corrupted and cannot be automatically interpreted by fsstat.

- Enter **mmls 11-carve-fat.dd** to display the layout details of the partitions.
  ```bash
  mmls 11-carve-fat.dd
  ```

![](./Performing%20Digital%20Forensics/35.png)

This tool will provide no results. Therefore, you cannot determine the partition offset values to use with fsstat, fls, or istat.

we will be unable to obtain information about the drive image with fsstat.

As demonstrated in this case, it is often necessary to try numerous tools and techniques to recover evidence and access files that have been corrupted, deleted, or otherwise purposefully destroyed. Many forensic tools use different data recovery techniques, so don't give up until you have exhausted all of your options.

Use **testdisk** to use file carving to recover files from the drive image. Store the recovered files in the **output** sub-folder (which you need to create first).

- Enter **mkdir output** to create an output folder.
  ```bash
  mkdir output
  ```
- Enter **testdisk 11-carve-fat.dd* to attempt to open the drive image in testdisk's interactive mode.
  ```bash
  testdisk 11-carve-fat.dd
  ```
The TestDisk tool should open and present the 11-carve-fat.dd file for processing.

- Notice that at the bottom of the interface the **[Proceed ]** option is highlighted. Press Enter on your keyboard to select this option.

![](./Performing%20Digital%20Forensics/36.png)


- Use your keyboard's down arrow key to select **[None ]** as the partition table type, then press **Enter** on your keyboard.

![](./Performing%20Digital%20Forensics/37.png)

- This will result in a display of an Unknown partition, which is already highlighted. Press **Enter** on your keyboard.

![](./Performing%20Digital%20Forensics/38.png)

- Use your keyboard's down arrow key to select **FAT16** as the partition type, then press **Enter** on your keyboard.

![](./Performing%20Digital%20Forensics/39.png)

If you don't know the partition type, you may need to guess until to find a working option.

- You are returned to the previous screen, which now has the partition labeled as "FAT16". Use your keyboard's right arrow key to highlight **[Undelete]** at the bottom of the screen, then press **Enter** on your keyboard.

![](./Performing%20Digital%20Forensics/40.png)

The resulting page will have a message of: No file found, filesystems may be damaged.

![](./Performing%20Digital%20Forensics/41.png)

Type **q** to exit the Undelete function and return to the previous screen.

- Use your keyboard's left arrow key to highlight **[ Boot ]** at the bottom of the screen, then press **Enter** on your keyboard.

![](./Performing%20Digital%20Forensics/42.png)

- The **[Rebuild BS]** at the bottom of the screen is already highlighted. Press **Enter** on your keyboard.

![](./Performing%20Digital%20Forensics/43.png)

This will attempt to rebuild the boot sector. This could be the cause of the read and access problem of this drive image.

The rebuild function will provide a result that includes the statement "Extrapolated boot sector and current boot sector are different.". The **[ List ]** option at the bottom of the screen is already highlighted. Press **Enter** on your keyboard.

![](./Performing%20Digital%20Forensics/44.png)
![](./Performing%20Digital%20Forensics/45.png)

Notice that a list of files is presented. The rebuild boot sector operation was able to restore access to the drive image. However, this repair is only in memory and has not changed the original .dd file. we need to extract all recoverable files from the in-memory repaired image.

- Type **a** to select all files, then type **C** to copy the selected files. 

> ⚠️ Be sure to type a capital C. Otherwise, the lowercase version will only copy a single file

![](./Performing%20Digital%20Forensics/46.png)

A output directory selection screen is shown. Use your keyboard's down arrow key to highlight the **output** directory, press **Enter** on your keyboard to enter the output directory, then type **C** to copy the files to the current directory.

![](./Performing%20Digital%20Forensics/47.png)
![](./Performing%20Digital%20Forensics/48.png)

This operation should result in a success message above the file list of "Copy done! 15 OK, 0 failed".

Type **CTRL+C** to break/exit TestDisk. 

View the contents of the recovered files. 

- Enter **cd output**, then **ls -l** to view the contents of the recovered files.
  ```bash
  cd output
  ```
  ```bash
  ls -l
  ```
  
![](./Performing%20Digital%20Forensics/49.png)

If the directory listing does not display properly, enter reset, then try the command again.

![](./Performing%20Digital%20Forensics/50.png)

You should see the 15 recovered files.

- Enter **xdg-open haxor2.jpg** to open this graphics file to view the picture.
  ```bash
   xdg-open haxor2.jpg
  ```

![](./Performing%20Digital%20Forensics/51.jpg)

Use your keyboard's up and down arrows to view the other graphics from this directory. When you are finished viewing these pictures, type **CTRL+C** to exit the graphics viewer.

- Enter **xdg-open lin_1.2.pdf** to open this PDF document to view its contents.
  ```bash
  xdg-open lin_1.2.pdf
  ```

![](./Performing%20Digital%20Forensics/52.png)

When you are finished looking at this PDF, type **CTRL+C** to exit.

You can open several of these files using xdg-open, including surf.mov. You can unzip wword60t.zip to view its contents.

**Quick question** 
Which recovered image file includes cats?
```text
paul.jpg
pumpkin.jpg
shark.jpg
haxor2.jpg
```

- <details>
  <summary>Answer</summary>
  
  ```
  pumpkin.jpg
  ```
</details>

If you have followed those steps congrats have successfully performed file carving to extract the files that were destroyed by the perpetrator. These evidence files should be shared with the IRT and other interested groups so they can continue to perform their investigations.

---

## Appendix: Tips for Working with Evidence

- **Never** alter original evidence. Work on **verified copies**; keep originals read-only and hash-verified (e.g., `sha256sum`).
- Document **offsets**, **timestamps**, and **tool versions** in your notes.
- When mounting raw images, prefer loop devices and avoid writes to source images.
- On export, retain the **directory structure** and include a short **README** with tool commands used.

---

