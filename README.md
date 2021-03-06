# Database of Canadian open government data catalogs

The purpose of this project is to compile the most complete list of Canadian open government data catalogs. It aggregates all known but incomplete lists, including those of [Namara](https://namara.io/), [Canada](http://open.canada.ca/en/maps/open-data-canada), [Calgary](https://data.calgary.ca/Government/Alberta-Open-Data-Portals-Map-View/grtv-hw7b), Jury Konga and Tracey Lauriault.

This project also collects the open data licenses, open data policies, contact information and catalog software of the more than 100 open data catalogs that it tracks.

Contributions, corrections and suggestions are welcome. [Create an issue on GitHub](https://github.com/jpmckinney/open_data_canada/issues/new) or [contact James McKinney](mailto:james@slashpoundbang.com).

Care about open source? Head over to [auditing tools for Canadian governments' open source code](https://github.com/jpmckinney/open_source_canada).

## History

* **2019-01-25**: Added Add Markham, Norfolk County, Fort Erie, Scugog, Uxbridge, Niagara-on-the-Lake, Brock.
* **2018-03-04**: Added Cowichan Valley, Dufferin, Hastings. British Columbia no longer [maintains its own list](https://www2.gov.bc.ca/gov/content/data/open-data/open-data-nationally-globally).
* **2017-08-03**: Added Ajax, Quinte West.
* **2017-01-06**: Added Oshawa.
* **2016-11-05**: Added Cambridge, Durham, East Hants, Huron, Moncton, North Cowichan, Shawinigan, St. Albert, Yellowknife.

## Downloads

Download the data table as:

* [CSV](https://raw.githubusercontent.com/jpmckinney/open_data_canada/master/tables/catalogs.csv) ([view online](/tables/catalogs.csv))
* [XLSX](https://raw.githubusercontent.com/jpmckinney/open_data_canada/master/tables/catalogs.xlsx) (Excel 2007 and later)
* [XLS](https://raw.githubusercontent.com/jpmckinney/open_data_canada/master/tables/catalogs.xls) (older Excel versions)
* [Read online](/tables/catalogs.md)

View the maps of:

* All Canadian governments with open data: [as map pins](/maps/canada-markers.geojson)
* Provinces and territories with open data: [as map pins](/maps/provinces-and-territories-markers.geojson), [as areas](/maps/provinces-and-territories-areas.geojson)
* Census divisions (regions) with open data: [as map pins](/maps/census-divisions-markers.geojson), [as areas](/maps/census-divisions-areas.geojson)
* Census subdivisions (municipalities) with open data: [as map pins](/maps/census-subdivisions-markers.geojson), [as areas](/maps/census-subdivisions-areas.geojson)

## Development

    bundle
    bundle exec rake --tasks

Get a [Namara](https://namara.io) API key.

The typical sequence is:

    bundle exec rake missing API_KEY=…
    bundle exec rake spreadsheet
    bundle exec rake markdown
    bundle exec rake map_provinces_and_territories map_census_divisions map_census_subdivisions map

## Why?

Ideally, this repository will be a point of collaboration between [Geothink](http://geothink.ca/), the [MISA/ASIM Open Data Special Interest Group (SIG)](http://c.ymcdn.com/sites/www.misa-asim.ca/resource/resmgr/misa_pdfs/open_data_sig_-_terms_of_ref.pdf), the [RPCO Regional Information Systems Working Group (RISWG)](http://www.rpco.ca/regional-information-systems-working-group.html), Public Sector Open Data (PSOD) and the [Government of Canada](http://open.canada.ca/en/maps/open-data-canada).

Copyright (c) 2016 James McKinney, released under the MIT license
