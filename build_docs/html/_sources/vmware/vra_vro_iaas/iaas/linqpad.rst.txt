Linqpad configuration
=====================

Type: WCF Data Services 5.5(OData3)
https://iaas_server_fqdn/repository/data/ManagementModelEntities.svc/
Formatter XML
Accept Invalid certificates true


Request example
===============

ProvisioningGroups.Where(x => x.GroupName == "<<owner>>")

HostReservations.Where(x=>x.HostReservationName == "<<reservation_name>>")

HostReservations.Expand("HostReservationToStorages").Where(x=>x.HostReservationName == "<<reservation_name>>")

HostReservationToStorages.Expand("HostToStorage").Where(s=>s.HostReservationToStorageID == Guid.Parse("<<guid>>"))

HostReservations.Expand("Host").Where(x=>x.HostReservationName == "<<reservation_name>>")

HostNicToReservations.Where(x => x.HostReservationID == new Guid ("<<guid>>") )

Links
=====

https://www.simplygeek.co.uk/2019/01/14/performing-create-read-update-and-delete-operations-on-vra-iaas-entity-objects/

https://docs.microsoft.com/en-us/previous-versions/dynamicsnav-2016/hh169248(v=nav.90)

https://docs.microsoft.com/en-us/aspnet/web-api/overview/odata-support-in-aspnet-web-api/using-select-expand-and-value

https://www.simplygeek.co.uk/2019/01/14/performing-create-read-update-and-delete-operations-on-vra-iaas-entity-objects/
