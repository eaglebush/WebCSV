# WebCSV:  A Proposal to Use CSV for Web Data Interchange

## Introduction

A CSV or Comma-Separated Values file has been with us since the 1970s and pre-dates personal computers by more than a decade. 
It is a tabular data format that includes rows and columns. It is written in plain text. In its simplest form, it only
needs a comma to separate values and a carriage return (Enter key) to indicate a complete line.

In this internet and cloud-enabled times, we have standardized structured data interchange formats to enable computing machines to talk to each other.

XML, or eXtensible Markup Language is designed to emphasize simplicity, generality, and usability across the Internet. 
An XML file is expected to describe what data is inside it by using tags, attributes and node hierachy. The values are mostly contained
inside their nodes. To know more about XML, please [read](https://www.w3.org/XML/) it on W3C website.

JSON, or Javascript Object Notation is also one of the popular internet exchange formats. A JSON file, just like XML, also describes the 
data it contains by including its name in opposite of its value. It is also a structured data interchange format. To know more about JSON,
please [read](https://www.json.org/json-en.html) on their website.

Both XML and JSON are human and machine readable files. This means that you can read the parameters and values inside it, and you can also write it
in any text processor you can get your hands on. Both formats can be validated for completeness. For XML files, a basic validation could be done by 
looking the root opening and end tags if it is present. For JSON files, a basic validation could be done by looking the opening and ending curly braces.
Both needs to have a parser and a syntax checker to safely flag a file as valid. 

While both can be syntactically valid, data structure validation is also required to check the integrity of the document. 
This is to ensure that the file conforms to the expected data structure, aside from identifying what the file was for.
XML has document type definition (DTD) and XML Schema Definition (XSD). On the other hand, JSON, earlier has no data structure validation. 
It was thought that the structure validation should be up to the implementation of the application that produces and consumes it. 
Recent developments about a JSON schema suggests that a new draft was just [released](https://json-schema.org/draft/2019-09/json-schema-core.html).

Compared to CSV files, both XML and JSON has to include enclosing identification characters or tags. 
An XML file, discounting the file header, need at least three (3) characters to open and at least four (4) characters to close. 
Each node down the hierarchy needs to have these closing tags.
A JSON file, being with no file header, needs one character to open (`{`) and another character (`}`) to close.
Each value being represented needs to have a name, enclosed in double quotes and terminated in colon. 
If the value is a text type, it also needs to be enclosed in double quotes.
CSV, only needs to be delimited by commas for its columns and carriage returns for its rows. 
A full CSV implementation would only need double quotes to enclose values that includes the delimiter character, by default, a comma.

When it comes to byte savings on disk and on the wire, CSV comes first, JSON and then XML. 

This is how the idea of using CSV as an alternative data interchange format came up.

Other data interchange formats has been introduced recently to minimize data traffic. 
These are Google's [Protocol Buffers](https://developers.google.com/protocol-buffers) and Sadayuki Furuhashi's [MessagePack](https://msgpack.org/). However, these are not file formats. 
Protocol Buffers is an application layer to serialize data into a binary representation for lightweight transmission of data to be deserialized into structured objects.
MessagePack also does the same process, but it needs JSON data to be serialized and deserialized.

## WebCSV

Web Character-Separated Values or WebCSV is the proposed name for this specification. 
CSV, which casually means Comma-Separated Values, can sometimes use a different character for a delimiter. 
The tab and pipe characters are commonly used to replace comma. 
This is the reason why we need to change the `Comma` in the CSV as `Character`. 
`Web` is prepended to the name to signify where it should primarily be used.

## WebCSV and REST

WebCSV is intended to be used for RESTful applications. It can readily be adapted for storing and reading data just like how we use JSON and XML for web services.
Just like JSON APIs, the transport should be secured (SSL) because the data is human readable.


## Structure

Because a CSV structure can be loose, a lightweight schema must be included. 

The structure is proposed as follows:

- Schema
- Body

### Schema

The WebCSV schema is sent through an HTTP custom header in a REST request. Any header could be set to send this schema for a request, 
but a new custom header `Content-Schema` could be used to be appropriate.

> Note: 
> To save on data, a client program could store the schema in memory and forego with the header request
> by setting "Content-Schema" to None. The key of the map where the schema is stored could be the
> endpoint and the path.

A web service should set this header to the schema format defined below:

>`ver:<any>,hdr<true/false>,del:<character>; Column1:<type>,Column2:<type>,Column3:<type>`

A working example is shown below:

>`ver:1.0,hdr:false,del:,; LastName:string(50),FirstName:string(50),MiddleName:string(50),Age:int,Height:decimal(13,3),Weight:decimal(13,3),Alive:bool,DateBorn:date,LastUpdated:datetime`

There are two parts of the schema. The **schema information** and the **column information**. 
The first part of the header value is the basic information of the data header. The rest are the column information. 
The schema parts are delimited by a semicolon (`;`).

#### Schema Information

The *schema information* has basic required data:

- `ver` - Version information. This can be either a [semver](https://semver.org/) or just plain version indication.
- `hdr` - Data includes column header (for flexibility reasons). Defaults to none. No `hdr` tag would mean the data does not contain column header
- `del` - Delimiter. The delimiter tag should instruct the client to use the delimiter specified. This defaults to comma `,`. No del tag would mean it is a comma.

Example:

> `ver:1.0,hdr:false,del:\t;`

Where version is `1.0`, hdr is not included and `tab` delimited.

Custom headers can also be added. An application can be further modified to retrieve and produce the headers. Some of the suggested headers are:

- `app` - Application id or name
- `rdt` - Release or last modification date
- `prs` - Parsing type. Simple CSV or Full CSV file. Simple CSV expects values not to contain commas.

#### Column Information

The *column information* is defined as follows:

>  `Column1:<type>,Column2:<type(length)>,Column3:<type(precision,scale)>`

The currently supported column types are `string`, `int`, `bool`, `decimal`, `date` and `datetime`.

- For `string`s, a length can be defined inside a parenthesis to indicate a limit on the length of the text that a column could handle. If the length is omitted, the length will be set to 4000 bytes.
- For `decimal`, the precision and scale should be defined with two values inside a parenthesis separated by a comma.
- For `bool`, `int`, `date` and `datetime`, no further attributes are required.

> Note:
> A schema column information can omit the column name, leaving the data type alone

As a side-effect of the column information, the data type for a database table structure could be adapted easily. 
The column definitions could be used to create a simple table in a database.

### Body

The body comes the payload of an HTTP request. The payload is a regular CSV data format. Any CSV parser that can handle quoted strings should have no problem parsing it.
The CSV column headers should be removed if the `hdr` schema flag is set to false. To validate the parsed data, the schema column information can be used. 
			
Sample: 

```
Pike,Robert,C,63,8.7,60.6,true,1956-10-08,2020-04-08T14:00:00Z
Griesemer,Robert,C,25,8.7,8.9,false,1995-09-01,2020-04-08T14:00:00Z
Smith,John,Porter,65,6.7,6.8,true,1955-08-08,2020-04-08T14:00:00Z
Chi,Kwan,Tai,35,7.7,20.9,true,1985-11-08,2020-04-08T14:00:00Z
```
	
## Features

- Only the data will be sent through the body. This will *save* a lot of bytes on HTTP transport
- The data columns could be matched against the schema columns sent in the headers on the fly, down to validation of expected data from a loaded desired schema
- The schema could be mapped to the database structure. It allows the developer to create a table for the CSV data.
- The data could be a file download. It can directly be stored in a client computer by specifying **Content-Disposition** header
- The data could be streamed. Each row could be sent one after another until it ends. There is no need to wait for the whole structure to load before parsing
- The data could be read and written by humans

## Limitations

- The schema and data could not carry hierchical data

## Comparisons

If we only wanted to retrieve the essential data, the above mentioned data formats would yield the number of bytes from a CSV sample. Please see comparisons below:

CSV: (**258** bytes)

```
Pike,Robert,C,63,8.7,60.6,true,1956-10-08,2020-04-08T14:00:00Z
Griesemer,Robert,C,25,8.7,8.9,false,1995-09-01,2020-04-08T14:00:00Z
Smith,John,Porter,65,6.7,6.8,true,1955-08-08,2020-04-08T14:00:00Z
Chi,Kwan,Tai,35,7.7,20.9,true,1985-11-08,2020-04-08T14:00:00Z
```
		
JSON: (prettified: **965** bytes, minified: **680** bytes ) 
```json
[
    { "lastname": "Pike", "firstname": "Robert", "middlename": "F", "age": 63, "height": 8.7, "weight": 60.6, "alive": true, "dateborn": "1956-10-08", "lastupdated": "2020-04-08T14:00:00Z" },
    { "lastname": "Griesemer", "firstname": "Robert", "middlename": "U", "age": 25, "height": 8.7, "weight": 8.9, "alive": false, "dateborn": "1995-09-01", "lastupdated": "2020-04-08T14:00:00Z" },
    { "lastname": "Smith", "firstname": "John", "middlename": "Porter", "age": 65,"height": 6.7, "weight": 6.8,"alive": true, "dateborn": "1955-08-08", "lastupdated": "2020-04-08T14:00:00Z" },
    { "lastname": "Chi", "firstname": "Kwan", "middlename": "Tai", "age": 35, "height":7.7, "weight":20.9, "alive": true, "dateborn": "1985-11-08", "lastupdated": "2020-04-08T14:00:00Z"}
]
```

XML: (prettified: **1,058**, minified: **1,344**)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<root>
   <element>
      <age>63</age>
      <alive>true</alive>
      <dateborn>1956-10-08</dateborn>
      <firstname>Robert</firstname>
      <height>8.7</height>
      <lastname>Pike</lastname>
      <lastupdated>2020-04-08T14:00:00Z</lastupdated>
      <middlename>F</middlename>
      <weight>60.6</weight>
   </element>
   <element>
      <age>25</age>
      <alive>false</alive>
      <dateborn>1995-09-01</dateborn>
      <firstname>Robert</firstname>
      <height>8.7</height>
      <lastname>Griesemer</lastname>
      <lastupdated>2020-04-08T14:00:00Z</lastupdated>
      <middlename>U</middlename>
      <weight>8.9</weight>
   </element>
   <element>
      <age>65</age>
      <alive>true</alive>
      <dateborn>1955-08-08</dateborn>
      <firstname>John</firstname>
      <height>6.7</height>
      <lastname>Smith</lastname>
      <lastupdated>2020-04-08T14:00:00Z</lastupdated>
      <middlename>Porter</middlename>
      <weight>6.8</weight>
   </element>
   <element>
      <age>35</age>
      <alive>true</alive>
      <dateborn>1985-11-08</dateborn>
      <firstname>Kwan</firstname>
      <height>7.7</height>
      <lastname>Chi</lastname>
      <lastupdated>2020-04-08T14:00:00Z</lastupdated>
      <middlename>Tai</middlename>
      <weight>20.9</weight>
   </element>
</root>
```

As you can see, when the essential data goes up, the data descriptors in JSON and XML also goes up. The CSV sample data is the leanest of them all.
				
## Proof of Concept

This proposal specification includes a server programmed in Go to better explain the implementation details. A Postman collection for CRUD sample requests is also included.

Sample web service: [https://github.com/eaglebush/WebCSV](https://github.com/eaglebush/WebCSV)

Postman request collection: [https://github.com/eaglebush/WebCSV](https://github.com/eaglebush/WebCSV/tree/master/client)

## Possible Solution for Hierchical Data

To simplify the proposal, the Limitation section stated that the CSV data format cannot handle hierchical data. But by expanding the schema and sub-parsing, the mechanism to handle hierchical data in a WebCSV can be done.

### Add another part on the schema by adding a sub-schema

#### Schema

>`ver:1.0,hdr:false,del:\t; LastName:string(50),FirstName:string(50),MiddleName:50,Age:int,Location:Coordinates; Coordinates:Lng:decimal(15,4),Lat:decimal(15,4)`
    
The following schema has the following columns:
    
- `LastName` - string with a length of 50
- `FirstName` - string with a length of 50
- `MiddleName` - string with a length of 50
- `Age` - integer
- `Location` - `Coordinates`. The structure for the coordinate is validated on the third part of the schema.
				
#### Data
		
CSV supports any printable character when it is inside the double quotes. When the data in the `Location` column is retrieved, it can be *re-parsed* with a CSV parsing function.

Example:

> `Baguinon,Elizalde,Gonzales,45,"40.730610,-73.935242"`

#### Other Posibilities
    
A schema could be an array. Multiple entries are separated by *carriage returns*:

> `ver:1.0,hdr:false,del:\t; LastName:string(50),FirstName:string(50),MiddleName:50,Age:int,Entries:Entry; Entry:Door:string(10),Time:timestamp; Entries:[]Entry`

Example:

```
    Baguinon,Elizalde,Gonzales,45, "Front,2020-04-08T14:00.00.000Z
    Back,2020-04-08T14:01.00.000Z
    Balcony,2020-04-08T14:05.00.000Z
    Back,2020-04-08T15:00.00.000Z"
    Baguinon,Elizalde,Gonzales,45, "Front,2020-05-08T04:00.00.000Z
    Back,2020-05-08T04:10.00.000Z
    Balcony,2020-05-08T14:20.00.000Z
    Back,2020-05-08T14:50.00.000Z"
```

## Acknowledgements

- [MotherEff UTF-8 string length & byte counter](https://mothereff.in/byte-counter)
- [FreeFormatter.com](https://www.freeformatter.com/json-to-xml-converter.html)
- [Web Toolkit Online](https://www.webtoolkitonline.com/xml-minifier.html)

## License
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

*Elizalde Baguinon,
April 9, 2020*