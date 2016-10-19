# Database of Canadian open government data catalogs

## Downloads

Download the data table as:

* [CSV](https://raw.githubusercontent.com/jpmckinney/open_data_canada/master/tables/catalogs.csv) ([view online](/tables/catalogs.csv))
* [XLSX](https://raw.githubusercontent.com/jpmckinney/open_data_canada/master/tables/catalogs.xlsx) (Excel 2007 and later)
* [XLS](https://raw.githubusercontent.com/jpmckinney/open_data_canada/master/tables/catalogs.xls) (older Excel versions)
* [Read online](/tables/catalogs.md)

View the maps of:

* All Canadian governments with open data: [markers](/maps/canada-markers.geojson)
* Provinces and territories with open data: [markers](/maps/provinces-and-territories-markers.geojson), [areas](/maps/provinces-and-territories-areas.geojson)
* Census divisions (regions) with open data: [markers](/maps/census-divisions-markers.geojson), [areas](/maps/census-divisions-areas.geojson)
* Census subdivisions (municipalities) with open data: [markers](/maps/census-subdivisions-markers.geojson), [areas](/maps/census-subdivisions-areas.geojson)

## Development

Install:

    bundle

Find any new domain fragments among external sources:

    bundle exec rake missing

Create the spreadsheet:

    bundle exec spreadsheet

Convert the spreadsheet into Markdown:

    bundle exec markdown

Create a map of the provinces and territories with open data:

    bundle exec rake map_provinces_and_territories

Create a map of the census divisions with open data:

    bundle exec rake map_census_divisions

Create a map of the census subdivisions with open data:

    bundle exec rake map_census_subdivisions

Create one map of all the above:

    bundle exec rake map

## Why?

Ideally, this repository will be a point of collaboration between [Geothink](http://geothink.ca/), [MISA/ASIM Open Data Special Interest Group (SIG)](http://c.ymcdn.com/sites/www.misa-asim.ca/resource/resmgr/misa_pdfs/open_data_sig_-_terms_of_ref.pdf), [RPCO Regional Information Systems Working Group (RISWG)](http://www.rpco.ca/regional-information-systems-working-group.html) and Public Sector Open Data (PSOD).

Copyright (c) 2016 James McKinney, released under the MIT license
