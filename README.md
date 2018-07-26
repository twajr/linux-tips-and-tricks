# TESTING
# linux-tips-and-tricks

## Version Checking
Ubuntu:
```
cat /etc/*release
```
## CRON
Push cron command output to syslog (stops MTA errors)
```
0 3 * * * (cmd1;  cmd2) 2>&1 | logger -t mycmd
```
## Some basic command line tips

  * 'cd -'  Takes you back to previous directory
  * '!!' - repeats last command
  * 'history' - displays your command history - '![line number] executes that command again
  * 'whoami' - shows my ID
  * '

## File / Directory Permissions

 * 'chmod' to change file an directory 'modes'
 * 'umask' to set default mask for file creation

## User Admin

 * /etc/passwd - contains the user information
 * 'useradd username' - Adds users to the system
 * 'passwd username' - Sets the password for the account

### UIDs and GIDs

 * UIDs - Users
   * Root account is always UID 0
   * User identifiers 
   * System accounts have UIDs < 1000
 * GIDs - Groups
   * 'groups' command shows members
   * 'groupadd', 'groupmod', 'groupdel' to manage groups
   * /etc/group stores the groups
   * GID is listed in /etc/passwd is defauilt group for the account
   * New files will belong to a user's default group
   * Use 'newgrp' to switch between groups

## Networking

 * 'route -n' 
 * 'ip a s' or 'ip addresss show'
 * 'ifconfig' - Is deprecated, but is around still, replaced by ip command above
 * 'hostname -f' - to show FQDN - This is stored in /etc/hostname on Rhel
 * 'host' or 'dig' to show resolve of DNS
 * /etc/hosts local file to resolve hosts or override DNS entries
 * /etc/nsswitch.conf - Controls order of name lookup (files dns)
 * /etc/services - maps port names to port numbers
 * Troubleshooting - 'ping, traceroute, netstat, tcpdump, telnet'

### MTU
Dynamically change the MTU size on a NIC.
```
ip link set eth0 mtu 1400
```
### iptables
drop all the rules
```
iptables -F
```
Zero, then have a look at the current nat / masq rules
```
iptables -t nat -Z
iptables -t nat -L -v -n
```


The ip route get command is a useful feature that allows you to query the route on which the system will send packets to reach a specified IP address, for example:
```
$ ip route get 23.6.118.140 
23.6.118.140 via 10.0.2.2 dev eth0  src 10.0.2.15 
cache  mtu 1500 advmss 1460 hoplimit 64 
```
In this example, packets to 23.6.118.140 are sent out of the eth0 interface via the gateway 10.0.2.2.

Stop all interfaces using
```
sudo /etc/init.d/networking stop
```
Configure (edit) your interfaces as you wish, then
Start them again
```
sudo /etc/init.d/networking start
```
### libvirt and Networking
You can find the network device logical name by the command:
```
sudo lshw -C network
```

## Jobs

 * bg, fg, jobs, and CTRL-Z manage background and foreground processes
 * Jobs can be managed with either the process ID or job ID

## Shell Scripting Details
/etc/shells contains list of available shells - Users initial shell is set in /etc/passwd

## Script basics

Scripts begin with the 'Shebang' of #!/bin/bash and then continue with the script commands. The script needs to be executable so a 'chmod 755 myscript.sh' is required before running it.

 ```
 #!/bin/bash
 sleep 90
 SERVER_NAME=$(hostname)
 echo "You are running on ${SERVER_NAME}."
 ```

### variables
 * `varname=value` - setting our own values
   * usage: `foo=bar && echo $foo` (basic)
   * quotes: `greeting='hello world' && echo $greeting` (literal)
   * quotes: `intro="well $greeting" && echo $intro` (substitution)
   * commands: `lines=$(cat somefile | wc -l)` (command substitution)
   * export: `export var=value` (export to shell so accessible in other scripts)
   * length: `${#varname}` (prints number of characters in varname)
   * truncate: `"${varname: -n}"` (prints last n chars - NEED QUOTES)
   * after: `${varname##*[char]}` (prints chars following char `mypod=kafka-1 && echo ${mypod##*-} # 1`)
 * `$0` - name of script
 * `$1-9` - first 9 arguments passed to script
   * usage: `echo $1`
 * `$@` - all arguments passed to script
 * `$_` - prints target of last command (like folder created)
   * usage: `mkdir somefolder && cd $_`
 * `$#` - counts number of arguments
   * usage: `if [$# -lt 2] then echo "missing required args" exit fi`
 * `$$` - current process ID
 * `$!` - background process ID
 * `$?` - exit status of last executed process
 * `$USER` - username of user running script
 * `$HOSTNAME` - hostname of machine script is running on
 * `$SECONDS` - number of seconds since script was started
 * `$RANDOM` - returns different random number each time
   * usage `echo $RANDOM`
 * `$LINENO` - returns current line number in bash script

Demonstrates passing list of similar arguments
```
for USER in $@
do
  echo "Argument is: $USER"
done
```

## Reading STDIN
 * `read varname` - waits for user input and assigns value to `$varname`
   * usage: `echo "What is your name?" && read firstname && echo -e "Hello, $firstname"`
 * `read var1 var2 var3` - allows user to input multiple values
   * usage: `echo "3 favorite colors?" && read col1 col2 col3 && echo -e "You like $col1, $col2, $col3"`
 * `read -p 'Prompt: ' varname` - prompts for input
   * usage: `read -p 'Username: ' uservar && echo -e "Hello, $uservar"`
 * `read -sp 'Prompt: ' varname` - prompt for input with hidden values
   * usage: `read -sp 'Password: ' passvar && echo -e 'Your password is updated!'`

## arithmetic
 * `let var = 5 + 3` - assigns result to var
 * `expr 5 + 3` - prints result
 * `$(( 5 + 3 ))` - returns result

## conditional
If: 
```
if [ $1 -ge 18 ] && ( [ $USER == 'bob' ] || [ $USER == 'andy' ] )
then
  echo You may go to the party.
elif [ $2 == 'yes' ]
then
  echo You may go to the party but be back before midnight.
else
  echo You may not go to the party.
fi
```

Case:
```
case $1 in
  start)
    echo starting
    ;;
  stop)
    echo stoping
    ;;
  restart)
    echo restarting
    ;;
  *)
    echo don\'t know
    ;;
esac
```

## tests
| Operator	| Description |
|----------|-------------|
| ! EXPRESSION	| The EXPRESSION is false. |
| -n STRING	| The length of STRING is greater than zero. |
| -z STRING	| The lengh of STRING is zero (ie it is empty). |
| STRING1 = STRING2	| STRING1 is equal to STRING2 |
| STRING1 != STRING2	| STRING1 is not equal to STRING2 |
| INTEGER1 -eq INTEGER2	| INTEGER1 is numerically equal to INTEGER2 |
| INTEGER1 -gt INTEGER2	| INTEGER1 is numerically greater than INTEGER2 |
| INTEGER1 -lt INTEGER2	| INTEGER1 is numerically less than INTEGER2 |
| -d FILE	| FILE exists and is a directory. |
| -e FILE	| FILE exists. |
| -r FILE	| FILE exists and the read permission is granted. |
| -s FILE	| FILE exists and it's size is greater than zero (ie. it is not empty). |
| -w FILE	| FILE exists and the write permission is granted. |
| -x FILE	| FILE exists and the execute permission is granted. |
| -f FILE | FILE exists and is a REGULAR file (ie. it's not a device).|

## loops
 * You can use `break` and `continue` within loop

While:
```
counter=1
while [ $counter -le 10 ]
do
  echo $counter
  ((counter++))
done
echo All done
```

Until:
```
counter=1
until [ $counter -gt 10 ]
do
  echo $counter
  ((counter++))
done

echo All done
```

For:
```
names='Stan Kyle Cartman'
for name in $names
do
  echo $name
done
...
for COLOR in red green blue
do
  echo "COLOR: $COLOR"
done
```
