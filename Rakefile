require 'rubygems'
require 'bundler/setup'

require 'csv'
require 'set'
require 'uri'

require 'nokogiri'
require 'open-uri/cached'

def parse(url, options={})
  data = open(url).read
  if encoding = options.delete(:encoding)
    data.force_encoding(encoding).encode('utf-8')
  end
  CSV.parse(data, options)
end

def scrub(rows, skip=0)
  skip.times do
    rows.shift
  end
  rows.take_while do |row|
    !row.empty?
  end
end

def rows
  parse('https://docs.google.com/spreadsheets/d/1AmLQD2KwSpz3B4eStLUPmUQJmOOjRLI3ZUZSD5xUTWM/pub?gid=0&single=true&output=csv', headers: true)
end

def echo(command)
  puts command
  system(command)
end

def get_host(url)
  begin
    URI.parse(url).host
  rescue URI::InvalidURIError
    puts "invalid #{url}"
  end
end

def get_base(host)
  host.sub(/\.
    (?:
      (?:(?:gov\.)?(?:ab|bc|on|nl|qc|public\.esolutionsgroup)\.)?ca|
      (?:(?:(?:rdco\.)?opendata\.arcgis|wpengine)\.)?com|
      cloudapp\.net|
      org
    )\z/x, '').sub(/(?:opendata-|.+\.)/, '')
end

def software(url)
  begin
    data = open(url).read
    if data['https://dobt-opener.s3.amazonaws.com/uploads/']
      'https://www.dobt.co/'
    elsif data['/esri/css/esri.css']
      'http://opendata.arcgis.com/'
    elsif data['/Scripts/ogdi/list.js']
      'https://github.com/openlab/OGDI-DataLab'
    elsif data['/profiles/dkan/themes/contrib/']
      'http://www.nucivic.com/dkan/'
    elsif data["You don't have permission to use Voyager."]
      'https://www.voyagersearch.com/'
    elsif data[/ckan/i]
      'https://ckan.org/'
    elsif data[/socrata/i]
      'https://socrata.com/'
    else
      number_of_files = %w(pb shp zip).reduce(0) do |memo,extension|
        memo += Nokogiri::HTML(data).xpath("//@href['.#{extension}'=substring(., string-length(.) - #{extension.size})]").size
      end
      if number_of_files >= 2 # Brandon 2 Thunder Bay 4
        'custom'
      end
    end
  rescue OpenURI::HTTPError => error
    error.io.status.first
  rescue Errno::ETIMEDOUT
    'timeout'
  end
end

def map(output, input, column=nil, size=nil, format='GeoJSON', output_path=nil, append=false)
  input_path = "#{input}.shp"
  if format == 'GeoJSON'
    output_path = "maps/#{output}.geojson"
  end

  if File.exist?(input_path)
    if File.exist?(output_path) && !append
      File.unlink(output_path)
    end

    codes = rows.select do |row|
      row['Catalog URL'] && row['Code'] && row['Code'].size == size
    end.map do |row|
      "'#{row['Code']}'"
    end

    echo("ogr2ogr #{output_path} #{input_path}#{' -append' if append} -f '#{format}' -t_srs EPSG:4326#{%( -where "#{column} IN (#{codes.join(',')})") if column && size}")
    if format == 'GeoJSON'
      echo("topojson -o maps/#{output}.topojson #{output_path}")
    end
  else
    puts "You must have a copy of the shapefile before running this task:"
    puts "curl -O http://www12.statcan.gc.ca/census-recensement/2011/geo/bound-limit/files-fichiers/#{input}.zip"
    puts "unzip #{input}.zip"
    puts "rm -f #{input}.zip"
  end
end

task :map_provinces_and_territories do
  map('provinces-and-territories', 'gpr_000a11a_e', 'PRUID', 2)
end

task :map_census_divisions do
  map('census-divisions', 'gcd_000a11a_e', 'CDUID', 4)
end

task :map_census_subdivisions do
  map('census-subdivisions', 'gcsd000a11a_e', 'CSDUID', 7)
end

task :map do
  map('canada', 'gpr_000a11a_e', 'PRUID', 2, 'ESRI Shapefile', 'canada.shp')
  map('canada', 'gcd_000a11a_e', 'CDUID', 4, 'ESRI Shapefile', 'canada.shp', true)
  map('canada', 'gcsd000a11a_e', 'CSDUID', 7, 'ESRI Shapefile', 'canada.shp', true)
  map('canada', 'canada')
end

task :spreadsheet do
  map = {}
  rows.each do |row|
    map[row['Code']] = row
  end

  spreadsheet = [['Geographic name', 'Geographic code', 'Population, 2011', 'Catalog URL', 'License URL', 'Policy URL', 'Contact name', 'Contact email', 'Generic contact', 'Twitter', 'Software']]

  {
    # Provinces and territories
    'http://www12.statcan.gc.ca/census-recensement/2011/dp-pd/hlt-fst/pd-pl/FullFile.cfm?T=101&LANG=Eng&OFT=CSV&OFN=98-310-XWE2011002-101.CSV' => true,
    # Census divisions
    'http://www12.statcan.gc.ca/census-recensement/2011/dp-pd/hlt-fst/pd-pl/FullFile.cfm?T=701&LANG=Eng&OFT=CSV&OFN=98-310-XWE2011002-701.CSV' => false,
    # Census subdivisions
    'http://www12.statcan.gc.ca/census-recensement/2011/dp-pd/hlt-fst/pd-pl/FullFile.cfm?T=301&LANG=Eng&OFT=CSV&OFN=98-310-XWE2011002-301.CSV' => false,
  }.each do |url,subnational|
    scrub(parse(url, encoding: 'iso-8859-1'), subnational ? 2 : 3).each do |row|
      old_row = map[row[0]]

      if old_row && old_row['Catalog URL']
        new_row = if subnational
          [row[1], row[0], row[3].to_i]
        else
          [row[1], row[0], row[4].to_i]
        end

        new_row += old_row.values_at('Catalog URL', 'License URL', 'Policy URL', 'Contact name', 'Contact email', 'Generic contact', 'Twitter')

        if new_row[-1]
          new_row[-1] = "https://twitter.com/#{new_row[-1]}"
        end

        new_row << software(old_row['Catalog URL'])

        spreadsheet << new_row
      end
    end
  end

  CSV.open('tables/catalogs.csv', 'w') do |csv|
    spreadsheet.each do |row|
      csv << row
    end
  end
end

task :missing do
  allowed_duplicates = Set.new(%w(
    calgaryregionopendata
    donneesquebec
    niagaraopendata
  ))

  excluded_hosts = Set.new(
    # Civil society
    %w(
      capitaleouverte.org
      gatineauouverte.org
      montrealouvert.net
      opendatask.ca
      openhalton.ca
      openhamilton.ca
      www.dataott.org
      www.datato.org
      www.opendatalondon.ca
      www.opendataottawa.ca
      www.opendatawr.ca
      www.quebecouvert.org
    ) +
    # Private sector
    %w(
      sites.google.com
      www.meetup.com
      www.slideshare.net
      www.socialtext.net
    ) +
    # Non-municipal
    %w(
      data.wpsgn.opendata.arcgis.com
      maps.grandriver.ca
    ) +
    # Non-Canadian
    %w(
      www.egov.vic.gov.au
    )
  )

  base_corrections = {
    'gouv' => 'donneesquebec',
    'halifax' => 'hrm',
    'kitchener' => 'kitchenergis',
    'niagararegion' => 'niagaraopendata',
    'openguelph' => 'guelph',
    'openregina' => 'regina',
    'princegeorge' => 'cityofpg',
    'qualicumbeach' => 'opendatabc',
    'regionaldistrict' => 'rdcodatadownload',
    'sherbrooke' => 'donneesquebec',
    'waterloo' => 'city-of-waterloo',
  }

  bases = Set.new
  other_bases = Set.new

  rows.each do |row|
    if row['Catalog URL']
      base = get_base(get_host(row['Catalog URL']))
      if bases.include?(base) && !allowed_duplicates.include?(base)
        raise "duplicate base #{base}"
      end
      bases << base
    end
  end

  # Get others' lists of data catalogs.
  urls = []
  # Local government
  urls += Nokogiri::XML(open('http://open.canada.ca/sites/default/files/kml_js/open-cities-en.kml').read).remove_namespaces!.xpath('//url').map{|url| Nokogiri::HTML(url.text).xpath('//@href')[0].value}
  urls += CSV.parse(open('https://docs.google.com/spreadsheets/d/1Qy8LBpm5qd6C7EEMQeNLNDS7ZZAvwqy9LcjjgpOiIFQ/pub?gid=0&single=true&output=csv').read, row_sep: "\r\n").drop(2).map{|row| row[2]}.compact
  urls += Nokogiri::HTML(open('http://datalibre.ca/links-resources/').read).xpath('//ol[4]//@href').map{|href| href.value}
  # Regional government
  urls += Nokogiri::XML(open('http://open.canada.ca/sites/default/files/kml_js/open-provinces-en.kml').read).remove_namespaces!.xpath('//url').map{|url| Nokogiri::HTML(url.text).xpath('//@href')[0].value}
  urls += Nokogiri::HTML(open('http://datalibre.ca/links-resources/').read).xpath('//ol[3]//@href').map{|href| href.value}

  urls.each do |url|
    host = get_host(url)
    if host && !excluded_hosts.include?(host)
      base = get_base(host)
      # Account for different domains for the data catalogs.
      other_bases << base_corrections.fetch(base, base)
    end
  end

  other_bases.to_a.each do |base|
    unless bases.include?(base)
      puts base
    end
  end
end