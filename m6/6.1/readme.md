A. Create a script that uses the following keys:1. When starting without parameters, it will display a list of possible keys and their description. 
2. The --all key displays the IP addresses and symbolic names of all hosts in the current subnet 
3. The --target key displays a list of open system TCP ports.The code that performs the functionality of each of the subtasks must be placed in a separate function

#!/bin/bash

function show_subnet {
    echo "Hosts in subnet: "
    
    ip -o -f inet addr show | awk '/scope global / {print $4}'
    
    arp -a
}

function show_open_ports {
    echo "All open ports in system: "
    sudo netstat -tulpn | grep tcp
}


# check parameters that were sent
if [[ $1 = "--all" ]]
then
    show_subnet
elif [[ $1 = "--target" ]]
then
    show_open_ports
elif [[ $# -eq 0 ]]
then
    echo "Usage of script:"
    echo "--all - displays the IP addresse and symbolic names of all hosts in the current subnet"
    echo "--target - displays a list of open system TCP ports"
fi





B. Using Apache log example create a script to answer the following questions:1. From which ip were the most requests? 
2. What is the most requested page? 
3. How many requests were there from each ip? 
4. What non-existent pages were clients referred to? 
5. What time did site get the most requests?
6. What search bots have accessed the site? (UA + IP)

#!/bin/env bash

set -e

FNAME="./apache_logs.txt"

IP_REQ=$(awk '{print $1}' "$FNAME" | sort | uniq -c | sort -nr | head -1 | awk '{print $1 " requests from " $2}')
PAGE_REQ=$(awk '{print $7}' "$FNAME" | sort | uniq -c | sort -nr | head -1 | awk '{print $2 " requested " $1 " times"}')
EACH_IP_REQS=$(awk '{print $1}' "$FNAME" | sort | uniq -c | sort -nr | awk '{print $1 " times from " $2}')
ERR_PAGES=$(awk '{if ($9 != 200) {print "ERR " $9 " -> " $7}}' "$FNAME" | sed '/^ERR [125]/d' | uniq)
ACTIVE_TIME=$(awk '{print $4, $5}' "$FNAME" | tr -d '[]' | sed 's|\(.*\):.*|\1|' | sort | uniq -c | sort -nr | head -3 | awk '{print $1 " requests at " $2}')
BOTS_ACCESSED=$(awk '{print $1, " -> ", $12, $13, $14, $15}' "$FNAME" | grep -i 'bot' | sed -e 's/\^M//' | uniq -u)


printf "IP with most requests:\n%s\n\n" "$IP_REQ"
printf "Most requested resource:\n%s\n\n" "$PAGE_REQ"
printf "Request stats per IP:\n%s\n\n" "$EACH_IP_REQS"
printf "Non-existent pages encounteres:\n%s\n\n" "$ERR_PAGES"
printf "Most active time:\n%s\n\n" "$ACTIVE_TIME"
printf "Search bot access:\n%s\n\n" "$BOTS_ACCESSED"








C. Create a data backup script that takes the following data as parameters:
1. Path to the syncing  directory.
2. The path to the directory where the copies of the files will be stored.
In case of adding new or deleting old files, the script must add a corresponding entry to the log file indicating the time, type of operation and file name. [The command to run the script must be added to crontab with a run frequency of one minute]

SCRIPT
  GNU nano 4.8                                                                             
#!/bin/bash
backup_1()
{
        rsync -av --delete $1 $2 --log-file=backup.log
}

backup_2()
{
        read -p "Please enter the path to source directory: " src
        read -p "Please enter the path to destination directory: " dst
        rsync -av --delete $src $dst --log-file=backup.log
}

if [ "$#" -ne "2" ]
then
        backup_2
else
        backup_1 "$1" "$2"
fi
                                    
