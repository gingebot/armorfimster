Fimster
=======
Simple pastable bash function to reduce multiple commands and manual validation to one command to add custom FIM filepaths via Armor agent 3.x

Usage
-----
* First paste the above function into your shell.
* Run the function with required arguments

```
fimster <rule-name> <filepath> <description>
```

Fimster function
----------------

```
fimster () {
if [[ -z $1 ]];then printf "\nplease supply 3 arguments; name, path, description\n\n"; return 1;elif [[ ! -e $2 ]]; then printf "\n 2nd argument must be an existing filepath/directory\n\n"; return 1;elif [[ -z $3 ]];then printf "\n 3rd argument must be a description\n\n";return 1;fi
addrule=;id=;
addrule=$(/opt/armor/armor fim add-custom-filepath-rule "$1,$2,$3");
id=$(echo $addrule | grep -oP '(?<=Output: ).*' | python -m json.tool | grep -oP "(?<=\"ID\": )\d+");
if [[ -n $id ]];then printf "\n\n enabling rule:\n\n"; echo $addrule |  grep -oP '(?<=Output: ).*' | python -m json.tool | grep -P "(?<=\")\S+_$1(?=\")" -B17 -A 5;echo; /opt/armor/armor fim assign-rules $id;
elif [[ $addrule == *"The requested Integrity Monitoring Rule name already exists."* ]]; then printf "\n\n The follow rule with the same name exists:\n\n";
id=$(/opt/armor/armor fim list-available-rules | grep -oP '(?<=Output: ).*' | python -m json.tool | grep -P "(?<=\")\S+_$1(?=\")" -B17 -A 5 | tee /dev/tty | grep -oP "(?<=\"ID\": )\d+");
/opt/armor/armor fim list-assigned-rules | grep -qP "(?<=\")\S+_$1(?=\")" && printf "\n\n Rule is already active\n\n" || printf "\n\n Rule is NOT active. \n\n Run the following command to activate the rule:\n\n /opt/armor/armor fim assign-rules $id\n\n";
else printf "\nunknown error, see output: \n" echo $addrule;fi
}
```
