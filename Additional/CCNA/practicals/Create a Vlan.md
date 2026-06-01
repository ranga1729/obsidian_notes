```python
# go to the global config mode
# create a vlan by giving it an ID

vlan 10
# the number has to be unique on the switch
# pressing enter will get you to the sub confg mode (config-vlan)

# give the vlan a name
name IT

exit
exit

# see the vlan list
sh vlan br
show vlan brief

# now add a port to the newly create IT vlan
# to add a port to the vlan, you need to go to the port's conig

# got to the global config mode
interface fastEthernet 0/1

# now we are on the (config-if)
# now we change the interface mode to 'access'
# access mode allows the port to be in only a one vlan
switchport mode access 
switchport acess vlan 10

exit

# if you want to add first 10 port to the vlan easily
interface range fa0/1 -10
switchport mode access
switchport access vlan 10
```
