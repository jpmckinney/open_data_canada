# Database of Canadian open government data catalogs

## Downloads

Download the data table as:

* [CSV](https://raw.githubusercontent.com/jpmckinney/open_data_canada/master/tables/catalogs.csv) ([preview](/tables/catalogs.csv))
* [Excel (2007 and later) (XLSX)](https://raw.githubusercontent.com/jpmckinney/open_data_canada/master/tables/catalogs.xlsx)
* [Excel (older versions) (XLS)](https://raw.githubusercontent.com/jpmckinney/open_data_canada/master/tables/catalogs.xls)

See also:

* [A map of all Canadian governments with open data](/maps/canada.topojson)
* [A map of provinces and territories with open data](/maps/provinces-and-territories.geojson)
* [A map of census divisions (regions) with open data](/maps/census-divisions.geojson)
* [A map of census subdivisions (municipalities) with open data](/maps/census-subdivisions.geojson)

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
