# bash-scripting
add user script written in bash

                                                ftp account creation
                                    Instructions to add user account to ftp servers
                    /nas_ftp5/Customer/Troy/t<project_name> – 1GB– add to quotabyfs_version_4 H-1126m, S-11024m
                    
                          Make sure you create a ftp and sftp for “ftp1” & “ftp2”, & possibly “luna”.
                          Copy sytem files from a users home directory. 
                          “cp –Rp bin, etc,  dev, lib, lib64, usr” to home directory “.”
                          Get next “UID” from book. For luna add “ftp” to end of username.
                          Use Same id for both ftp1 and ftp2 and luna group id is “49”.
                          if it is a “ftp” account use /bin/false
                          “cp” user from passwd file and create passwd file in home directory.
                          “vi /etc/security/chroot.conf”  add “uid” and “location”.
                          Disable ftp “vi /etc/ftpusers”, to disable user login with ftp.
                          Chown directory
                          Chmod directory

                                  FTP2
                          “cp passwd from ftp1”
                          “cp shadow from ftp1”
                          “tail -1  /etc/security/chroot.conf”  add “uid” and “location”.
                          Disable ftp “tail -1  /etc/ftpusers”, to disable user login with ftp.


                          Steps to add 1 user account.
                          SYSTEM 1
                          Login with username and password 
                          run command  “useradd” username
                          open password file and edit the uid,gid comment field, home directory and shell
                          open group file  and add user to “jailed” group
                          open “/etc/security/chroot.conf” and add user name and home directory.
                          open /etc/ftpusers and add username to this file
                          chown directory
                          chmod directory
                          if secure ftp then copy system files from some random home directory 
                          SYSTEM 2
                          Login  to system 2
                          copy line from system 1 password file 
                          copy line from system 1 shadow file
                          add user to “jailed” group
                          copy line from system 1 chroot file 
                          add user to /etc/ftpusers file

                          This would normally take me about 20 minutes
                          To fix this issue I set up ssh keys from system 2 to system 1 so I could login from system 1 to system 2 with                             out a password.
                          edited the “/etc/default/useradd” file on both systems so that it would add the user to the jailed group                                 automatically without options.
                          set a directory for copying files from instead of some random user.
                          then I wrote a script to do the rest of the work “see attached”
                          This took my work down from 20 minutes to about 6 minutes for secure and 2 minutes for un-secure






#!/bin/bash 
clear
# Script to add a user to this Linux system
if [ $(id -u) -eq 0 ]; then                                     # check if user is root
        read -p "Enter User Name : " username

        while [ -z $username ]|| egrep "^$username" /etc/passwd  1>/dev/null;
        do
        echo -ne "Either user exists or you entered a blank, enter username again: ";read -e  username
        done

        echo -ne "Enter your password: ";read -s password

        while [ -z $password ];
        do
        echo -ne "\nEnter your password again: ";read  -s -e password
        done

        echo -ne "\nPlease Enter your User ID Number: ";read -ern5 uid
        while [[ ! $uid =~ ^[0-9]+$ ]]||egrep $uid /etc/passwd >/dev/null; do
        echo -ne "Please re-enter your uid positive intergers only: ";read -ern5 uid
        done

        read -p "Enter a Comment : " comment

        read -p "Enter Users Home Directory : " homedir
        while [ ! -d "$homedir" ];
        do
        echo -ne "\n$homedir Directory Not Found! Please re-enter: "; read homedir
        done        

pass=$(perl -e 'print crypt($ARGV[0], "password")' $password) # passing the password entered

echo ""
                echo "Select the type of shell you will be using"
echo""
                        # Shell selection statement
echo -e "1) Bash Shell - SFTP Secure\n" 
echo -e "2) False Shell - FTP Unsecure\n"
echo -ne "Enter choice: ";read shell;
case $shell in
1)
        shell=/bin/bash              # case statment for shell selection.
        useradd -u $uid -p $pass -c "$comment" -d $homedir -s $shell $username
        echo -e "Copying System Files ...."
        cd /nas_ftp5/Customer/Imaging/Troy/T_Skel
        cp -Rp `ls` $homedir
        echo -e "Finished Copying System Files ..."
        tail -1 /etc/passwd > $homedir/etc/passwd
        echo "$username" >> /etc/ftpusers
        ;;
2)
        shell=/bin/false
        useradd -u $uid -p $pass -c "$comment" -d $homedir -s $shell $username
        ;;
esac
        echo "Setting security on users home directory"
        chown $username:ftp $homedir         # security settings for both shells
        chmod 775 $homedir
        echo -e "$username" '\t' "$homedir" >> /etc/security/chroot.conf

        echo "Sending information to FTP2"            # sending information to failover server
        
        ssh root@ftp2 useradd -u $uid -c '"$comment"' -p $pass  -d $homedir -s $shell $username
        
        ssh root@ftp2 "echo -e '"$username"' '\t' '"$homedir"' >> /etc/security/chroot.conf"
        
        ssh root@ftp2 "echo -e '"$username"' >> /etc/ftpusers"
        
        
clear
echo -e "\n\tThis users login details is as follows: \n"

echo -e "\n\tUsername is: $username \n"

echo -e "\tPassword is: $password \n"

echo -e "\tUser's ID Number is: $uid \n"

echo -e "\tComment is: $comment \n"

echo -e "\tUsers Home Directory is: $homedir \n"

echo -e "\tUsers Shell is: $shell \f"

fi
