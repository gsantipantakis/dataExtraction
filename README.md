# dataExtraction
This data extractor tool has been implemented and tailored for use with the set of triples available at https://zenodo.org/records/10006470 . It uses a SPARQL query specified by the user to filter and extract data for a given time window. It is applied directly on the archival files to minimize the requirements on resources. The generated output is stored in Tab Separated Value (TSV) format. 

Compiled and tested using JDK 17.0.2 2022-01-18 LTS.

# Usage
At the command line write the following for a brief help text:
java -jar extractData.jar

The tool can be run using the following arguments:

	-dateFrom <dateTime>: the starting time instant for the temporal filter (e.g. "2017-5-12T00:00:00")
	-dateTo <dateTime>: the ending time instant for the temporal filter (e.g. "2017-5-12T00:00:00")
	-files <path_to_folder>: the local file system path where the zenodo archives can be found
	-filter <path_to_file>: the file that contains the SPARQL query that will be used for retrieving the data from the set of triples
	-output <path_to_file>: the file where the extracted data will be stored
	-process {raw,synopses,all}: the type of archives to be processed
Example:
	<pre>java -jar extractData.jar -dateFrom "2017-5-12T00:00:00" -dateTo "2017-5-12T12:00:00" -files ./zenodoFiles/ -filter selectByDate.q -output sampleData -process all</pre>
 
The configuration in the above example instructs the tool to process both synopses and raw data (`-process all`) stored in `./zenodoFiles/` within the temporal window `[2017-5-12T00:00:00, 2017-5-12T12:00:00]`, that satisfy the SPARQL query stored in the file `selectByDate.q`. The output will be stored in the files `sampleDataRaw.tsv` and `sampleDataSynopses.tsv` (output `sampleData` appended with `Raw.tsv` and `Synopses.tsv` respectively). The file `selectByDate.q` contains the following SPARQL query:

<pre>
PREFIX s:   &lt;http://www.unipi.gr/dataExtraction/spatial#&gt;
PREFIX t:   &lt;http://www.unipi.gr/dataExtraction/temporal#&gt;
PREFIX txt: &lt;http://www.unipi.gr/dataExtraction/textual#&gt;
PREFIX ogc: &lt;http://www.opengis.net/ont/geosparql#&gt;
PREFIX xsd: &lt;http://www.w3.org/2001/XMLSchema#&gt;
PREFIX :    &lt;http://www.vesselAI-project.eu/ontology#&gt;

SELECT ?tr (xsd:string(?wktOGC) as ?wkt) (xsd:string(?dateTime) as ?dtime) ?heading ?speed 
	?temperature ?min3hTemp ?max3hTemp ?pressure ?relHumidity ?windGust ?visibility 
	?dewpoint ?precipitation ?uWind ?vWind ?mathWindDir ?meteoWindDir 
	(txt:fragment(?a) as ?annotation)
WHERE{
	?tr :hasPart ?p .
	?p :hasGeometry [ ogc:asWKT ?wktOGC] ; 
		:hasTemporalFeature ?t ; :hasHeading ?heading ; 
		:hasSpeed ?speed ; 
		:hasWeatherConditions [ :hasTemperature ?temperature; :hasMinTemperature3H 
			?min3hTemp; :hasMaxTemperature3H ?max3hTemp; :atmPressureMSL ?pressure; 
			:relativeHumidity ?relHumidity; :windGust ?windGust; 
			:horVisibility ?visibility; :dewpointTemp ?dewpoint; 
			:precipitation3H ?precipitation; :u_windComponent ?uWind; 
			:v_windComponent ?vWind; :mathematicalWindDirection ?mathWindDir; 
			:meteoWindDirection ?meteoWindDir ] .
	?t :hasTimeStart ?dateTime .
	OPTIONAL{?p :hasAnnotation ?a}.
	FILTER(t:during(?dateTime,###FROM###,###TO###))
}
</pre>

Please notice that `###FROM###,###TO###` are placeholders that will be automatically replaced by the boundaries of the temporal window specified by the user at command line. 

Finally, the data extraction tool extends the SPARQL 1.1 set of functions to simplify the data extraction process. In the above example the function `t:during/3` is not a SPARQL 1.1 function, and it is available in the extension provided by the tool. The extension currently includes the following functions:

	<http://www.unipi.gr/dataExtraction/temporal#during>: 
 		returns true if a time instant is within a specified time interval. It takes 
   		three dateTime literal arguments where the first is the time instant to be 
     		tested, the second and third arguments define the time interval. 
     		Example:
       		   during("2017-5-15T14:30:00", "2017-5-12T00:00:00", "2017-5-17T00:00:00") 
       		      returns true.
 	<http://www.unipi.gr/dataExtraction/textual#fragment>: 
  		returns the fragment identifier of a given URI. 
  		Example:
    		   fragment(<http://www.vesselAI-project.eu/ontology#STOP>) 
    		      returns "STOP".
	<http://www.unipi.gr/dataExtraction/spatial#within>: 
 		returns true if the geometry specified in the first argument is topologically within
   		the geometry specified within the geometry specified in the second argument. 
     		Geometries are provided as OGC literals or plain WKT strings. 
     		Example:
       		   within("POINT(1 1)","POLYGON((0 0, 0 2, 2 2, 2 0, 0 0))") 
       		      returns true.


**Acknowledgement**

The research leading to the results presented in this repository has received funding from the European Union’s funded Project MobiSpaces under grant agreement no 101070279
