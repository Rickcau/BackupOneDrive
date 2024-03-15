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
      sudo mount -t nfs 192.168.42.14:/mnt/BigDisks/OneDriveNFS ~/nfsmount 
   ~~~
