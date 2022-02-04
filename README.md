### remoteSync bash script
Utility to synchronise a local Laravel project to a a remote location using rsync.
Following the files synchronisation the script will proceed by optimising the remote Laravel instance.

#### Installation
- Copy the script `remoteSync.sh` into your projects' folder. 
- Attribute execution permissions to it, by running the command `chmod +x remoteSync.sh`

#### Usage
```
% ./remoteSync.sh mycredentials@myRemote
Source folder name > myLocalProjectFolder
Remote folder name [myLocalProjectFolder]> myRemoteProjectFolder

Environment 
1 Development
2 Production
Choose an option 1-2 [2]> 2
```

The script starts by validating the existence of the local and remote folders.  
It proceeds with the synchronization then displays the statistics.  
Then it:
- Recursively fixes the ownership and the permissions of the remote folder and its content.
- Runs composer dump-autoload -o

Furthermore in production mode, the script will:
- Runs php artisan run optimize
- Replaces in .env the APP_DEBUG value to false and APP_ENV value to production.
- Restarts php8.0-fpm.  
  
Happy synching!
