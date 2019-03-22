Monitoring
**********

nova-manage service list
ps -aux | grep nova-api
/var/log/nova*.log

cinder service-list

pgrep -l neutron

glance-control all status

heat service-list

Check service listening on port 5000 (default keystone port): 
lsof -i :5000
netstat -ant | grep 5000