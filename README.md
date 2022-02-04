### remoteSync bash script
Utility to sync a local Laravel project to a a remote location using rsync.
Following the file synchronisation the script will optimise the remote instance.

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

Finally it restart php-fpm.

#### The script

```
#!/bin/bash
credentials=root@staging
lastUsedSourceFolder=cfemhbcu.cfrev.com
remoteHtmlPath=/var/www/html
red='\033[0;31m'
green='\033[0;32m'
# functions
replaceCredentials() {
  arg1=$1
   sed -i '' -e '1,/credentials=.*/ s/credentials=.*/credentials='$arg1'/1' $0
}
replaceLastUsedSourcePath() {
  arg1=$1
  sed -i '' -e '1,/lastUsedSourceFolder=.*/ s/lastUsedSourceFolder=.*/lastUsedSourceFolder='$arg1'/1' $0
}
printMenu() {
  printf "\nEnvironment \n1 Development\n2 Production\nChoose an option 1-2 "
}
# remote
if [ "$1" != "" ]
then
  replaceCredentials $1
  credentials=$1
fi
# input
if [ "${lastUsedSourceFolder}" = "" ]
then
  read -p "Source folder path > " sourceFolder
  replaceLastUsedSourcePath ${sourceFolder}
else
  read -p "Source folder path [${lastUsedSourceFolder}] > " sourceFolder
  sourceFolder=${sourceFolder:-${lastUsedSourceFolder}}
fi
# replace the last used source path with the new input
if [ "${sourceFolder}" != "${lastUsedSourceFolder}" ]
then
  replaceLastUsedSourcePath ${sourceFolder}
fi
# remote folder
read -p "Remote folder name [${sourceFolder}] > " remoteFolder
remoteFolder=${remoteFolder:-${sourceFolder}}
# set environment
printMenu
read -p "[2]> " environment
environment=${environment:-2}

# strip input from trailing slashes
strippedSourcePath=$(echo ${sourceFolder} | sed 's:/*$::')
strippedRemoteFolder=$(echo ${remoteFolder} | sed 's:/*$::')

remotePath=${remoteHtmlPath}/${strippedRemoteFolder}/

# check if source folder exist
[[ -d ${strippedSourcePath} ]] || {
  echo -e "\n${red}The source folder does not exist!\n"
  exit 1
}
# check if remote folder exist
(ssh ${credentials} "test -e ${remotePath}") || {
  echo -e "\n${red}The remote folder does not exist!\n"
  exit 1
}

# rsync command
rsync -a \
      --stats \
      --delete \
      --exclude '.idea' \
      --exclude '.env' \
      --exclude '.env.example' \
      --exclude '.git' \
      --exclude 'composer.lock' \
      --exclude 'node_modules' \
      --exclude 'vendor' \
      --exclude 'storage' \
      ${strippedSourcePath}/ \
      ${credentials}:${remotePath}

echo -e "\n${green}Syncing completed.\n"
sleep 1

# fix remote files ownership and permissions
# run composer dump-autoload
ssh ${credentials} "chown -R www-data:www-data ${remotePath} \
      && chmod -R 755 ${remotePath} \
      && /usr/bin/php /usr/bin/composer -n dump-autoload -o --working-dir="${remotePath}""

if [ ${environment} == "2" ]
then
echo -e "\n${green}Production environment\n"
# run php artisan optimize
# set the environment debug to false and the env to production
ssh ${credentials} "sed -i 's/APP_DEBUG=true/APP_DEBUG=false/g' ${remotePath}.env \
      && sed -i 's/APP_ENV=local/APP_ENV=production/g' ${remotePath}.env \
      && /usr/bin/php ${remotePath}artisan optimize \
      && service php8.0-fpm restart"
fi

echo -e "\n${green}Optimization completed.\n"
echo -e "Bye"

```
