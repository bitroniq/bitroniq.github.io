 #! /bin/bash
 #usage: Dynamic_Menu.bash /home/user/target_directory

 declare -a array

 i=1 #Index counter for adding to array
 j=1 #Option menu value generator

 while read line
 do
    array[ $i ]=$j
    (( j++ ))
    array[ ($i + 1) ]=$line
    (( i=($i+2) ))

 done < <(find $1 -type f) #consume file path provided as argument

 #Define parameters for menu
 TERMINAL=$(tty) #Gather current terminal session for appropriate redirection
 HEIGHT=20
 WIDTH=76
 CHOICE_HEIGHT=16
 BACKTITLE="Back_Title"
 TITLE="Dynamic Dialog"
 MENU="Choose a file:"

 #Build the menu with variables & dynamic content
 CHOICE=$(dialog --clear \
                 --backtitle "$BACKTITLE" \
                 --title "$TITLE" \
                 --menu "$MENU" \
                 $HEIGHT $WIDTH $CHOICE_HEIGHT \
                 "${array[@]}" \
                 2>&1 >$TERMINAL)
