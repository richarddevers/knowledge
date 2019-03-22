Manual installation
********************

.. code-block:: bash

    #!/bin/sh -e

    #Pour swift il faut spécifier P1, P2 et P3
    #Surement à /dev/sda6, /dev/sda7 et /dev/sda8
    #Specifier /var/file1, /var/file2, /var/file3 pour utiliser des fichiers
    #montés en loop pour la formation sur les VM Cloudwatt
    #P1=/dev/sda6
    #P2=/dev/sda7
    #P3=/dev/sda8


    if [ $USER != 'root' ]; then
        echo "Must be root."
        exit 1
    fi

    APTINSTALL="apt install -y --force-yes -o Dpkg::Options::=--force-confold"

    get_id () {
        echo `"$@" | awk '/ id / { print $4 }'`
    }

    deploy_mysql () {
        apt update
        apt install ubuntu-cloud-keyring -y
        echo 'deb http://ubuntu-cloud.archive.canonical.com/ubuntu xenial-updates/pike main' > /etc/apt/sources.list.d/pike.list
        apt update

        $APTINSTALL python-pymysql mariadb-server

        mkdir -p /etc/mysql/mariadb.conf.d
        cp configs/mariadb.conf /etc/mysql/mariadb.conf.d/60-openstack.cnf

        systemctl restart mysql
    }

    deploy_rabbit () {
        $APTINSTALL rabbitmq-server
        rabbitmqctl add_user openstack openstack
        rabbitmqctl set_permissions openstack ".*" ".*" ".*"
    }

    deploy_keystone () {
        mysql -uroot -proot << EOF
    DROP DATABASE IF EXISTS keystonedb;
    CREATE DATABASE keystonedb;
    GRANT ALL ON keystonedb.* TO 'keystone'@'localhost' IDENTIFIED BY 'keystonemypass';
    GRANT ALL ON keystonedb.* TO 'keystone'@'%' IDENTIFIED BY 'keystonemypass';
    EOF

        apt update
        $APTINSTALL keystone python-openstackclient
        cp configs/keystone.conf /etc/keystone/keystone.conf
        keystone-manage db_sync
        keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone

        keystone-manage bootstrap \
            --bootstrap-username admin \
            --bootstrap-password adminpass \
            --bootstrap-project-name admin \
            --bootstrap-role-name admin \
            --bootstrap-service-name keystone \
            --bootstrap-admin-url http://controller:35357/v3 \
            --bootstrap-public-url http://controller:5000/v3 \
            --bootstrap-internal-url http://controller:5000/v3 \
            --bootstrap-region-id RegionOne

        systemctl restart apache2

        sleep 5

        export OS_PROJECT_DOMAIN_ID=default
        export OS_USER_DOMAIN_ID=default
        export OS_PROJECT_NAME=admin
        export OS_TENANT_NAME=admin
        export OS_USERNAME=admin
        export OS_PASSWORD=adminpass
        export OS_AUTH_URL=http://controller:5000/v3
        export OS_REGION_NAME=RegionOne
        export OS_IDENTITY_API_VERSION=3

        openstack project create demo
        openstack project create service
        openstack role add --project demo --user admin admin

        cat > ~/admrc << EOF
    export OS_PROJECT_DOMAIN_ID=default
    export OS_USER_DOMAIN_ID=default
    export OS_PROJECT_NAME=demo
    export OS_TENANT_NAME=demo
    export OS_USERNAME=admin
    export OS_PASSWORD=adminpass
    export OS_AUTH_URL=http://controller:5000/v3
    export OS_REGION_NAME=RegionOne
    export OS_IDENTITY_API_VERSION=3
    export OS_VOLUME_API_VERSION=2
    export OS_IMAGE_API_VERSION=2
    EOF

        grep -q admrc ~/.bashrc || echo '. ~/admrc' >> ~/.bashrc
    }

    deploy_glance () {
        . ~/admrc

        mysql -uroot -proot << EOF
    DROP DATABASE IF EXISTS glancedb;
    CREATE DATABASE glancedb;
    GRANT ALL ON glancedb.* TO 'glance'@'localhost' IDENTIFIED BY 'glancemypass';
    GRANT ALL ON glancedb.* TO 'glance'@'%' IDENTIFIED BY 'glancemypass';
    EOF

        apt update
        $APTINSTALL glance-api
        cp configs/glance-api.conf /etc/glance
        glance-manage db_sync
        glance-manage db_load_metadefs

        openstack user create --password glancekeypass glance
        openstack role add --project service --user glance admin
        openstack service create --name glance \
            --description "OpenStack Image service" image
        openstack endpoint create --region RegionOne \
            image public http://controller:9292
        openstack endpoint create --region RegionOne \
            image internal http://controller:9292
        openstack endpoint create --region RegionOne \
            image admin http://controller:9292

        systemctl restart glance-api.service
        sleep 5

        grep OS_IMAGE_API_VERSION ~/admrc || echo "export OS_IMAGE_API_VERSION=2" >> ~/admrc
        wget http://archive.objectif-libre.com/images/cirros-0.3.4-x86_64-disk.img \
            -O cirros-0.3.4-x86_64-disk.img
        openstack image create --disk-format qcow2 \
            --public --file cirros-0.3.4-x86_64-disk.img CirrOS
    }

    deploy_nova () {
        . ~/admrc

        mysql -uroot -proot << EOF
    DROP DATABASE IF EXISTS novadb;
    CREATE DATABASE novadb;
    GRANT ALL ON novadb.* TO 'nova'@'localhost' IDENTIFIED BY 'novamypass';
    GRANT ALL ON novadb.* TO 'nova'@'%' IDENTIFIED BY 'novamypass';
    DROP DATABASE IF EXISTS novaapidb;
    CREATE DATABASE novaapidb;
    GRANT ALL ON novaapidb.* TO 'nova'@'%' IDENTIFIED BY 'novamypass';
    GRANT ALL ON novaapidb.* TO 'nova'@'localhost' IDENTIFIED BY 'novamypass';
    DROP DATABASE IF EXISTS novadb_cell0;
    CREATE DATABASE novadb_cell0;
    GRANT ALL ON novadb_cell0.* TO 'nova'@'%' IDENTIFIED BY 'novamypass';
    GRANT ALL ON novadb_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'novamypass';
    EOF

        apt update
        $APTINSTALL nova-api nova-scheduler nova-conductor \
            nova-novncproxy nova-consoleauth nova-placement-api

        cp configs/nova.conf /etc/nova/nova.conf
        nova-manage api_db sync
        # Config Cells_v2 From https://docs.openstack.org/developer/nova/cells.html :
        # nova-manage will use the [database]/connection value from your config file,
        # and mangle the database name to have a _cell0 suffix
        nova-manage cell_v2 map_cell0
        # cell0 is a special cell, we now must create a regular default cell where
        # to put all our hosts
        nova-manage cell_v2 create_cell --name cell1
        # Run common database migration
        nova-manage db sync

        # Nova API
        openstack user create --password novakeypass nova
        openstack role add --project service --user nova admin
        openstack service create --name nova \
            --description "OpenStack Compute API" compute
        openstack endpoint create --region RegionOne \
            compute public http://controller:8774/v2.1
        openstack endpoint create --region RegionOne \
            compute internal http://controller:8774/v2.1
        openstack endpoint create --region RegionOne \
            compute admin http://controller:8774/v2.1

        # Nova placement API
        openstack user create --password placementkeypass placement
        openstack role add --project service --user placement admin
        openstack service create --name placement \
            --description "OpenStack Placement API" placement
        openstack endpoint create --region RegionOne \
            placement public http://controller:8778
        openstack endpoint create --region RegionOne \
            placement internal http://controller:8778
        openstack endpoint create --region RegionOne \
            placement admin http://controller:8778

        for i in api conductor novncproxy consoleauth scheduler; do
            systemctl restart nova-$i
        done
    }

    deploy_horizon () {
        apt update
        $APTINSTALL openstack-dashboard

        cat >> /etc/openstack-dashboard/local_settings.py << EOF

    OPENSTACK_API_VERSIONS = {
        "identity": 3,
        "volume": 2,
        "compute": 2,
    }
    EOF
        sed -i 's/^OPENSTACK_HOST.*/OPENSTACK_HOST = "192.168.122.10"/' /etc/openstack-dashboard/local_settings.py
        sed -i 's/^DEFAULT_THEME.*/DEFAULT_THEME = "default"/' /etc/openstack-dashboard/local_settings.py

        systemctl restart apache2.service
    }

    deploy_neutron () {
        . ~/admrc

        mysql -uroot -proot << EOF
    DROP DATABASE IF EXISTS neutrondb;
    CREATE DATABASE neutrondb;
    GRANT ALL ON neutrondb.* TO 'neutron'@'localhost' IDENTIFIED BY 'neutronmypass';
    GRANT ALL ON neutrondb.* TO 'neutron'@'%' IDENTIFIED BY 'neutronmypass';
    EOF

        # then neutron
        cat > /etc/sysctl.d/20-neutron.conf << EOF
    net.ipv4.ip_forward=1
    net.ipv4.conf.all.rp_filter=0
    net.ipv4.conf.default.rp_filter=0
    EOF
        modprobe bridge
        sysctl -p /etc/sysctl.d/20-neutron.conf

        apt update
        $APTINSTALL neutron-server neutron-plugin-ml2 neutron-openvswitch-agent \
            neutron-l3-agent neutron-dhcp-agent
        for i in neutron.conf l3_agent.ini dhcp_agent.ini metadata_agent.ini; do
            cp configs/$i /etc/neutron/
        done
        echo 'dhcp-option-force=26,1450' > /etc/neutron/dnsmasq.conf
        cp configs/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini
        ln -sf ml2_conf.ini /etc/neutron/plugins/ml2/openvswitch_agent.ini

        openstack user create --password neutronkeypass neutron
        openstack role add --project service --user neutron admin
        openstack service create --name neutron \
            --description "OpenStack Networking" network
        openstack endpoint create --region RegionOne \
            network public http://controller:9696
        openstack endpoint create --region RegionOne \
            network internal http://controller:9696
        openstack endpoint create --region RegionOne \
            network admin http://controller:9696

        neutron-db-manage --config-file /etc/neutron/neutron.conf \
            --config-file /etc/neutron/plugins/ml2/ml2_conf.ini \
            upgrade head

        cat >/etc/network/interfaces << EOF
    auto lo
    iface lo inet loopback

    auto eth0
    iface eth0 inet static
    address 192.168.122.10
    netmask 255.255.255.0
    gateway 192.168.122.1
    dns-nameservers 8.8.8.8

    auto eth1
    iface eth1 inet static
    address 192.168.123.10
    netmask 255.255.255.0

    auto eth2
    iface eth2 inet static
    address 0.0.0.0

    auto br-ex
    iface br-ex inet static
    address 0.0.0.0
    EOF

        ovs-vsctl add-br br-ex
        ovs-vsctl add-port br-ex eth2
        ifup eth1
        ifconfig eth2 up
        ifconfig br-ex up

        for i in server openvswitch-agent l3-agent dhcp-agent metadata-agent; do
            service neutron-$i restart
        done
    }

    deploy_cinder () {
        . ~/admrc

        mysql -uroot -proot << EOF
    DROP DATABASE IF EXISTS cinderdb;
    CREATE DATABASE cinderdb;
    GRANT ALL ON cinderdb.* TO 'cinder'@'localhost' IDENTIFIED BY 'cindermypass';
    GRANT ALL ON cinderdb.* TO 'cinder'@'%' IDENTIFIED BY 'cindermypass';
    EOF

        apt update
        $APTINSTALL cinder-scheduler cinder-api cinder-volume thin-provisioning-tools

        pvcreate -ff /dev/vdb
        vgcreate -ff cinder-volumes /dev/vdb

        cp configs/cinder.conf /etc/cinder/cinder.conf
        cinder-manage db sync

        openstack user create --password cinderkeypass cinder
        openstack role add --project service --user cinder admin
        openstack service create --name cinder \
            --description "OpenStack Block Storage" volume
        openstack endpoint create --region RegionOne \
            volume public http://controller:8776/v1/%\(tenant_id\)s
        openstack endpoint create --region RegionOne \
            volume internal http://controller:8776/v1/%\(tenant_id\)s
        openstack endpoint create --region RegionOne \
            volume admin http://controller:8776/v1/%\(tenant_id\)s
        openstack service create --name cinderv2 \
            --description "OpenStack Block Storage" volumev2
        openstack endpoint create --region RegionOne \
            volumev2 public http://controller:8776/v2/%\(tenant_id\)s
        openstack endpoint create --region RegionOne \
            volumev2 internal http://controller:8776/v2/%\(tenant_id\)s
        openstack endpoint create --region RegionOne \
            volumev2 admin http://controller:8776/v2/%\(tenant_id\)s
        openstack service create --name cinderv3 \
            --description "OpenStack Block Storage" volumev3
        openstack endpoint create --region RegionOne \
            volumev3 public http://controller:8776/v3/%\(project_id\)s
        openstack endpoint create --region RegionOne \
            volumev3 internal http://controller:8776/v3/%\(project_id\)s
        openstack endpoint create --region RegionOne \
            volumev3 admin http://controller:8776/v3/%\(project_id\)s

        service lvm2-lvmetad restart
        service apache2 restart
        service cinder-scheduler restart
        service cinder-volume restart
        service nova-api restart

        grep OS_VOLUME_API_VERSION ~/admrc || echo "export OS_VOLUME_API_VERSION=2" >> ~/admrc
    }

    deploy_compute () {
        cat > ~/admrc << EOF
    export OS_PROJECT_DOMAIN_ID=default
    export OS_USER_DOMAIN_ID=default
    export OS_PROJECT_NAME=demo
    export OS_TENANT_NAME=demo
    export OS_USERNAME=admin
    export OS_PASSWORD=adminpass
    export OS_AUTH_URL=http://controller:5000/v3
    export OS_REGION_NAME=RegionOne
    export OS_IDENTITY_API_VERSION=3
    export OS_VOLUME_API_VERSION=2
    export OS_IMAGE_API_VERSION=2
    EOF
        . ~/admrc

        MYIP=$(ifconfig | egrep -o 'addr:192.168.122.[^ ]+' | cut -d: -f2)
        [ -z "$MYIP" ] && echo "IP non trouvée" && exit 1

        MYNEUTRONIP=$(ifconfig | egrep -o 'addr:192.168.123.[^ ]+' | cut -d: -f2)
        [ -z "$MYNEUTRONIP" ] && echo "IP non trouvée" && exit 1

        apt update
        $APTINSTALL nova-compute-kvm nova-compute \
            neutron-plugin-ml2 neutron-openvswitch-agent

        cat > /etc/sysctl.d/20-neutron.conf << EOF
    net.ipv4.ip_forward=1
    net.ipv4.conf.all.rp_filter=0
    net.ipv4.conf.default.rp_filter=0
    EOF
        sysctl -p /etc/sysctl.d/20-neutron.conf

        cp configs/neutron.conf /etc/neutron
        cp configs/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini

        sed -i "s/local_ip.*/local_ip = $MYNEUTRONIP/" /etc/neutron/plugins/ml2/ml2_conf.ini
        sed -i "/bridge_mappings/d" /etc/neutron/plugins/ml2/ml2_conf.ini

        ln -sf ml2_conf.ini /etc/neutron/plugins/ml2/openvswitch_agent.ini

        cat > /etc/nova/nova.conf << EOF
    [DEFAULT]
    debug=True
    dhcpbridge_flagfile=/etc/nova/nova.conf
    dhcpbridge=/usr/bin/nova-dhcpbridge
    log-dir=/var/log/nova
    state_path=/var/lib/nova
    force_dhcp_release=True
    verbose=True
    ec2_private_dns_show_ip=True
    enabled_apis=osapi_compute,metadata
    my_ip = 192.168.122.1
    use_neutron = True
    firewall_driver = nova.virt.firewall.NoopFirewallDriver

    transport_url = rabbit://openstack:openstack@controller:5672

    [api]
    auth_strategy = keystone

    [placement]
    os_region_name = RegionOne
    project_domain_name = Default
    project_name = service
    auth_type = password
    user_domain_name = Default
    auth_url = http://controller:35357/v3
    username = placement
    password = placementkeypass

    [vnc]
    enabled = True
    novncproxy_port = 6080
    novncproxy_host = 192.168.122.10
    vncserver_listen = 192.168.122.1
    vncserver_proxyclient_address = 192.168.122.1
    novncproxy_base_url = http://192.168.122.10:6080/vnc_auto.html

    [glance]
    api_servers = http://controller:9292

    [neutron]
    url = http://controller:9696
    auth_url = http://controller:35357
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    region_name = RegionOne
    project_name = service
    username = neutron
    password = neutronkeypass
    EOF

        adduser nova kvm

        service nova-compute restart
        service neutron-openvswitch-agent restart

        # Map our new compute to "cell1" (default cell in our configuration)
        ssh root@controller nova-manage cell_v2 discover_hosts --verbose
    }

    deploy_heat () {
        . ~/admrc

        mysql -uroot -proot << EOF
    DROP DATABASE IF EXISTS heatdb;
    CREATE DATABASE heatdb;
    GRANT ALL ON heatdb.* TO 'heat'@'localhost' IDENTIFIED BY 'heatmypass';
    GRANT ALL ON heatdb.* TO 'heat'@'%' IDENTIFIED BY 'heatmypass';
    EOF

        apt update
        $APTINSTALL heat-api heat-api-cfn heat-engine
        cp configs/heat.conf /etc/heat/heat.conf
        heat-manage db_sync

        openstack user create --password heatkeypass heat
        openstack role add --project service --user heat admin
        openstack role create heat_stack_owner
        openstack role create heat_stack_user
        openstack service create --name heat \
            --description "Orchestration" orchestration
        openstack service create --name heat-cfn \
            --description "Orchestration" cloudformation
        openstack endpoint create --region RegionOne \
            orchestration public http://controller:8004/v1/%\(tenant_id\)s
        openstack endpoint create --region RegionOne \
            orchestration internal http://controller:8004/v1/%\(tenant_id\)s
        openstack endpoint create --region RegionOne \
            orchestration admin http://controller:8004/v1/%\(tenant_id\)s
        openstack endpoint create --region RegionOne \
            cloudformation public http://controller:8000/v1
        openstack endpoint create --region RegionOne \
            cloudformation internal http://controller:8000/v1
        openstack endpoint create --region RegionOne \
            cloudformation admin http://controller:8000/v1

        openstack domain create heat_user_domain
        openstack user create \
            --domain heat_user_domain \
            --password heat_domain_pass \
            heat_domain_admin
        openstack role add \
            --domain heat_user_domain \
            --user heat_domain_admin \
            admin

        service heat-api restart
        service heat-api-cfn restart
        service heat-engine restart
    }

    deploy_ceilometer () {
        apt update
        $APTINSTALL mongodb-server mongodb-clients python-pymongo
        sed -i 's/127.0.0.1/0.0.0.0/' /etc/mongodb.conf
        echo smallfiles = true >> /etc/mongodb.conf
        service mongodb stop
        rm -f /var/lib/mongodb/journal/prealloc.*
        service mongodb start
        sleep 10
        mongo --host controller --eval '
        db = db.getSiblingDB("ceilometerdb");
        db.addUser({user: "ceilometer",
        pwd: "ceilometermypass",
        roles: [ "readWrite", "dbAdmin" ]})'

        . ~/admrc

        $APTINSTALL ceilometer-api ceilometer-collector ceilometer-agent-central \
            ceilometer-agent-notification ceilometer-alarm-evaluator ceilometer-alarm-notifier \
            python-ceilometerclient

        cp configs/ceilometer.conf /etc/ceilometer/ceilometer.conf
        keystone user-create --name=ceilometer --pass="ceilometerkeypass" --email=ceilometer@forma.com
        keystone user-role-add --user ceilometer --role admin --tenant service
        CEILOMETER_ID=$(get_id keystone service-create --name ceilometer --type metering \
            --description 'Telemetry Service')
        keystone endpoint-create --region RegionOne --service-id $CEILOMETER_ID \
            --publicurl 'http://controller:8777' \
            --adminurl 'http://controller:8777' \
            --internalurl 'http://controller:8777'

        service ceilometer-agent-central restart
        service ceilometer-agent-notification restart
        service ceilometer-api restart
        service ceilometer-collector restart
        service ceilometer-alarm-evaluator restart
        service ceilometer-alarm-notifier restart
    }

    deploy_swift () {
        [ -z "$P1" -o -z "$P2" -o -z "$P3" ] && echo "P1, P2 et P3 ne sont pas définies" && exit 1

        chown -R root:root /root/.ssh

        scp root@controller:admrc .
        . ./admrc

        apt update
        $APTINSTALL python-openstackclient

        openstack user create --password swiftkeypass swift
        openstack role add --project service --user swift admin
        openstack service create --name swift \
            --description "OpenStack Object Storage" object-store
        openstack endpoint create --region RegionOne \
            object-store public http://controller:8080/v1/AUTH_%\(tenant_id\)s
        openstack endpoint create --region RegionOne \
            object-store internal http://controller:8080/v1/AUTH_%\(tenant_id\)s
        openstack endpoint create --region RegionOne \
            object-store admin http://controller:8080/v1/AUTH_%\(tenant_id\)s

        $APTINSTALL swift-account swift-container swift-object swift-object-expirer xfsprogs rsync swift
        umount $P1 || true
        umount $P2 || true
        umount $P3 || true

        grep -q '/dev/' $P1 || dd if=/dev/zero of=$P1 bs=1M count=5000
        mkfs.xfs -f $P1
        mkdir -p /srv/node/d1
        mount -t xfs $P1 /srv/node/d1
        grep -q '/dev/' $P2 || dd if=/dev/zero of=$P2 bs=1M count=5000
        mkfs.xfs -f $P2
        mkdir -p /srv/node/d2
        mount -t xfs $P2 /srv/node/d2
        grep -q '/dev/' $P3 || dd if=/dev/zero of=$P3 bs=1M count=5000
        mkfs.xfs -f $P3
        mkdir -p /srv/node/d3
        mount -t xfs $P3 /srv/node/d3
        chown -R swift:swift /srv/node

        mkdir -p /etc/swift
        cp configs/swift.conf /etc/swift/
        cp configs/proxy-server.conf /tmp/
        cp configs/rsyncd.conf /etc/
        for f in account-server.conf container-server.conf object-server.conf \
                container-reconciler.conf object-expirer.conf; do
            cp configs/$f /etc/swift
        done
        sed -i 's/^RSYNC_ENABLE=.*/RSYNC_ENABLE=true/' /etc/default/rsync
        service rsync start

        cd /etc/swift
        swift-ring-builder account.builder create 14 3 1
        swift-ring-builder container.builder create 14 3 1
        swift-ring-builder object.builder create 14 3 1
        swift-ring-builder object.builder add z1-192.168.122.1:6000/d1 100
        swift-ring-builder object.builder add z1-192.168.122.1:6000/d2 100
        swift-ring-builder object.builder add z1-192.168.122.1:6000/d3 100
        swift-ring-builder container.builder add z1-192.168.122.1:6001/d1 100
        swift-ring-builder container.builder add z1-192.168.122.1:6001/d2 100
        swift-ring-builder container.builder add z1-192.168.122.1:6001/d3 100
        swift-ring-builder account.builder add z1-192.168.122.1:6002/d1 100
        swift-ring-builder account.builder add z1-192.168.122.1:6002/d2 100
        swift-ring-builder account.builder add z1-192.168.122.1:6002/d3 100
        swift-ring-builder account.builder rebalance
        swift-ring-builder container.builder rebalance
        swift-ring-builder object.builder rebalance

        chown -R swift:swift /etc/swift

        ssh root@controller apt update
        ssh root@controller $APTINSTALL swift swift-proxy memcached
        ssh root@controller mkdir -p /etc/swift
        scp /tmp/proxy-server.conf root@controller:/etc/swift/
        scp /etc/swift/swift.conf root@controller:/etc/swift/
        scp /etc/swift/*.gz root@controller:/etc/swift
        ssh root@controller chown -R swift:swift /etc/swift
        ssh root@controller swift-init proxy restart || true
        sleep 5
        swift-init all restart
    }

    deploy_networks () {
        . ~/admrc
        ADMIN_ID=$(openstack project list | awk '/ admin / { print $2 }')
        neutron net-create public \
            --provider:network-type flat --provider:physical_network physnet1 \
            --router:external \
            --tenant-id $ADMIN_ID
        neutron subnet-create public --name ext-subnet \
            --allocation-pool start=192.168.124.100,end=192.168.124.254 \
            --gateway 192.168.124.1 --disable-dhcp 192.168.124.0/24 \
            --tenant-id $ADMIN_ID

        neutron net-create demo-net1
        neutron subnet-create --ip-version 4 --name demo-subnet1 \
            --dns-nameserver 8.8.8.8 demo-net1 10.5.5.0/24
        neutron router-create router1
        neutron router-interface-add router1 demo-subnet1
        neutron router-gateway-set router1 public
    }

    for arg in "$@"; do
        deploy_$arg
    done
