# System Automation and User Management Scripts

This Assignment consists of two main scripts, System Setup Scripts and User Creation script.
Each script contains multiple part to it which ultimately automate the process of either installing scripts and making configuration scripts or making a new user with a specified name, groups they can be added to and a default shell.

These scripts will include error handling and flags which will give the user the ability to run the options that they need and benefit from.
## Part 1: Configuration Scripts
the point of these Configuration Scripts are to automate the set up of a new system which can be very time consuming. These scripts solve two main problems, 1. installing packages and 2. copying configuration files into different locations to make the software to your liking.

##### The configuration scrips include the following components:
1. A file which contains user-defined list of packages to be installed
2. A script to install those packages
3. Existing configuration files for several applications on a repository to be cloned and the script to make symbolic links from those configuration files
5. A script to call the other scripts

#### 1. Userdef-packages:
This first file contains the names of packages that will be installed in the upcoming script.
``` bash
kakoune
tmux
gawk
```
these are just packages that are put inside the file so the upcoming script can download them

Note: if you wish to change these then edit them manually with either `nvim`
#### 2. install-Userdef-script
This is the first script of this assignment which will install all the packages in Userdef-packages.



this is what the script looks like:
``` bash
#!/bin/bash

#this script takes the file Userdef-packages and installs all the packages listed inside of it and if the package is already installed then it will tell the user 

#empty array named package
package=()

#command substitution which gives us the content of Userdef-packages and stores it into readthefile, the . indicted that the Userdef-packages should not be outside the current directory
readthefile=$(cat ./Userdef-packages)

#adds the readthefile to the empty array so we can loop through it. https://ioflood.com/blog/bash-append-to-array/
package+=$readthefile


#this for loop, loops through package which holds all the packages to be installed, we iterate through it with the @ inside the square brackets then ask it to do something for us
for i in ${package[@]}; do

	#pacman -Q is a command which shows us the installed packages. we pipe it into a grep and pull out the keyword which is the iterate in this case, this is all in a command substitution which gives us result and if it returns something then the double sqaure brackets will either return true or false. If it returns True then the echo will run telling the user that the package is already installed https://www.atlantic.net/dedicated-server-hosting/list-installed-packages-with-pacman-on-arch-linux/
	if [[ $(pacman -Q | grep $i) ]]; then
		echo "$i is already installed"

	#otherwise install the package
	else
		# install the package -- no confirm means not ask for y and n since we are autoaming a package install
		pacman -S --noconfirm $i
	fi
done
```

### How to use this script:
1. make the script executable:
```bash
chmod u+x install-Userdef-script
```
3. make sure to use `sudo` which elevates your permissions
4. include the name of the script like so:
```bash
sudo ./install-Userdef-script
```

this should return a install all the packages inside the previous file

#### 3.make-symlink-script:
This script makes a directory called config-files which a git repository will be clone into, then it proceeds to make symbolic links named bin, .config, and .bashrc. If you already have these directories then the script will still run and place all the symbolic links inside the directories. with the exception of .bashrc this will replace an already existing bashrc.

Note: if you have a .bashrc file running this script will replace it

this is what the script looks like:
```bash
#!/bin/bash


#This script makes a directory called config-files and clones a repository to it then it makes symbolic links to all those directory to different places like bin .confg and .bashrc


#this is the path to the config-files inside of a list. we are using absolute path so when sudo is run then we are not upgrading to the root user and staying as the specified user, the /*/* means include everything inside the next two directory levels
fullpath=( /home/$1/config-files/*/* )

#we are using $ 1 which is a parameter and if it return empty which is what the -z indicates, then echo to the user that they have to run the script with their name and let them see an example. the exit 1 indicate terminate the script so it dosent continue and they have to run it again properly this time
if [[ -z $1 ]]; then
	echo "error: run this script with your username"
	echo "./make-symlink-script <your-username>"
exit 1
fi


# we are using pacman -Q to see the pacakges the user has installed and giving the output into the input of grep which takes a keyword from that output which is git and returns it to us, if the command substitution returns false which is what the ! indicates then echo git is needed to run this script and exit 1
if [[ ! $(pacman -Q | grep git) ]]; then
	echo "git is needed to run this script"
exit 1
fi



#an if statement that checks if config-files dir exists -d tell us if the directory exists and the rest is the path of the directory
if [[ -d /home/$1/config-files ]]; then
	#clones the git repo to the config-files dir, we are running git clone and giving it the url of the repository and specifying where we want the repository to be stored in.
	git clone https://gitlab.com/cit2420/2420-as2-starting-files.git /home/$1/config-files
else
	#make the directory when it dosent exist then run git clone in the directory
	mkdir /home/$1/config-files
	git clone https://gitlab.com/cit2420/2420-as2-starting-files.git /home/$1/config-files
	echo "config-files created and repo has been cloned to it"
fi

#this is an if statement which checks if the user does not have a bin directory, if they dont then it runs and makes then a directory by running the mkdir command and then print to the user letting them know that you made one
if [[ ! -d /home/$1/bin ]]; then
	mkdir /home/$1/bin
	echo "made a bin directory"
fi

#this does the same thing as the above if statement just with the .config directory instead
if [[ ! -d /home/$1/.config ]]; then
	mkdir /home/$1/.config
	echo "made a .config directory"
fi

#This is a for loop which loops through the path array listed at the top
for i in ${fullpath[@]}; do

	#if the path we are looping equals to this one then 
	if [[ $i == /home/$1/config-files/bin/* ]]; then

		#make a symlink which is the ln -s. -s makes the the symbolic link. the iterate is the path we are making it from and the path we are making it to is listed beside it. basename takes the lastname of the path specified and we are giving it the iterate so it takes the last part and makes a symbolic link.
		# REFERNCE: https://bbs.archlinux.org/viewtopic.php?id=42284 
		# https://www.cyberciti.biz/faq/bash-get-basename-of-filename-or-directory-name/
		ln -s $i /home/$1/bin/$(basename $i)

	elif [[ $i == /home/$1/config-files/home/* ]]; then
		#this does the same thing as the one above except for /home and we are replacing the bashrc for them which is why we use -f flag also that stands for force and will replace the current bashrc
		ln -sf $i /home/$1/.$(basename $i)

	elif [[ $i == /home/$1/config-files/config/* ]]; then
		#this does the same as the ones above
		ln -s $i /home/$1/.config/$(basename $i)
	else
		#if any other path pops up then echo an error message to them and tell them that it dosent meey any matches 
		echo "error: $i dosent meet any matches"
	fi
done
```

### How to use this script:
1. make the script executable:
```bash
chmod u+x make-symlink-script
```
3. make sure to use `sudo` which elevates your permissions
4. include the name of the script like so:
```bash
sudo ./make-symlink-script
```
5. specify your username like so:
```bash
sudo ./make-symlink-script arch
```

This script will make 2 directories if they already don't exist and 1 bashrc file.

### 3.call-the-scripts:
This script is used to run the other two scripts above this one.

what the script looks like:
```bash
#!/bin/bash


#This script is used to run the other two scripts in this directory


#EUID is effective user id and changes to the root users when sudo is run. if the command is run without sudo the uid which is not equal to 0 then echo error and exit 1
#https://stackoverflow.com/questions/18215973/how-to-check-if-running-as-root-in-a-bash-script
#https://superuser.com/questions/1696909/what-is-the-meaning-of-using-euidif-ruid-and-euid-of-a-process-is-1000-0-and-i
if [[ "$EUID" -ne 0 ]]; then
	echo "Error: run the script with sudo"
	exit1
fi
#https://kodekloud.com/blog/bash-getopts/
#while loop using getopts, getopts is a shell commands used to parse command line options and arguments, the "im:" means those are the flags which will be available to run and the colon after the m indicates that an argument is expected after the flag. the opt is a variable that the option gets assigned to
while getopts "im:" opt; do
	#this is a case statement which matches the opt to the options below
	case "${opt}" in
#This is the flag -i and when this flag is called then the install-Userdef-script will run the ./ represents the script in the same directory as the two other scripts should not be in a different directory
	i)
		./install-Userdef-script
		#these double ;; mark the end of where this flag ends
		;;
# this is the -m flag when this flag is called we store the argument which is required for this script in user as it should be theusername which is rquired to run the symlink script
	m)
	#defining the user up there to the optarg here
		user=${OPTARG}
		#running the script with the argument it requires
		./make-symlink-script $user
		;;
# this is a case statement match that runs when a flag that expects an argument is run without one then this will run and exit 1 which means the script will stop and terminate
	:)
		echo "Error: run -m with an argument"
		exit 1
		;;
# this is a placeholder for any other single letter so if a flag which isnt an option here runs here then this option will run and the script will terminate
	?)
		echo "Error: that flag does not exist"
		exit 1
		;;
	esac
done
```

### How to use this script:
1. make the script executable:
```bash
chmod u+x call-the-scripts
```
3. make sure to use `sudo` which elevates your permissions
4. include the name of the script like so:
```bash
sudo ./call-the-scripts
```
5. run the scripts with the -i flag for install and -m flag and specify a username for the -m flag, here is what an example looks like:
```bash
sudo ./call-the-scripts -i -m arch
```
if it all goes according to plan then you should have 2 directories a bashrc file, and some packages installed.
## Part 2: New User Script
The purpose of this script is to create a new user without the assistance of any existing built in tools like useradd, adduser, and usermod. it will make the following components a new user which would be appended to existing groups the input would need to specify a home directory and a default shell that the user needs to specify.


##### The new user script is 1 file and includes the following inputs from the user:
A username which needs to be unique, A string of existing groups which the user will be appended to, A default shell which is automatically set to bash if not specified elsewise, and a password which the user will be prompted for if everything goes successfully.

Note: please run the -g flag inside strings separated by spaces like so
```bash
-g "wheel admn arch"
```
Note: please run the -s flag as a full path like so
```bash
-s /bin/bash
```

what this script looks like:
```bash
#!/bin/bash

# this is checking if sudo is used to run this script if its not then exit the script
# explained in detail in call-the-scripts
if [[ "$EUID" -ne 0 ]]; then
	echo "Error: run script with sudo"
	exit 1
fi

# awk is a text finder in here i am using --field-separator to seperate the lines with the colon and printing the third index from the file /etc/passwd
# we pipe it into sort and use -n which is numeric and -r for reverse putting the highest number at the top
# we are piping it into grep with -E which stands for extended regular expression and only looking for number which are 4 digits no less no more which is what the two \ b's are for they are borders
# then finally we pipe it into head -n is for number and we are specifying to only give us the highest number at the top that is 4 digits
# this ultimately is used to give us the latest userid so we can increment it by 1 when we make the new one
# https://opensource.com/article/20/9/awk-ebook
# https://man7.org/linux/man-pages/man1/sort.1.html
highestuserid=$(awk --field-separator ":" '{print $3}' /etc/passwd | sort -nr | grep -E '\b[0-9]{4}\b' | head -n 1)


#we are taking the above variable and incrementing by 1 inside a math expantion then assinging it to newUid
newUid=$(($highestuserid + 1))
# setting the Guid to the same as uid
newGuid=$newUid
# empty name variable which will be filled later
name=""
#default argument incase nothing is provided so we have a shell
specifiedshell="/bin/bash"


#while loop with getopts which expects -u -s -g with arguments for them all
while getopts "u:g:s:" opt; do
	case "${opt}" in
	u)
		# add the user to this variable which they provide after running -u
		name=${OPTARG}
		;;
	g)	
		# add the string they provide to an array called groups
		groups=( ${OPTARG} )
		;;

	s)	
		#specified shell is reassigned here if something is provided
		specifiedshell=${OPTARG}
		;;
		
	:)
		#these are explained in call-the-scripts
		echo "Error:this flag requires an argument"
		exit 1
		;;
	?)
		#these are explained in call-the-scripts
		echo "Error: this option does not exist
		-u <username> (example: -u arch)
		-g <"groups"> (example: -g 'wheel adm group3')
		-s shell (example: -s /bin/bash) 
		"
		exit 1
		;;
	esac
done

# we are reading passwd and trying to pull the name they provided with grep if it return something then this will run stating that this name already exists and to provide a unique one
if [[ $(cat /etc/passwd | grep $name) ]]; then
	echo "Error: This user already exists please provide a unique user"
	exit 1
fi

#this checks if no arguments were passed since they script needs arguments inorder to work
# the hash and dollar sign return the length of the paramteres and if its 0 then no arguments were giving resulting in exit 1
if [[ $# -eq 0 ]]; then
	echo "Error: no options or arguments were passed
        -u <username> (example: -u arch)
        -g <"groups"> (example: -g 'wheel adm group3')
        -s shell (example: -s /bin/bash)"
	exit 1
fi


# we are reading the etc/shell and greping the specified shell they give us and if it dosent exist then we cant set it since they would have to download it first, we then exit 1
if [[ ! $(cat /etc/shells | grep $specifiedshell) ]]; then
	echo "this shell cannot be set it is not in the /etc/shells file to set your shell to $specifiedshell download it first"
	exit 1
fi


# we loop through the array of groups and for every single group they provide we check if the group exists in /etc/group if it dosent then we are exiting and telling them that the group dosent exist
for i in ${groups[@]}; do
	if [[ ! $(cat /etc/group | grep $i) ]]; then
		echo "Error: one of these groups dont exist run the script again with existing groups"
	exit 1
	fi
done


# when both name and shell is not empty so if its a filled string which is what -n is used for and the two & stands for and then we start the creation process
if [[ -n $name && -n $specifiedshell ]]; then

	# this is the exact format of /etc/passwd they are seperated by colons and this is what each field means
	# the first field is username : the second is a placeholder for the password as the real password is in /etc/shadow : the third field is the new userid which we set earlier: the fourth field is the groupid : the fifth field is user id info and its extra info which i put as the username this field isnt as important as the others: the sixth field is the home directory and: the last field is the shell which the user starts on when they log in and we are echoing it all into the /etc/passwd which is what the >> stand for
	echo "$name:x:$newUid:$newGuid::/home/$name:$specifiedshell" >> /etc/passwd
	#each user needs a group the first field is the usersname : the second is a placeholder for the password of the group : and the last one is the unique group id and is the same as uidthen we are appending this to the /etc/group which is where all of the groups are stored
	echo "$name:x:$newGuid:" >> /etc/group
	#we make the home directory for the user and -p lets us make recursive directory which is /home/name
	mkdir -p /home/$name
	# we are copying the contents of /etc/skel which holds all the default home content
	# the . signifies that we are copying the content and not the directory to the directory we made above
	# https://askubuntu.com/questions/86822/how-can-i-copy-the-contents-of-a-folder-to-another-folder-in-a-different-directo
	cp -r /etc/skel/. /home/$name

	# giving the user we just created ownership of their home directory
	chown $name:$name /home/$name
	# adding the user to etc shadow as a placeholder because when we run passwd it changes these field anyway etc shadow is where the hashed passwords are stored
	# https://linuxize.com/post/etc-shadow-file/
	echo "$name:!*:::::::" >> /etc/shadow
	
	# iterating through the groups
	for i in ${groups[@]}; do
		#reading the group file and greping the whole word -w of the group which is the iterate with a colon and x placeholder, this is to make sure the group exists in the file
		groupline=$(cat /etc/group | grep -w "$i:x")
		# then we do the samething as above but pass it to an awk which will divide everything by the colon and print the 4 index, this is to see if a user is already appened to a group or its empty -F is for field seperator and we are seperating it by : 
		# https://www.nixcraft.com/t/linux-append-text-to-end-of-line/1838
		command=$(cat /etc/group | grep -w "$i:x" | awk -F'[ :]' '{print $4}')
		
		#if the command variable above does return something it means that someone there is someone else in the group and it isnt gonna be the first one so it needs to inlcude a comma
		if [[ $command ]]; then
			#sed is a text editor that we are using to replace the current line that get from groupline and add the user with a comma 
			# the dollarsign specifies the end of the line and we are adding a comma and username to it
			# https://www.gnu.org/software/sed/manual/sed.html
			newline=$(sed "s/$/,${name}/" <<< $groupline)
			# -i stands for write in place and now we are replacing the old line without the user and adding the user in it inside /etc/group
			sed -i "s/${groupline}/${newline}/g" /etc/group
		else
			#we are doing the same as the top but there is no comma here which means that there is no other memeber of the group and we are the first
			# https://www.nixcraft.com/t/linux-append-text-to-end-of-line/1838/3
			nofirstuser=$(sed "s/$/${name}/" <<< $groupline)
			sed -i "s/${groupline}/${nofirstuser}/g" /etc/group
		fi
	done
#we use the passwd command to set a password for the user now and echo a successful creation letting them know it went successfully
passwd $name
echo "successful: User $name is created"
#if a user and shell werent specified this is a alst resort to catch them and tell them its an error
else
	echo "Error: provide a username and specifiedshell"
fi
```

### How to use this script:
1. make the script executable:
```bash
chmod u+x NewUser-script
```
3. make sure to use `sudo` which elevates your permissions
4. include the name of the script like so:
Note: this is not the complete command its just to show you how it looks before you run the flags and arguments.
```bash
sudo ./NewUser-script
```
5. type a unique username:
Note: DO NOT include letters or special characters inside the name
```bash
sudo ./NewUser-script -u mellaad
```
6. type a list of existing groups you want to be appended to
```bash
sudo ./NewUser-script -u mellaad -g "wheel arch"
```
7. type a shell you want set as your default
Note: if this shell is not inside /etc/shells then the script will not let you set it as a default
```bash
sudo ./NewUser-script -u mellaad -g "wheel arch" -s /bin/bash
```

Now you can run the script and your new user will be created. If everything goes according to plan you should see a success message indicating that your use has been made:
```bash
successful: User mellaad is created
```
