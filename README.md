# BackupOneDrive
Backup All My OneDrive files

## Summary
Recently, I lost access to my personal email as a result of my M365 subscription expiring and my Personal OneDrive filling up.  I was a bit frustrated by this so I started pundering the idea of how best to deal with this.  

   1. Simply make sure my M365 subscription never expires and I never exceed the 6TB that my yearly plan gives me.
   2. I could simply come up with a way to backup all my data to another location i.e. to a NAS on my Home Network!

I had recently redesigned my Home Network and at the same time I built a new server for Proxmox and TrueNAS with close to 40TB of storage so I can play with all sorts of Tech on my Home Network.  Since I can serve up NFS and SMB shares using TrueNAS, I decided to impliment a solution that allows me to backup my OneDrive to my TrueNAS.  Of course I could write some code to do this, but why do that if the problem has already been solved.  

This is when I came across an Open Source solution call [RClone](https://rclone.org/). RClone is a very powerful command line tool that can be used for all sorts of things.  You can use it to mount your OneDrive to a local folder in Linux just like using the OneDrive Client on Windows.  But, this is not what I wanted to do.  I simply want to have a tool that runs on a schedule that copies all my OneDrive data to a backup.

## What are the step for this?
First, this is super simple and since my Proxmox and TrueNAS are Linux Based I will doing this on Linux.

1. Open a Linux Terminal and run the following to install rclone

      ~~~
         sudo apt install rclone
      ~~~

2. Now, we need to create a new Remote Config for rclone to use, so run the following command:

     ~~~
        rclone config
     ~~~
     You will be presented with the following menu, press **n** and enter a name for your new remote, I entered **onedrive**.
     
     ![rclone-1](https://github.com/Rickcau/BackupOneDrive/assets/17052492/788a66f8-90d9-4f37-9a9f-ca6a3c1cbe13)

3. Next, you will be presented with a list of Cloud Services, in my case I am using OneDrive so I entered **21**
   
     ![rclone-2](https://github.com/Rickcau/BackupOneDrive/assets/17052492/92234242-fe73-44e4-a7ca-4aae52965bdd)

4. Next, you will be presented with several more questions about Microsoft App Client Id, Microsoft App Client Secret.  You should enter **n** for both and these and when you get to the **Use auto config** enter **y**.

   ![rclone-3](https://github.com/Rickcau/BackupOneDrive/assets/17052492/6e22663e-851c-49e2-b92a-f1afde9eacf2)

5. After entering **y** your default browser will open and you have to log into **OneDrive**, it will ask you to grant permissions to rclone and you need to select **yes**.

   ![rclone-4](https://github.com/Rickcau/BackupOneDrive/assets/17052492/ac9816a9-7ba6-4339-83af-f5094f98ee53)

6. Next, you have to select the account type, in my case it's **1 Onedrive Personal**.

   ![rclone-5](https://github.com/Rickcau/BackupOneDrive/assets/17052492/c1a89d8c-7fbf-4f5e-8045-f1d2f147e5d8)

7. It will ask one last time if this configuration is okay.  Press **y** if everything is fine, now you will be presented with the **Rclone** configuration menu.

   ![rclone-6](https://github.com/Rickcau/BackupOneDrive/assets/17052492/d805f0d7-d53f-4b5f-b688-91664deb803f)

   Now, you can press **q** to exit the configuration menu.  This process will result on an Rclone configuration file be created, which will be used with you execute **Rclone** commands.

8. Now, we are ready to copy the files from OneDrive to a local directory on Linux.  Open a terminal and run the following commands:

   ~~~
      mkdir ~/OneDrive
      rclone copy onedrive: ~/OneDrive --transfers 16 --fast-list
   ~~~

   You will see rclone will use the remote config you just created and it will start copying the files from OneDrive to the ~/OneDrive folder.
 
## Closing Summary
This process is super simple and you can simply create a cron job to run the **rclone copy** on a schedule.  You can also do this on a Windows OS as well.  If you do some searching you will find various .sh scripts that are much more robust, but I am not looking for complexity, I simply want my files on OneDrive to be backed up to local folder once a week. 

## Helpful commands
   
   ~~~
      apt install update
      sudo apt install nfs-common
      sudo mkdir ~/nfsmount
      sudo mount -t nfs 192.168.42.14:/mnt/BigDisks/OneDriveNFS ~/nfsmount 
   ~~~

## Important Notes when working with SAMBA (SMB) shares on Linux

### Background
I have a Truenas Server that I have various shares created, both NFS and SMB.  When creating SMB shares in Truenas and connecting from Windows it's super simple, but when connected to that same SMB from Linux it requires more work to do so and ensure you have permissions to be able to perform CRUD operations.

### Steps to perform on the Linux VM to access the SMB share from TrueNAS

1. Make sure you have CIFS and SAMBA installed on the Linux machine you are mounting the SMB.

   ~~~
       sudo apt-get update
       sudo apt-get install cifs.utils
       sudo apt-get install samba
   ~~~

2. Now, let's create an SMB credentials file which will be used with our mount command

   ~~~
      sudo nano /etc/samba/.smbcreds
   ~~~

   Now, you can do an internet search to find examples of what the format for this file should be. 

   ~~~
      username=mytestaccount
      password=mytestpassword
   ~~~

   Now that you have created the file, lets set the permissions to read-only for root.

   ~~~
      sudo chmod 400 /etc/samba/.smbcreds
      ls -l /etc/samba/.smbcreds
   ~~~

4. Now, lets create a directory so we can mount the SMB share into.

   ~~~
      sudo mkdir /mnt/test
   ~~~

5. Now, we need to find out what the uid and gid is for the user that will be accessing this SMB.  You can do that with the following command:

   ~~~
      cat /etc/passwd
   ~~~

   You will output that looks like this from the **cat** command:

   ~~~
      mytest:x:1000:1000:Mytest,,,:/home/mytest:/bin/bash
   ~~~

   The first 1000 represents the uid and the second represents the guid, you need to make note of this.  This is the user that will be access the SMB and we need to pass these values in the **mount** command
   

7. Now, we can verify that we are able to mount the SMB and create a folder to ensure we have permissions to perform CRUD operations.  We will using our .smbcreds file we created in step 2.  

**Important Note:** Pay close attention to the uid and gid parameters here.  This is the uid and gid of the person that is logged into Linux and this gives them permissions to the mount point.  If you do not do this, you will not be able to create files.
   
   ~~~
     sudo mount -t cifs -o uid=1000,gid=1000,rw,vers=3.0,credentials=/etc/samba/.smbcreds //<Your IP Address>/<ShareName> /<mount>/<test>
   ~~~

8. Now, using a File Explorer navigate to the mounted share and create a new folder to ensure you have permissions to create a file.

## Here is how you can **auto mount** the the SMB using FSTAB

1. Let's open and modify the fstab file using **nano**.

   ~~~
       sudo nano /etc/fstab
   ~~~

2. Now, using the example **mount** command from previous steps, which you have tested as working, add the following entry:

   ~~~
      //<Your IP Address>/OneDriveBackup  </your mount/point>  cifs  uid=1000,gid=1000,rw,vers=3.0,credentials=/etc/samba/.smbcreds  0  0
   ~~~

3. Now, you can test if your fstab entry is correct by running the following command, it will attempt to mount all entries in the **fstab** file.

   ~~~
      sudo mount -a
   ~~~
