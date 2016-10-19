# Open Data Canada

## Tables

* [Database of Canadian open government data catalogs](/tables/catalogs.csv)

## Maps

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

Copyright (c) 2016 James McKinney, released under the MIT license
