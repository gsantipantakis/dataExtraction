# dataExtraction
This data extractor tool has been implemented and tailored for use with the set of triples available at https://zenodo.org/records/10006470 . It uses a SPARQL query specified by the user to filter and extract data for a given time window. It is applied directly on the archival files to minimize the requirements on resources. The generated output is stored in Tab Separated Value (TSV) format. Compiled and tested using JDK 17.0.2 2022-01-18 LTS.

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
	java -jar extractData.jar -dateFrom "2017-5-12T00:00:00" -dateTo "2017-5-12T12:00:00" -files ./zenodoFiles/ -filter selectByDate.q -output sampleData -process all
The arguments in the above example will process both synopses and raw data stored in ./zenodoFiles/ within the temporal window [2017-5-12T00:00:00, 2017-5-12T12:00:00], that satisfy the SPARQL query stored in the file selectByDate.q. The output will be stored in files sampleDataRaw.tsv and sampleDataSynopses.tsv


**Acknowledgement**
The research leading to the results presented in this repository has received funding from the European Union’s funded Project MobiSpaces under grant agreement no 101070279
