require 'rubygems'
require 'bundler/setup'

require 'csv'
require 'set'
require 'uri'

require 'nokogiri'
require 'open-uri/cached'

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
      else
        'unknown'
      end
    end
  rescue OpenURI::HTTPError => error
    error.io.status.first
  rescue Errno::ETIMEDOUT
    'timeout'
  end
end

def rows
  CSV.parse(open('https://docs.google.com/spreadsheets/d/1AmLQD2KwSpz3B4eStLUPmUQJmOOjRLI3ZUZSD5xUTWM/pub?gid=0&single=true&output=csv').read, headers: true)
end

task :geojson do
  if File.exist?('gcsd000a11a_e.shp')
    File.unlink('maps/census-subdivisions.geojson')
    codes = rows.select do |row|
      row['Catalog URL'] && row['Code'].size == 7
    end.map do |row|
      "'#{row['Code']}'"
    end
    `ogr2ogr maps/census-subdivisions.geojson gcsd000a11a_e.shp -f GeoJSON -t_srs EPSG:4326 -where "CSDUID IN (#{codes.join(',')})"`
  else
    puts "You must have a copy of the census subdivisions to generate the GeoJSON:"
    puts 'curl -O http://www12.statcan.gc.ca/census-recensement/2011/geo/bound-limit/files-fichiers/gcsd000a11a_e.zip'
    puts 'unzip gcsd000a11a_e.zip'
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
