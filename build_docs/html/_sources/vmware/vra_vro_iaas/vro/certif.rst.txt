List certificate
================

keytool -list -keystore /etc/vco/app-server/security/jssecacerts -v -alias vco.sso.ssl.certificate  -storepass $PASS
keytool -list -keystore /etc/vco/app-server/security/jssecacerts -v -alias vco.cafe.component-registry.ssl.certificate -storepass $PASS 

Delete certificate
==================

keytool -delete -keystore /etc/vco/app-server/security/jssecacerts -v -alias vco.sso.ssl.certificate  -storepass $PASS
keytool -delete -keystore /etc/vco/app-server/security/jssecacerts -v -alias vco.cafe.component-registry.ssl.certificate -storepass $PASS
 
Reimport certificate
=====================

/var/lib/vco/tools/configuration-cli/bin/vro-configure.sh trust --registry-certificate --uri https://<cafe_uri>
/var/lib/vco/tools/configuration-cli/bin/vro-configure.sh trust --sso-certificate --uri https://<cafe_uri>