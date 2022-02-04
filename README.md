### remoteSync bash script
Utility to synchronise a local Laravel project to a a remote location using rsync.
Following the files synchronisation the script will proceed by optimising the remote Laravel instance.

#### Installation
- Copy and paste the script into a file named remoteSync.sh
- Add execution permissions by running the command `chmod +x remoteSync.sh`

#### Usage
```
% ./remoteSync.sh user@remote
Source folder name > ./myProject
Remote folder name > myRemoteProject

Environment 
1 Development
2 Production
Choose an option 1-2 > 2

The source folder does not exist!

```
Run the script as ./remoteSync.sh mycredentials@myRemote
The script starts by validating the existence of the local and remote folders.  
It proceeds with the synchronization and displays the statistics.  
Then it:
- Fixes the ownership and the permissions of the remote folder and its content.
- Runs composer dump-autoload -o

In production mode, the following is executed:
- Runs php artisan run optimize
- Replaces in .env the APP_DEBUG and APP_ENV variables.
- Restarts php8.0-fpm
