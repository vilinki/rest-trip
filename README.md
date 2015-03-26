Arkiston resti
==============
- Arkiston REST-rajapinta antaa lukupääsyn arkiston sisältöihin.
- Käyttöoikeudet tarkistetaan Oauth / basic -tunnistuksella. Käyttöoikeudet arkiston käyttäjätunnuksien mukaisesti.
- Jokaisen kutsun mukana on välitettävä api-key

Root endpoints
==============
GET /apis/rest
```http
GET /apis/rest 
HTTP/1.1 200 OK
Content-Type: application/json
{
  "response": {
    "name": "archive-api",
    "description": "REST API for archive access",
    "version": "v1.0",
    "referenceAlmaSpec": "v1.0.0",
    "status": "current"
  }
}
```
GET /apis/rest/v1.0
```http
GET /apis/rest/v1.0
HTTP/1.1 200 OK
Content-Type: application/json
{
  "response": {
    "name": "archive-api",
    "description": "REST API for archive access",
    "version": "v1.0",
    "referenceAlmaSpec": "v1.0.0",
    "status": "current"
  }
}
```

General info
============
GET /apis/rest/v1.0/datasources
```http
GET /apis/rest/v1.0/datasources
HTTP/1.1 200 OK
Content-Type: application/json
{
  "response": [
    {
      "name": "KAL1415TX",
      "type": "xml", 
      "records": "53015",
      "last_indexed": "20150326T08:01:22",
      "last_updated": "20150326T08:01:20",
      "formats": ["xml_only","ma","ma2","nitf","jsonrec"]                      
    },
    {
      "name": "KIL1415T",
      "type": "text", 
      "records": "14060",
      "last_indexed": "20150325T23:14:06",
      "last_updated": "20150325T23:13:55",
      "formats": ["all_fields","ma","ma2","nitf","jsonrec"]                      
    }
  ]
}
```
GET /apis/rest/v1.0/datasources/list?format=xml
```http
GET /apis/rest/v1.0/datasources/list?format=xml
HTTP/1.1 200 OK
Content-Type: text/xml

<?xml version="1.1" encoding="UTF-8"?>
<response>
  <database>
     <name>KAL1415TX</name>
     <type>xml</type> 
     <records>53015</records>   
     <last_indexed>20150326T08:01:22</last_indexed>
     <last_updated>20150326T08:01:20</last_updated> 
     <formats>
       <name>xml_only</name>
       <name>ma</name>
       <name>ma2</name>
       <name>nitf</name>
       <name>jsonrec</name>                    
      </formats>         
  </database>
  <database>
     <name>KIL1415T</name>
     <type>text</type> 
     <records>14060</records>   
     <last_indexed>20150325T23:14:06</last_indexed>
     <last_updated>20150325T23:13:55</last_updated> 
     <formats>
       <name>all_fields</name>
       <name>ma</name>
       <name>ma2</name>
       <name>nitf</name>
       <name>jsonrec</name>                    
      </formats>         
  </database>
</response>
```
GET /apis/rest/v1.0/datasources/kil1415t
```http    
GET /apis/rest/v1.0/datasources/kil1415t
HTTP/1.1 200 OK
Content-Type: application/json
{
  "response": [
    {
      "name": "KIL1415T",
      "type": "text", 
      "records": "14060",
      "last_indexed": "20150326T08:01:22",
      "last_updated": "20150326T08:01:20",
      "formats": ["xml_only","ma","ma2","nitf","jsonrec"],
      "fields": [
       {"number": "1","name": "OTSIKKO","type": "phrase","indexed": "yes","description": "Artikkelin otsikko","part_record_field": "no"},
       {"number": "2","name": "SIVU","type": "number","indexed": "yes","description": "Sivunumero","part_record_field": "no"},
       {"number": "3","name": "JULKAISU","type": "phrase","indexed": "yes","description": "Julkaisun nimi","part_record_field": "no"},       
       {"number": "4","name": "LEIPATEKSTI","type": "text","indexed": "yes","description": "Artikkelin leipäteksti","part_record_field": "no"},     
      ]                      
    }
  ]
}
```
GET /apis/rest/v1.0/datasources/kil1415t/info?format=json
```http 
GET /apis/rest/v1.0/datasources/kil1415t/info?format=json
HTTP/1.1 200 OK
Content-Type: application/json
{
  "response": [
    {
      "name": "KIL1415T",
      "type": "text", 
      "records": "14060",
      "last_indexed": "20150326T08:01:22",
      "last_updated": "20150326T08:01:20",
      "formats": ["xml_only","ma","ma2","nitf","jsonrec"],
      "fields": [
       {"number": "1","name": "OTSIKKO","type": "phrase","indexed": "yes","description": "Artikkelin otsikko","part_record_field": "no"},
       {"number": "2","name": "SIVU","type": "number","indexed": "yes","description": "Sivunumero","part_record_field": "no"},
       {"number": "3","name": "JULKAISU","type": "phrase","indexed": "yes","description": "Julkaisun nimi","part_record_field": "no"},       
       {"number": "4","name": "LEIPATEKSTI","type": "text","indexed": "yes","description": "Artikkelin leipäteksti","part_record_field": "no"},     
      ]                      
    }
  ]
}
```
List of methods
===============
GET /apis/rest/{version}/datasources/{database}/records
GET /apis/rest/{version}/datasources/{database}/records/list?format=csv,txt,xml,json&count=x&sort=y
GET /apis/rest/{version}/datasources/{database}/records/{id}/content?format=csv,txt,xml,json
