Playbook
--------

## Create a Stack and AlarmsDefition

DO:

Fill out resource block for **cpu_alarm_high**, **cpu_alarm_low**, **group**:
  - **OS::Heat::AutoScalingGroup**
  - **OS::Monasca::AlarmDefinition**  
  Add properties as defined in [heatintro.md](./heatintro.md)

Create a stack  
`openstack stack create --wait -t autoscaling.yaml test`

CHECK:

Refer to [examples.md](./examples.md) for detailed commands

There should be two alarms.  
`monasca alarm-definition-list`


Check metadata attribute of server.   
It should have `scale_group=(id of stack named teststack)`  
`openstack server show SERVER_ID`


Check outputs of these commands

This should what metrics are collected for each VM  
`monasca metric-list`

Observer the individual measurements corresponding to each metric  
`monasca measurement-list`

Try to do some aggregation and generate some statistics to the metrics.  
`monasca metric-statistics`

After some time there should some metrics for the vms from stack.
To get values other thann 0 (zero), create some load on the vms

```
  # Make sure to type in SERVER_ID on next line

  export FLOATING_IP=openstack floating ip create public  --port=$(openstack port list --device-id=$(openstack server show -f value -c id SERVER_ID ) -f value -c id) -f value -c floating_ip_address
  ssh cirros@$FLOATING_IP "dd if=/dev/zero of=/dev/null &"
```

## Add Notification for alarm definition

DO: 

Delete previous stack  
`openstack stack delete --wait --yes teststack`

Fill out  *up_notification* and *down_notification*:
  - **OS::Monasca::Notification**  
   Add properties as defined in [heatintro.md](./heatintro.md)

Create a stack  
`openstack stack create --wait -t autoscaling.yaml teststack`


CHECK:

Create some load on vm and watch for notifications
```
  # Make sure to type in SERVER_ID on next line

  export FLOATING_IP=openstack floating ip create public  --port=$(openstack port list --device-id=$(openstack server show -f value -c id SERVER_ID ) -f value -c id) -f value -c floating_ip_address
  ssh cirros@$FLOATING_IP "dd if=/dev/zero of=/dev/null &"
```

Refer to [examples.md](./examples.md) for detailed commands


This is the list of webhooks which monasca will call whenever corresponding alarms have been fired  
`monasca notification-list`

Initially no notfications as the alarm have not yet been in **ALARM** state.  
But after ~3 mins the alarm should be in ALARM state  
`monasca alarm-definition-list`

And there should be alarm corresponding to those definitions  
`monasca alarm-list`

## Add scaling policies

DO:

Delete previous stack  
`openstack stack delete --wait --yes teststack`

- Fill out  *scale_up_policy* and *scale_up_policy*:
  - **OS::Heat::ScalingPolicy**  
  Add properties as defined in [heatintro.md](./heatintro.md)

Create a stack  
`openstack stack create --wait -t autoscaling.yaml test`

CHECK

Create some load on vm and watch for notifications
```
  # Make sure to type in SERVER_ID on next line

  export FLOATING_IP=openstack floating ip create public  --port=$(openstack port list --device-id=$(openstack server show -f value -c id SERVER_ID ) -f value -c id) -f value -c floating_ip_address
  ssh cirros@$FLOATING_IP "dd if=/dev/zero of=/dev/null &"
```

As soon as there are alarms  
`monasca alarm-list`  

A new vm should be created  
`openstack server list`

## Experiment with scale-up and scale-down

DO:

ssh to different machines toggle load generating process:  

To stop:   `pkill dd`

To start:  `dd if=/dev/zero of=/dev/null`

CHECK:

Corresponding scale up and scale down in number of VMs  
`openstack server list`

References
----------
* Heat autoscaling [template](https://github.com/openstack/heat-templates/blob/master/hot/monasca/autoscaling.yaml)
* Monasca [libvirt plugin documentation](https://github.com/openstack/monasca-agent/blob/master/docs/Libvirt.md)
* Monasca API [reference](https://github.com/openstack/monasca-api/blob/master/docs/monasca-api-spec.md)