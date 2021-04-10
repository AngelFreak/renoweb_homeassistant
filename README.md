
# Renoweb / Affaldsportal in Homeassistant

## Intro / udfordring

Min kommune udstiller ikke selv noget API til rådighed jeg kan hive data fra, kun en statisk PDF kalender, eller en CSV fil. 

Tilgengæld har de et sammenarbejde med et firma som laver en app, “Affaldsportal”. Men altså, hvem gider have en app mere ?

Deres API var “desværre” beskyttet, og jeg skulle finde en key at sende med mine API kald. Så jeg kom frem til at den måtte jeg kunne få ud af deres app. 

Det var dog ikke helt nemt, da de har beskyttet deres HTTPS endpoints i appen, nok med  HPKP. Meeen med Android emulation, Fiddler og certificate pinning med Frida lykkedes det til sidst.

```
API URL: https://services.renoweb.dk/v1_13/AFP2/GetAffaldsportal2Config.aspx?
API key: 346B43B0-D1F0-4AFC-9EE8-C4AD1BFDC218
APP identifier: DDDD4A1D-DDD1-4436-DDDD-3F374DD683A1
```

Så nu er det “bare” at lave forespørgsler til deres API, og finde din Kommunes data struktur.

Datastrukturen er nok ikke ens i alle kommuner, men jeg vil tage udgangspunkt i min egen. 

Dette kan med fordel gøres med Postman (https://www.postman.com/downloads/), som er et vældig godt API værktøj.

## Procedure

For at få en liste over kommuner, og deres ID skal man lave en GET request til: 
https://services.renoweb.dk/v1_13/AFP2/GetAffaldsportal2Config.aspx?appidentifier=DDDD4A1D-DDD1-4436-DDDD-3F374DD683A1
Alternativt er der en liste med ID'er nederst på siden, men den er muligvis ikke opdateret.

Den værdi som skal bruges er "municipalitycode"

![Image Init Postman request](https://github.com/AngelFreak/renoweb_homeassistant/blob/main/postman_init_request.png)

Herefter laves et request medmunicipalitycode på https://servicesgh.renoweb.dk/v1_13/GetJSONAdress.aspx?municipalitycode=259&apikey=346B43B0-D1F0-4AFC-9EE8-C4AD1BFDC218, men fik dette svar
```json
{"status": {"id": 2,"status": null,"msg": "The roadid is empty."}} Jeg mangler altså et roadid.
```

Heldigvis kan der gives et roadname parameter, som returnere et id. https://servicesgh.renoweb.dk/v1_13/GetJSONRoad.aspx?municipalitycode=259&apikey=346B43B0-D1F0-4AFC-9EE8-C4AD1BFDC218&roadname=Steinmannsvej
```json
{"status":{"id":0,"status":null,"msg":"Ok"},"list":[{"id":782,"name":"Steinmannsvej (4600)","fulllist":true}]}
```

Nu hiver jeg en liste over min vej https://servicesgh.renoweb.dk/v1_13/GetJSONAdress.aspx?municipalitycode=259&apikey=346B43B0-D1F0-4AFC-9EE8-C4AD1BFDC218&roadid=782&showall=1 finder min adresses id, og laver det endelige kald https://servicesgh.renoweb.dk/v1_13/GetJSONContainerList.aspx?municipalitycode=259&apikey=346B43B0-D1F0-4AFC-9EE8-C4AD1BFDC218&adressId=7650&fullinfo=1&supportsSharedEquipment=1

Nu har jeg en fin liste der er struktureret således.
```json
{"status": {"id": 0,"status": null,"msg": "Ok"},"list": [{"id": 38283,"rwadrid": 7650,"customerid": 18723,"name": "Storskrald stk/STORSKRALD","count": "1","module": {"id": 2,"name": "Storskrald","fractionname": "Storskrald"
```

![Image Final Postman request](https://github.com/AngelFreak/renoweb_homeassistant/blob/main/postman_final_request.png)

Min sensor kode er derfor lavet ud fra hoved elementet “list”, din er måske anderledes.

Sensor konfiguration kan findes i dette repo: https://github.com/AngelFreak/renoweb_homeassistant/blob/main/sensor.yml
Konfiguration af card med sensorerne kan i dette repo: https://github.com/AngelFreak/renoweb_homeassistant/blob/main/lovelace.yml

Det endelige resultat ser således ud:

![Image Hassio card](https://github.com/AngelFreak/renoweb_homeassistant/blob/main/hassio_garbage_card.png)

Det kan gøres pænere, men min HA kode er ikke skrevet til deling (endnu).
Sig til hvis nogle har forslag til ændringer, forbedringer eller forslag.

## Kreditering
Dette er skrevet med stor! inspiration fra Esben, og Jacob.

Jeg havde mange af de samme ønsker som Esben, og store dele af denne proces er tyv stjålet fra ham. Igen tak Esben.

For hans oprindelige skriv klik her: https://www.facebook.com/notes/dansk-home-assistant-gruppe/affaldgenbrug-afhentning-via-renovationselskabets-api/645979892637386/

## Supporterede kommuner
Listen er opdateret: 10-04-2021 19:30:00

```txt
Allerød- ID: 201
Brøndby- ID: 153
Dragør - ID: 155
Egedal - ID: 240
Esbjerg - ID: 561
Fredensborg - ID: 210
Frederikssund - ID: 250
Gentofte - ID: 157
Gladsaxe - ID: 159
Glostrup - ID: 161
Greve - ID: 253
Halsnæs - ID: 260
Helsingør - ID: 217
Herlev - ID: 163
Hillerød - ID: 219
Hvidovre - ID: 167
Høje-Taastrup - ID: 169
Køge - ID: 259
Lejre - ID: 350
Lyngby-Taarbæk - ID: 173
Rebild - ID: 840
Ringkøbing-Skjern - ID: 760
Roskilde - ID: 265
Rudersdal - ID: 230
Solrød - ID: 269
Stevns - ID: 336
Svendborg - ID: 479
Sweco Demo - ID: 10210
Sweco Test B - ID: 16161
Sweco Test T - ID: 20161
Tårnby - ID: 185
Vejen - ID: 575
Aalborg - ID: 851
```
