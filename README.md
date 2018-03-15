# grpid.TST
#!/bin/bash
#set -vx

# 2015 Mar 19   RDG
# execute a specific epicmenu based on group membership
# traps are present to keep users captive in epicmenus
# must be in only 1 of 3 groups else the user is kicked off
# this also functions as an ad-hoc security layer
#  if user is not in 1 of the groups they are kicked off
#   because LDAP presents a valid login acct to all in AD
#   the `last` and `lastb` outputs can be used to do checks
#
# upon login GRP is checked for 1 of 3 values
# there can be only 1

# 2015 Aug 24   RDG
# modified to use ldapsearch command - much quicker!
#

trap '' 2 3 6 9 15
OIFS=$IFS
IN=`/usr/bin/ldapsearch -LLL -H ldap://vwdc06.hosp.dhha.org -Y GSSAPI -D svc_epic_ldap -w EpicLDAP! -N -b dc=hosp,dc=dhha,dc=org "(sAMAccountName=${LOGNAME})" 2>&1 | grep '^memberOf' | egrep -i -o 'epicusers|epictechsvcs|epicsrvsvcs|epicis'`
IFS=$'\n' arrIN=($IN)
len=${#arrIN[*]}
#echo $len
#echo ${arrIN[@]}
#echo $LOGNAME

if [ "$len" == "1" ]
then
   case $IN in
      EpicUsers)
         /epic/bin/epicmenu -c epicmenu-user.conf
         ;;
      EpicTechSvcs)
         /epic/bin/epicmenu -c epicmenu-ts.conf
         ;;
      EpicSrvSvcs)
         /epic/bin/epicmenu -c epicmenu-ssts.conf
         ;;
      EpicIS)
         /epic/bin/epicmenu -c epicmenu-is.conf
         ;;
   esac
else
   echo "Incorrect group memberships - too many or not enough"
   exit
fi
IFS=$OIFS

exit
