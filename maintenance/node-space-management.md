# Node Space Management

## **1. Freeing up space**

* C**heck available space**

```text
df -h
```

* **Check the size of your log file**

```text
du -h $(docker inspect -f '{{.LogPath}}' otnode)
```

* **Delete the log file**

```text
truncate -s 0 $(docker inspect -f '{{.LogPath}}' otnode)
```

* Clean old versions and old backups

```text
docker exec -it otnode bash
```

```text
cd /ot-node/
```

```text
ls
```

Then delete all old versions except the latest one by adjusting the below command accordingly \(for example the below command will remove version 4.1.2, as my node already has version 4.1.3\):

```text
rm -r 4.1.2
```

![](../.gitbook/assets/image%20%2826%29.png)

Then enter the backup directory and delete any old backups you might have there which you don’t need:

```text
cd backup
```

```text
rm -r backupfolder
```

Once you finish, type exit:

```text
exit
```

* **Delete all logs on the server with FTP to free up even more space**

Further, you can navigate to the following directory on your server \(login using any FTP client, and search for all \*.log files within that directory and delete them.

/var/lib/docker/overlay2/

**Important: Be careful to delete logs only within that directory, or you can break the VPS.** Do this at your own discretion

Further, if you are running an old node since 2019, check the below folder \(var/lib/docker/volumes\) and browse the folders inside to check if you have similar 2.0.xx folders as in the screenshot below. If you do, you can safely delete all 2.0.xx folders.

![](../.gitbook/assets/image%20%283%29.png)

* Clean cache

```text
sudo du -sh /var/cache/apt
```

```text
sudo apt clean
```

* Clean server logs \(3d = will delete any journals older than 3 days, can be a different number, i.e. 1d, or 1h\)

```text
journalctl --disk-usage
```

```text
sudo journalctl --vacuum-time=3d
```

## **2. Adding additional space to the server**

By some mistake or just number of jobs growing you can find your VPS is lacking disk space. Disk space can be checked with the command **df -h** which will display disk partitions and their usage:

![](../.gitbook/assets/image%20%2828%29.png)

You should save the path displayed in the **overlay** row into Notepad as you will need it later on.

On Hetzner and Digital Ocean you can add additional space to your VPS by adding a Volume space. This operation requires restarting your server, but is possible to do so even without restarting by following the guide below \(In case you are not sure if VPS will reboot, i.e. you deleted some important file on the server\).

* Login to your Hetzner Console at [https://console.hetzner.cloud/](https://console.hetzner.cloud/)
* Select your **Project \(default is called Default\)**
* Click on the server which is having problems. This will bring up server settings.
* Select **Volumes:**

![](../.gitbook/assets/image%20%2813%29.png)

* Click on **Create Volume** button. You can leave all the settings at default values just make sure the volume size is enough to accommodate your backup. _NOTE: the name of the volume can be different and depends on the region your server is deployed in._ 

![](../.gitbook/assets/image%20%281%29.png)

* After volume is created click on the 3 dots on the right side of volume you have just created. In the menu choose **Show Configuration**. This will bring up the commands you must use to make the space on the new volume available inside your VPS.

![](../.gitbook/assets/image%20%2832%29.png)

_NOTE: This are just example commands. Just MUST use the commands that are displayed on your Hetzner Console._

* Copy, paste and run the first command in the terminal of your VPS.
* The second and third commands must be changed. You will need the path of your docker container which you saved at the beginning of the guide. If you did not you must run **df -h** and save the path in the row **overlay. It should look something like this:** /var/lib/docker/overlay2/5b0720e1a89e407a48aa3be9112f6f531ec47d7ec5662a7dafcc98506968af8d/diff
* Now onto the next two commands. You will use the value you got from the **df -h** on your node. **IMPORTANT: add the missing directories which are not present in the** _**df -h**_ **output.**

```text
mkdir /var/lib/docker/overlay2/XXXXXXXXXXXXXXXXXX/diff/ot-node/backup
```

```text
mount -o discard,defaults /dev/disk/by-id/scsi-0HC_Volume_9591049 /var/lib/docker/overlay2/xxxxxxxxxxxxxxxxxxx/diff/ot-node/backup
```

After this commands your new volume should be available in tempbackup folder inside docker container.

* Next you run the command for backup to AWS S3 storage but changing the default backup directory \(note the changed directory name in bold\):

```text
docker exec otnode node scripts/backup-upload-aws.js --config=/ot-node/.origintrail_noderc --configDir=/ot-node/data --backupDirectory=/ot-node/backup --AWSAccessKeyId=YOUR_AWS_ACCESS_KEY_ID --AWSSecretAccessKey=YOUR_AWS_SECRET_ACCESS_KEY --AWSBucketName=YOUR_AWS_BUCKET_NAME
```
