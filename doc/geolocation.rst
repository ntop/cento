Geolocation Database
====================

nProbe Cento includes Geolocation support provided by the following companies

- MaxMind https://www.maxmind.com
- DB-IP https://db-ip.com

The geolocation implementation is based on a database file stored locally with no cloud access whatsoever.

You can choose to install the free (albeit not very accurate) GeoIP databases or the commercial ones.

Using MaxMind geolocation
-------------------------

New privacy regulations, such as GDPR and CCPA, place restrictions that impact our ability to distre MaxMind GeoLite2 databases in our packages.
Reasons are explained in detail at the following page https://blog.maxmind.com/2019/12/18/significant-changes-to-accessing-and-using-geolite2-databases/.
For this reason you are required to register for a MaxMind account and obtain a license key in order to download GeoLite2 geolocation databases.

The steps below are necessary to use geolocation.

Register for a MaxMind account at https://www.maxmind.com/en/geolite2/signup.

Create a license key at https://www.maxmind.com/en/accounts/current/license-key.

1. Select "Generate New License Key".
2. Add a license key description and answer "Yes" to the question "Will this key be used for GeoIP Update?".
3. Then choose one of the two available options "Generate a license key and config file". Choice depends on the installed `geoipupdate` version. Most likely, installed version is older than 3.1.1 so the correct option to select is "Generate a license key and config file for use with `geoipupdate` versions older than 3.1.1". If you don't know the version type `geoipupdate -V`.

Once the license is created, you will be promted to download file `GeoIP.conf` which contains account id and license key necessary to download the databases. Download and place this file in `/etc/GeoIP.conf`.

Make sure that the `EditionIDs` section (or `ProductIds` according to the `geoipupdate` version) in `/etc/GeoIP.conf` contains `GeoLite2-Country GeoLite2-City GeoLite2-ASN` (`GeoLite2-ASN` could be missing by default).

Run `sudo geoipupdate` to download the database files or manually download database files
`GeoLite2-ASN.mmdb` and `GeoLite2-City.mmdb` from the "GeoIP2 / GeoLite2" > "Download Files" section of your MaxMind account page

In order to update the database you can use `geoipupdate`. Instructions to use `geoipupdate` are available at https://dev.maxmind.com/geoip/geoipupdate/

Using DB-IP Lite
----------------

DB-IP Lite geolocation databseses `dbip-city-lite`, `dbip-asn-lite` and `dbip-country-lite` can be downloaded from https://db-ip.com/db/

Install DB files
----------------

Downloaded database files (mmdb) should be moved under one of the below folders:

- /var/lib/GeoIP
- /usr/share/GeoIP
- ./geoip (when running nProbe Cento manually, same folder)

And their name should match:

- GeoLite2-ASN.mmdb
- GeoLite2-City.mmdb

or

- dbip-asn*.mmdb
- dbip-city*.mmdb

.. code-block:: console

   mkdir /var/lib/GeoIP
   mv GeoLite2-ASN.mmdb /var/lib/GeoIP
   mv GeoLite2-City.mmdb /var/lib/GeoIP

Restart nProbe Cento. Upon restart, software will automatically locate and load the downloaded databases.

