
# This fork is mainly the same as xyou365/AutoRclone, with additional features that I've added.
Features added on top of xyou365/AutoRclone:
- filters (follows the syntax from [rclone's filtering](https://rclone.org/filtering/))
- move and sync with `--move` `--sync` respectively (in addition to copy from xyou365/AutoRclone which is now execute with `--copy` flag)

BELOW IS THE README.MD FROM xyou365/AutoRclone, BUT I'VE MADE SOME EDIT FOR READABILITY
---------------------------------
# AutoRclone: rclone copy/move/sync (automatically) with service accounts (still in the beta stage)
Many thanks for [rclone](https://rclone.org/) and [folderclone](https://github.com/Spazzlo/folderclone).

- [x] create service accounts using script
- [x] add massive service accounts into rclone config file
- [x] add massive service accounts into groups for your organization
- [x] automatically switch accounts when rclone copy/move/sync 
- [x] Windows system is supported

Step 1. Copy code to your VPS or local machine
---------------------------------
_Before everything, install **python3** and rclone._

Then clone this repository with:
```
sudo git clone https://github.com/xyou365/AutoRclone && cd AutoRclone && sudo pip3 install -r requirements.txt
```
If you're using Windows, just download from the repository as a ZIP file and extract it somewhere.

Then run the below command in the extracted folder:
```
python3 -m pip install -r requirements.txt
```

Step 2. Generate service accounts.
---------------------------------

[What is service account](https://cloud.google.com/iam/docs/service-accounts) 
[How to use service account in rclone](https://rclone.org/drive/#service-account-support)
**Warning:** abuse of this feature is not the aim of autorclone and we do **NOT** recommend that you make a lot of projects, just one project and 100 service accounts will allow you plenty of use, it's also possible that usage abuse might get your projects banned by google. 


Enable the Drive API in [Python Quickstart](https://developers.google.com/drive/api/v3/quickstart/python) and save the file `credentials.json` into project directory.

```
Note: 1 service account can copy around 750gb a day, 1 project makes 100 service accounts so thats 75tb a day. 
```

To create 100 service accounts **for existing projects** (you probably want this, because you've already have at least one project from the Python Quickstart above), run:
`python3 gen_sa_accounts.py --quick-setup -1`.
Note that this will overwrite the existing service accounts.

To create **one new project** and then 100 new service accounts **for every project in your account** (including the one that has just been created), run:  
 `python3 gen_sa_accounts.py --quick-setup 1`
 _*replace "1" with the number of projects you want_

To create **one new project** and then 100 new service accounts **for this new project only**, run:  
`python3 gen_sa_accounts.py --quick-setup 1 --new-only` 
_*replace "1" with the number of projects you want_

Note: You might have to verify that the process is complete from the command output. Most probably you'll have to copy some URLs from the output to your browser to manually activate some services for this to work.

After it is finished, there will be many json files in the folder named `accounts`. 


Step 3. Add service accounts to Google Groups (Optional but recommended for hassle free long term use)
---------------------------------
We use Google Groups to manager our service accounts considering the  
[Official limits to the members of Team Drive](https://support.google.com/a/answer/7338880?hl=en) (Limit for individuals and groups directly added as members: 600).

#### For GSuite Admin (If you're a GSuite admin)
1. Turn on the Directory API following the [official steps](https://developers.google.com/admin-sdk/directory/v1/quickstart/python) (save the generated json file to folder `credentials`).

2. Create group for your organization [in the Admin console](https://support.google.com/a/answer/33343?hl=en). After creating a group, you will have an email address for the group, for example `sa@yourdomain.com`.

3. Run `python3 add_to_google_group.py -g sa@yourdomain.com`

_For meaning of above flags, please run `python3 add_to_google_group.py -h`_

#### For normal user (If you're not a GSuite admin)

Create [Google Group](https://groups.google.com/) then add the service accounts as members by hand.
Limit is 10 at a time, 100 a day but if you read our warning and notes above, you would have only 1 project and hence easily in your range. 

Step 4. Add service accounts or Google Groups into Team Drive
---------------------------------
_If you do not use Team Drive, just skip._
**Warning:** It is **NOT** recommended to use service accounts to clone "to" folders that are not in teamdrives, SA work best for teamdrives. 

If you have already created Google Groups (**Step 2**) to manager your service accounts, add the group address `sa@yourdomain.com` or `sa@googlegroups.com` to your source Team Drive (tdsrc) and destination Team Drive (tddst). 
 
Otherwise, add service accounts directly into Team Drive.
Enable the Drive API in [Python Quickstart](https://developers.google.com/drive/api/v3/quickstart/python) and save the `credentials.json` into project root path if you have not done it in **Step 2**.
- Add service accounts into your source Team Drive:
`python3 add_to_team_drive.py -d SharedTeamDriveSrcID`
- Add service accounts into your destination Team Drive:
`python3 add_to_team_drive.py -d SharedTeamDriveDstID`

Step 5. Start your task
---------------------------------
Let us copy hundreds of TB resource using service accounts. 
**Note**: Sarcasm, over abuse of this (regardless of what cloning script you use) may get you noticed by google, we recommend you dont be a glutton and clone what is important instead of downloading entire wikipedia.

#### For server side copy
- [x] publicly shared folder to Team Drive
- [x] Team Drive to Team Drive
- [ ] publicly shared folder to publicly shared folder (with write privilege)
- [ ] Team Drive to publicly shared folder
```
python3 rclone_sa_magic.py -s SourceID -d DestinationID -dp DestinationPathName -b 1 -e 600
```
- _For meaning of above flags, please run python3 rclone_sa_magic.py -h_

- _Add `--disable_list_r` if `rclone` [cannot read all contents of public shared folder](https://forum.rclone.org/t/rclone-cannot-see-all-files-folder-in-public-shared-folder/12351)._

- _Please make sure the Rclone can read your source and destination directory. Check it using `rclone size`:_

1. ```rclone --config rclone.conf size --disable ListR src001:```

2. ```rclone --config rclone.conf size --disable ListR dst001:```

#### For local to Google Drive (needs some testing)
- [x] local to Team Drive
- [ ] local to private folder
- [ ] private folder to any (think service accounts cannot do anything about private folder)
```
python3 rclone_sa_magic.py -sp YourLocalPath -d DestinationID -dp DestinationPathName -b 1 -e 600
```

* Run command `tail -f log_rclone.txt` to see what happens in details (linux only).

![](AutoRclone.jpg)

Also let's talk about this project in Telegram Group [AutoRclone](https://t.me/AutoRclone)

[Blog（中文）](Blog (中文) 
https://gsuitems.com/index.php/archives/13/) | [Google Drive Group](https://t.me/google_drive) | [Google Drive Channel](https://t.me/gdurl)  



