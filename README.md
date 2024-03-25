# Fortigate
### How to remove (unset) the "mgmt" interface in a multi-VDOM environment from vdom "root":
It is not always easy to change the Fortigate's settings while dealing with multiple test/production/cmo/fmo environments.
The GUI shows 0.0.0.0/0 for the cluster management port and for the dedicated management port as well.
If you have all of your interfaces in a vdom "root" after issuing "show system interface" command:
```
config system interface
    edit "ha"
        set vdom "root"
        set type physical
        set snmp-index 1
    next
    edit "mgmt"
        set vdom "root"
        set management-ip 192.168.1.99 255.255.255.0
        set allowaccess ping https ssh fgfm
        set type physical
        set dedicated-to management
        set role lan
        set snmp-index 2
    next
    edit "port1"
        set vdom "root"
        set type physical
        set snmp-index 3
    next
    edit "port2"
        set vdom "root"
        set type physical
        set snmp-index 4
    next
    edit "port3"
        set vdom "root"
        set type physical
        set snmp-index 5
    next
end
```
Then the following steps will help you to move the "mgmt" interface back to the "global" section and let configure the port1 (or port5, or whatever port you want) as a dedicated cluster port:

1. Do the factory reset (either from the multi-vdom global area or from the top level in a non-vdom)
2. Configure mgmt & routing in order to access the GUI
3. Backup the config and open in notepad++
4. Remove the static route referring to the mgmt interface
5. Remove vdom from the mgmt interface section
6. Edit HA section & set your XXX gateway here (from the removed static routing section):
```
# HA
config system ha
    set group-id 555
    set group-name "test"
    set mode a-p
    set password test
    set hbdev "ha" 0 
    set ha-mgmt-status enable
    config ha-mgmt-interfaces
        edit 1
            set interface "mgmt"
            set gateway XXX.XXX.XXX.XXX
        next
    end
    set override disable
    set priority 200
end
```
7. Upload the config to the fortigate and let it to reboot
8. Once it is back online - change the vdom to the multi-vdom
9. Now you should see no vdom assignment for the mgmt interface after "show system interface" command in a global vdom area. Also mgmt IP and Port1 (or the cluster management port of your choice) IP should be visible in a GUI
