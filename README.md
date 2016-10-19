# Open Data Canada

## Resources

* [A map of all Canadian governments with open data](/maps/canada.topojson)
* [A map of provinces and territories with open data](/maps/provinces-and-territories.geojson)
* [A map of census divisions (regions) with open data](/maps/census-divisions.geojson)
* [A map of census subdivisions (municipalities) with open data](/maps/census-subdivisions.geojson)

## Development

Install:

    bundle

Find any new domain fragments among external sources:

    bundle exec rake missing

Create a map of the provinces and territories with open data:

    bundle exec rake map_provinces_and_territories

Create a map of the census divisions with open data:

    bundle exec rake map_census_divisions

Create a map of the census subdivisions with open data:

    bundle exec rake map_census_subdivisions

Copyright (c) 2016 James McKinney, released under the MIT license
