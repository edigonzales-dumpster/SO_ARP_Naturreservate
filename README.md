# SO_ARP_Naturreservate

## Datenumbau (Ergänzen mit UUID)
Existierende Docker-DB:
```
docker run --rm --name grundstuecksinformation-db -p 54321:5432 --hostname primary -e PG_DATABASE=grundstuecksinformation -e PG_LOCALE=de_CH.UTF-8 -e PG_PRIMARY_PORT=5432 -e PG_MODE=primary -e PG_USER=admin -e PG_PASSWORD=admin -e PG_PRIMARY_USER=repl -e PG_PRIMARY_PASSWORD=repl -e PG_ROOT_PASSWORD=secret -e PG_WRITE_USER=gretl -e PG_WRITE_PASSWORD=gretl -e PG_READ_USER=ogc_server -e PG_READ_PASSWORD=ogc_server -v ~/pgdata-grundstuecksinformation:/pgdata:delegated sogis/oereb-db:latest
```

```
java -jar /Users/stefan/apps/ili2pg-4.3.1/ili2pg-4.3.1.jar --dbhost localhost --dbdatabase grundstuecksinformation --dbport 54321 --dbusr admin --dbpwd admin --disableValidation --nameByTopic --createTidCol --importTid --strokeArcs --sqlEnableNull --models SO_ARP_Naturreservate_20180829 --defaultSrsCode 2056 --doSchemaImport --dbschema arp_naturreservate --import arp_naturreservate_20200630.xtf
```


```
UPDATE arp_naturreservate.reservate_dokument SET t_ili_tid = uuid_generate_v4();
UPDATE arp_naturreservate.reservate_pflanze SET t_ili_tid = uuid_generate_v4();
UPDATE arp_naturreservate.reservate_reservat SET t_ili_tid = uuid_generate_v4();
UPDATE arp_naturreservate.reservate_reservat_dokument SET t_ili_tid = uuid_generate_v4();
UPDATE arp_naturreservate.reservate_reservat_zustaendiger SET t_ili_tid = uuid_generate_v4();
UPDATE arp_naturreservate.reservate_teilgebiet SET t_ili_tid = uuid_generate_v4();
UPDATE arp_naturreservate.reservate_teilgebiet_pflanze SET t_ili_tid = uuid_generate_v4();
UPDATE arp_naturreservate.reservate_zustaendiger SET t_ili_tid = uuid_generate_v4();
```
(Das Update bei den Zwischentabellen (Assoziationen) wäre nicht notwendig)


```
java -jar /Users/stefan/apps/ili2pg-4.3.1/ili2pg-4.3.1.jar --dbhost localhost --dbdatabase grundstuecksinformation --dbport 54321 --dbusr admin --dbpwd admin --disableValidation --exportTid --models SO_ARP_Naturreservate_20180829 --defaultSrsCode 2056 --dbschema arp_naturreservate --export arp_naturreservate_20200630_uuid.xtf

xmllint --format arp_naturreservate_20200630_uuid.xtf -o arp_naturreservate_20200630_uuid.xtf
```

## Testen mit neuem Modell

Modellnamen im XTF anpassen.

Ili2pg-Befehl von `daten_tools` und an meine Dev-Umgegung angepasst und zuerst mit `--sqlEnableNull`, damit die neuen `MANDATORY`-Attribute nicht stören:

```
java -jar /Users/stefan/apps/ili2pg-4.3.1/ili2pg-4.3.1.jar  --dbhost localhost --dbdatabase grundstuecksinformation --dbport 54321 --dbusr admin --dbpwd admin \
--dbschema arp_naturreservate_v2 --models SO_ARP_Naturreservate_20200609 --modeldir 'http://models.geo.admin.ch/;http://geo.so.ch/models;' \
--defaultSrsCode 2056 --strokeArcs --createGeomIdx --createFk --createFkIdx --createEnumTabs --beautifyEnumDispName --createMetaInfo --createUnique --createNumChecks --nameByTopic \
--createTidCol \
--disableValidation --sqlEnableNull --doSchemaImport --import arp_naturreservate_20200630_uuid_modellnamen.xtf
```

Dummy-Werte für die neuen `MANDATORY`-Attribute abfüllen:

```
UPDATE arp_naturreservate_v2.reservate_reservat SET einzelschutz = true;
```

Wieder exportieren:

```
java -jar /Users/stefan/apps/ili2pg-4.3.1/ili2pg-4.3.1.jar --dbhost localhost --dbdatabase grundstuecksinformation --dbport 54321 --dbusr admin --dbpwd admin --disableValidation --exportTid --models SO_ARP_Naturreservate_20200609 --defaultSrsCode 2056 --dbschema arp_naturreservate_v2 --export arp_naturreservate_20200630_uuid_definitiv.xtf
```

Ilivalidator meldet Fehler weil Assoziationen doppelt vorkommen. Da glaube ich aber, dass es eine Fehler von ilivalidator ist:
```
java -jar /Users/stefan/apps/ilivalidator-1.11.6/ilivalidator-1.11.6.jar --trace arp_naturreservate_20200630_uuid_definitiv.xtf
```

Nochmals importieren (mit `--disableValidation` aber ohne `--sqlEnableNull`):

```
java -jar /Users/stefan/apps/ili2pg-4.3.1/ili2pg-4.3.1.jar  --dbhost localhost --dbdatabase grundstuecksinformation --dbport 54321 --dbusr admin --dbpwd admin \
--dbschema arp_naturreservate_v2 --models SO_ARP_Naturreservate_20200609 --modeldir 'http://models.geo.admin.ch/;http://geo.so.ch/models;' \
--defaultSrsCode 2056 --strokeArcs --createGeomIdx --createFk --createFkIdx --createEnumTabs --beautifyEnumDispName --createMetaInfo --createUnique --createNumChecks --nameByTopic \
--createTidCol \
--disableValidation --sqlEnableNull --doSchemaImport --import arp_naturreservate_20200630_uuid_modellnamen.xtf
```

