# Open Data Canada

## Resources

* [A map of census subdivisions (municipalities) with open data](/maps/census-subdivisions.geojson)
* [A map of census divisions (upper-tier municipalities) with open data](/maps/census-subdivisions.geojson)

## Development

Install:

    bundle

Find any new domain fragments among external sources:

    bundle exec rake missing

Create a map of the census subdivisions with open data:

    bundle exec rake map_census_subdivisions

Create a map of the census divisions with open data:

    bundle exec rake map_census_divisions

Copyright (c) 2016 James McKinney, released under the MIT license
