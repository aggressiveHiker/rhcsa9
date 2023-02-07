# Bash Scripting
## Finding help on the system
Run this to find the term "case" in all .sh scripts on the system:
```
find / -iname "*.sh" -exec fgrep -H -n -A3 -B3 -- "case" {} \; 2>/dev/null | less
```

You can also run the same command with a regular expression using egrep:
```
find / -iname "*.sh" -exec egrep -H -n -A3 -B3 -- "if.*then" {} \; 2>/dev/null | less
```

Breakdown of options:
- `-iname` - Name, case insensitive
- `-exec` - Tells "find" to execute "grep"
- `-H` - Print the filename of each match
- `-n` - Print the number line
- `-A3` - Print three lines after the match
- `-B3` - Print three lines before the match
- `--` - The "--" argument treats filenames beginning with "-" as filenames and not arguments. If a file is found that begins with "-" and this option is not present, it will return zero results. 
- `{}` - Hand off the filename retrieved by find to grep
- `\;` - Close the exec command and loop it, segmenting each command with `;`
- `2>/dev/null` - Send all errors to /dev/null

If you want to get crazy, add this to your ~/.bashrc file:
```
search_bash_script ()
{
  find / -xdev -iname "*.sh" -exec fgrep -H -A3 -B3 -- "$1" {} \; 2>/dev/null | less
}
```

Call it by using:
```
search_bash_script <search_term>
```
## Input Arguments

Positional arguments are the arguments you give the script, in order:
```
$0 = Name of Script

$1 = First argument
$2 = Second argument
$3 = Third argument
...
$n = n argument

$# = Number of arguments provided
$@ = List of arguments provided
```

Here's a script that shows how they can be used:
```
#!/bin/bash

if [ $# -eq 0 ] ; then
  echo "$0: Missing arguments"
  exit 1
elif [ $# -gt 25 ] ; then
  echo "$0: Too many arguments: $@"
  exit 1
else
  echo "We Got some argument(s)"
  echo "==========================="
  echo "Number of arguments.: $#"
  echo "List of arguments...: $@"
  echo "Arg #1..............: $1"
  echo "Arg #2..............: $2"
  echo "==========================="
fi

echo "And then we do something with $1 $2"
```

## Arithmetic Operators
Note: These can only be used on integers
- `-eq` - Equal to
- `-ne` - Not equal to
- `-lt` - Less than
- `-le` - Less than or equal to
- `-gt` - Greater than
- `-ge` - Greater than or equal to


```
#!/bin/bash

if [ "$1" = "me" ]; then
  echo "Yes, I'm awesome."
  exit 10

elif [ "$1" = "them" ]; then
  echo "Okay, they are awesome."
  exit 20

else
  echo "Usage ./awesom.sh me|them"
  exit 30

fi
```

```
#!/bin/bash

# Script to create users
# Takes input file with user names
# on the command line as the first argument

# User name file
USERFILE=$1

if [ "$USERFILE" = "" ]
then
        echo "Please specify an input file!"
        exit 10
elif test -e $USERFILE
then
        for user in `cat $USERFILE`
        do
    echo "Creating the "$user" user..."
                useradd -m $user

        done
        exit 20
else
        echo "Invalid input file specified!"
        exit 30
fi
```

```
#!/bin/bash

# Script to allow our web_admin users to perform som 'dnf' checks
# Usage: ./web_admin.sh <action> <package>
# check-updates: No package to specify
# check-installed: Please specify a package name
# check-available: Please specify a package name

# Define our variables
# Script inputs
ARG1=$1
ARG2=$2

# Other Variables
DNF=/usr/bin/dnf

# Let's go!

if [ "$ARG1" = "check-updates" ] ; then
  $DNF check-update >> web_admin.log
  DNF_RESULT=$?
    case $DNF_RESULT in
      100)
        echo "Updates available!"
        exit 111
        ;;
      0)
        echo "No updates available!"
        exit 112
        ;;
      1)
        echo "Error!"
        exit 113
        ;;
    esac


elif [ "$ARG1" = "check-installed" ] ; then
  $DNF list --installed $ARG2 >> web_admin.log 2>&1
  DNF_RESULT=$?
    case $DNF_RESULT in
      0)
        echo "Package is installed!"
        exit 114
        ;;
      1)
        echo "Package is NOT installed or not available!"
        exit 115
        ;;
    esac

elif [ "$ARG1" = "check-available" ] ; then
  $DNF list --available $ARG2 >> web_admin.log 2>&1
  DNF_RESULT=$?
    case $DNF_RESULT in
      0)
        echo "Package is available!"
        exit 116
        ;;
      1)
        echo "Package is NOT available or does not exist!"
        exit 117
        ;;
    esac


else
  echo "INVALID OPTIONS. Please specify one of the following:"
  echo "check-updates: No package to specify"
  echo "check-installed: Please specify a package name"
  echo "check-available: Please specify a package name"
  exit 118
fi
```

### Super-Awesome Create User Script
```
#!/bin/bash
### creategroup.sh

for group in dba_admin dba_managers dba_staff dba_intern it_staff it_managers ; do
    groupadd $group
    done

```

```
### userlist.txt
manny:1010:dba_admin,dba_managers,dba_staff
moe:1011:dba_admin,dba_staff
jack:1012:dba_intern,dba_staff
marcia:1013:it_staff,it_managers
jan:1014:dba_admin,dba_staff
cindy:1015:dba_intern,dba_staff

```

```
#!/bin/bash
### createuser.sh

while IFS=":" read -r user uid group
do useradd -u $uid -G $group $user;
done < userlist.txt

```