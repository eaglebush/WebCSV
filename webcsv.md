##WebCSV Specification Proposal
###Web Character Separated Values (CSV)

    A CSV or Comma-Separated Values file has been with us since the 1970s and pre-dates personal computers by more than a decade. 
    It is a tabular data format that includes rows and columns. It is written in plain text. In its simplest form, it only
    needs a comma to separate values and a carriage return (Enter key) to indicate a complete line to be a valid CSV file.

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

Structure:

	Content-Schema: 
	
		Example:
		
		ver:1.0,hdr:false,del:\t; LastName:string(50),FirstName:string(50),MiddleName:50,Age:int,Height:decimal(10,3),Alive:bool,DateBorn:date,LastUpdated:datetime

		1. Schema is posted and retrieved thru a custom HTTP header called "Content-Schema". 
			To save on data, a client program could store the schema in memory and forego with the header request
			by setting "Content-Schema" to None. The key of the map where the schema is stored could be the
			endpoint and the path.
			
			map["http://apis.google.com/v2/login"] = schema
			
		2. The first part of the header value is the basic information of the data header. The rest is the header schema itself.
			Schema parts are isolated by the semicolon (;) delimiter. 
			
			ver - Version information
			hdr - Data includes column header (for flexibility reasons). Defaults to none. No hdr tag would mean the data does not contain column header
			del - Delimiter. The delimiter tag should instruct the client to use the delimiter specified. This defaults to comma (,). No del tag would mean it is a comma.
			
			Custom schema information could be implemented like:
				
				app: Application ID or name
				rdt: Release or last modification date
			
		3. A schema header column can omit the data type specification. It will be interpreted as string and the length
			is unlimited
			
		4. A schema column with a length and precision could also validate the data as database validation does
		
		5. A schema header column can omit the column name, leaving the data type alone

	Body: 

		1. Body is a regular CSV data format. Any CSV parser that can handle quoted strings should have no problem
			parsing it. It should not include the column headers.
			
		Sample: 

			Pike,Robert,C,63,8.7,60.6,true,1956-10-08,2020-04-08T14:00:00Z
			Griesemer,Robert,C,25,8.7,8.9,false,1995-09-01,2020-04-08T14:00:00Z
			Smith,John,Porter,65,6.7,6.8,true,1955-08-08,2020-04-08T14:00:00Z
			Chi,Kwan,Tai,35,7.7,20.9,true,1985-11-08,2020-04-08T14:00:00Z
	
	
Features:

	- Only the data will be sent through the body. This will save a lot of bytes on HTTP transport
	- The data structure could be matched to the schema sent in the headers
	- The schema could be optional. Some CSV parsers could automatically detect the data type implied by the columns in the data.
	- The data could be a file download. It can directly be stored in a client directory by specifying Content-Disposition header
	- The data could be streamed. Each row could be sent one after another until it ends. There is no need to wait for the whote structure to load before parsing.
	- The data could be read by humans
	- The schema could be mapped to the database structure. It allows the developer to create a table for the CSV data.
	- This specification could be RESTful.
	
Limitations:

	- The schema and data could not carry hierchical data
		
	
Possible Solution for Hierchical Data:

	- Add another part on the schema by adding a sub-schema:
		Schema:
		
			ver:1.0,hdr:false,del:\t; LastName:string(50),FirstName:string(50),MiddleName:50,Age:int,Location:Coordinates; Coordinates:Lng:decimal(15,4),Lat:decimal(15,4)
			
			The following schema has the following columns:
			
				- LastName - string with a length of 50
				- FirstName - string with a length of 50
				- MiddleName - string with a length of 50
				- Age - integer
				- Location - Coordinates. The structure for the coordinate is validated on the third part of the schema.
				
		Data:
		
			CSV supports any character when it is inside the double quotes. 
			When the data in the Location column is retrieved, it can be re-parsed with a CSV parsing function.
			
			Example:
			
				Baguinon,Elizalde,Gonzales,45,"40.730610,-73.935242"
	
			A schema could be an array. Multiple entries are separated by carriage returns:
			
				ver:1.0,hdr:false,del:\t; LastName:string(50),FirstName:string(50),MiddleName:50,Age:int,Entries:Entry; Entry:Door:string(10),Time:timestamp; Entries:[]Entry
	
			Example:
			
				Baguinon,Elizalde,Gonzales,45, "Front,2020-04-08T14:00.00.000Z
				Back,2020-04-08T14:01.00.000Z
				Balcony,2020-04-08T14:05.00.000Z
				Back,2020-04-08T15:00.00.000Z"
				Baguinon,Elizalde,Gonzales,45, "Front,2020-05-08T04:00.00.000Z
				Back,2020-05-08T04:10.00.000Z
				Balcony,2020-05-08T14:20.00.000Z
				Back,2020-05-08T14:50.00.000Z"
				
Comparison:

	CSV: 258 bytes
	
		Pike,Robert,C,63,8.7,60.6,true,1956-10-08,2020-04-08T14:00:00Z
		Griesemer,Robert,C,25,8.7,8.9,false,1995-09-01,2020-04-08T14:00:00Z
		Smith,John,Porter,65,6.7,6.8,true,1955-08-08,2020-04-08T14:00:00Z
		Chi,Kwan,Tai,35,7.7,20.9,true,1985-11-08,2020-04-08T14:00:00Z
		
	JSON: (prettified: 965 bytes, minified: 680 bytes ) 
		[
			{ "lastname": "Pike", "firstname": "Robert", "middlename": "F", "age": 63, "height": 8.7, "weight": 60.6, "alive": true, "dateborn": "1956-10-08", "lastupdated": "2020-04-08T14:00:00Z" },
			{ "lastname": "Griesemer", "firstname": "Robert", "middlename": "U", "age": 25, "height": 8.7, "weight": 8.9, "alive": false, "dateborn": "1995-09-01", "lastupdated": "2020-04-08T14:00:00Z" },
			{ "lastname": "Smith", "firstname": "John", "middlename": "Porter", "age": 65,"height": 6.7, "weight": 6.8,"alive": true, "dateborn": "1955-08-08", "lastupdated": "2020-04-08T14:00:00Z" },
			{ "lastname": "Chi", "firstname": "Kwan", "middlename": "Tai", "age": 35, "height":7.7, "weight":20.9, "alive": true, "dateborn": "1985-11-08", "lastupdated": "2020-04-08T14:00:00Z"}
		]
Notes:
	- Just like JSON APIs, the transport should be secured (SSL) becase the data is human readable.