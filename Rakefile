require 'rubygems'
require 'bundler/setup'

require 'csv'
require 'json'
require 'set'
require 'uri'

require 'axlsx'
require 'nokogiri'
require 'open-uri/cached'
require 'spreadsheet'

def rows
  parse('https://docs.google.com/spreadsheets/d/1AmLQD2KwSpz3B4eStLUPmUQJmOOjRLI3ZUZSD5xUTWM/pub?gid=0&single=true&output=csv', headers: true)
end

def parse(url, options={})
  data = open(url, open_timeout: 1, read_timeout: 1).read
  if encoding = options.delete(:encoding)
    data = data.force_encoding(encoding).encode('utf-8')
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

def delete_if_exists(path)
  if File.exist?(path)
    File.unlink(path)
  end
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
    $stderr.puts "#{error.io.status.first} #{url}"
  rescue Errno::ETIMEDOUT
    $stderr.puts "OUT #{url}"
  end
end

def map(output, input, prefix, size, options = {})
  input_path = "#{input}.shp"
  output_path = options[:output_path]
  options[:format] ||= 'GeoJSON'

  if options[:format] == 'GeoJSON'
    output_path ||= "maps/#{output}.geojson"
  elsif output_path.nil?
    raise 'Expected :output_path to be set'
  end

  if File.exist?(input_path)
    unless options[:append]
      delete_if_exists(output_path)
    end

    codes = rows.select do |row|
      row['Catalog URL'] && row['Code'] && row['Code'].size == size
    end.map do |row|
      "'#{row['Code']}'"
    end

    sql = if options[:centroids]
      %(-dialect sqlite -sql "SELECT ST_Centroid(geometry), #{prefix}NAME AS Name, #{prefix}UID AS ID FROM #{input} WHERE #{prefix}UID IN (#{codes.join(',')})")
    else
      %(-dialect sqlite -sql "SELECT geometry, #{prefix}NAME AS Name, #{prefix}UID AS ID from #{input} WHERE #{prefix}UID IN (#{codes.join(',')})")
    end

    echo("SHAPE_ENCODING=ISO-8859-1 ogr2ogr #{output_path} #{input_path}#{' -append' if options[:append]}#{' -lco ENCODING=UTF-8' unless options[:append]} -f '#{options[:format]}' -t_srs EPSG:4326 -select Name,ID #{sql}")

    if options[:format] == 'GeoJSON'
      echo("topojson -o maps/#{output}.topojson #{output_path}")
    end
  else
    puts "You must have a copy of the shapefile before running this task:"
    puts "curl -O http://www12.statcan.gc.ca/census-recensement/2011/geo/bound-limit/files-fichiers/#{input}.zip"
    puts "unzip #{input}.zip"
    puts "rm -f #{input}.zip"
  end
end

def enrich(base)
  map = {}

  rows.each do |row|
    map[row['Code']] = row
  end

  %w(areas markers).each do |suffix|
    path = "maps/#{base}-#{suffix}.geojson"

    if File.exist?(path)
      data = JSON.load(File.read(path))

      data['features'].each do |feature|
        feature['properties']['Name'].sub!(%r{ *[(/].+\z}, '')
        feature['properties']['Links'] = %(<a href="#{map[feature['properties']['ID']]['Catalog URL']}">Data catalog</a>)
      end

      File.open(path, 'w') do |f|
        f.write(JSON.dump(data))
      end
    end
  end
end

task :map_provinces_and_territories do
  map('provinces-and-territories-areas', 'gpr_000a11a_e', 'PR', 2)
  map('provinces-and-territories-markers', 'gpr_000a11a_e', 'PR', 2, centroids: true)
  enrich('provinces-and-territories')
end

task :map_census_divisions do
  map('census-divisions-areas', 'gcd_000a11a_e', 'CD', 4)
  map('census-divisions-markers', 'gcd_000a11a_e', 'CD', 4, centroids: true)
  enrich('census-divisions')
end

task :map_census_subdivisions do
  map('census-subdivisions-areas', 'gcsd000a11a_e', 'CSD', 7)
  map('census-subdivisions-markers', 'gcsd000a11a_e', 'CSD', 7, centroids: true)
  enrich('census-subdivisions')
end

task :map do
  map('canada', 'gpr_000a11a_e', 'PR', 2, format: 'ESRI Shapefile', output_path: 'canada.shp', centroids: true)
  map('canada', 'gcd_000a11a_e', 'CD', 4, format: 'ESRI Shapefile', output_path: 'canada.shp', centroids: true, append: true)
  map('canada', 'gcsd000a11a_e', 'CSD', 7, format: 'ESRI Shapefile', output_path: 'canada.shp', centroids: true, append: true)

  output_path = 'maps/canada-markers.geojson'
  delete_if_exists(output_path)
  echo("SHAPE_ENCODING=ISO-8859-1 ogr2ogr #{output_path} canada.shp -lco ENCODING=UTF-8 -f GeoJSON -t_srs EPSG:4326")
  echo("topojson -o maps/canada-markers.topojson #{output_path}")
  enrich('canada')
end

task :spreadsheet do
  map = {}

  rows.each do |row|
    map[row['Code']] = row
  end

  census_division_type_names = {}
  Nokogiri::HTML(open('http://www12.statcan.gc.ca/census-recensement/2011/ref/dict/table-tableau/table-tableau-4-eng.cfm').read).xpath('//table/tbody/tr/th[1]/abbr').each do |abbr|
    census_division_type_names[abbr.text] = abbr['title'].sub(%r{ */.+\z}, '')
  end

  # Map census subdivision type codes to names.
  census_subdivision_type_names = {}
  Nokogiri::HTML(open('http://www12.statcan.gc.ca/census-recensement/2011/ref/dict/table-tableau/table-tableau-5-eng.cfm').read).xpath('//table/tbody/tr/th[1]/abbr').each do |abbr|
    census_subdivision_type_names[abbr.text] = abbr['title'].sub(%r{ */.+\z}, '')
  end

  data = [['Geographic name', 'Geographic type', 'Geographic code', 'Population, 2011', 'Catalog URL', 'License URL', 'Policy URL', 'Contact name', 'Contact email', 'Generic contact', 'Twitter', 'Software']]

  { # Provinces and territories
    'http://www12.statcan.gc.ca/census-recensement/2011/dp-pd/hlt-fst/pd-pl/FullFile.cfm?T=101&LANG=Eng&OFT=CSV&OFN=98-310-XWE2011002-101.CSV' => 'provinces-and-territories',
    # Census divisions
    'http://www12.statcan.gc.ca/census-recensement/2011/dp-pd/hlt-fst/pd-pl/FullFile.cfm?T=701&LANG=Eng&OFT=CSV&OFN=98-310-XWE2011002-701.CSV' => 'census-divisions',
    # Census subdivisions
    'http://www12.statcan.gc.ca/census-recensement/2011/dp-pd/hlt-fst/pd-pl/FullFile.cfm?T=301&LANG=Eng&OFT=CSV&OFN=98-310-XWE2011002-301.CSV' => 'census-subdivisions',
  }.each do |url,level|
    scrub(parse(url, encoding: 'iso-8859-1'), level == 'provinces-and-territories' ? 2 : 3).each do |row|
      old_row = map[row[0]]

      if old_row && old_row['Catalog URL']
        new_row = case level
        when 'provinces-and-territories'
          [row[1], nil, row[0], row[3].to_i]
        when 'census-divisions'
          [row[1], census_division_type_names.fetch(row[2].strip), row[0], row[4].to_i]
        when 'census-subdivisions'
          [row[1], census_subdivision_type_names.fetch(row[2].strip), row[0], row[4].to_i]
        end

        new_row += old_row.values_at('Catalog URL', 'License URL', 'Policy URL', 'Contact name', 'Contact email', 'Generic contact', 'Twitter')

        if new_row[-1]
          new_row[-1] = "https://twitter.com/#{new_row[-1]}"
        end

        new_row << software(old_row['Catalog URL'])

        data << new_row
      end
    end
  end

  CSV.open('tables/catalogs.csv', 'w') do |csv|
    data.each do |row|
      csv << row
    end
  end

  workbook = Spreadsheet::Workbook.new
  blue_style = Spreadsheet::Format.new(color: :blue, underline: :single)
  worksheet = workbook.create_worksheet
  data.each_with_index do |row, i|
    styles = row.map do |cell|
      if String === cell && cell[/@|http/]
        blue_style
      end
    end

    cells = row.map do |cell|
      if String === cell && cell['@']
        Spreadsheet::Link.new("mailto:#{cell}", cell)
      elsif String === cell && cell['http']
        Spreadsheet::Link.new(cell, cell)
      else
        cell
      end
    end

    worksheet.insert_row(i, cells)

    styles.each_with_index do |style, i|
      if style
        worksheet.last_row.set_format(i, blue_style)
      end
    end
  end
  workbook.write('tables/catalogs.xls')

  Axlsx::Package.new do |package|
    package.workbook.add_worksheet do |worksheet|
      blue_style = worksheet.styles.add_style(fg_color: 'FF0000FF', u: true)

      data.each do |row|
        styles = row.map do |cell|
          if String === cell && cell[/@|http/]
            blue_style
          end
        end

        worksheet.add_row(row, style: styles)

        row.each_with_index do |cell, i|
          if String === cell
            if cell['@']
              worksheet.add_hyperlink(location: "mailto:#{cell}", ref: worksheet.rows.last.cells[i])
            elsif cell['http']
              worksheet.add_hyperlink(location: cell, ref: worksheet.rows.last.cells[i])
            end
          end
        end
      end
    end
    package.serialize('tables/catalogs.xlsx')
  end
end

task :markdown do
  abbreviations = {}

  CSV.parse(open('https://raw.githubusercontent.com/opencivicdata/ocd-division-ids/master/identifiers/country-ca/ca_provinces_and_territories.csv').read, headers: true) do |row|
    abbreviations[row['abbreviation']] = row['name']
  end

  software_names = {
    'http://opendata.arcgis.com/' => 'ArcGIS Open Data',
    'http://www.nucivic.com/dkan/' => 'DKAN',
    'https://ckan.org/' => 'CKAN',
    'https://github.com/openlab/OGDI-DataLab' => 'OGDI DataLab',
    'https://socrata.com/' => 'Socrata',
    'https://www.dobt.co/' => 'Opener',
    'https://www.voyagersearch.com/' => 'Voyager Search',
  }

  File.open('tables/catalogs.md', 'w') do |f|
    f.write("# Database of Canadian open government data catalogs\n\n## Table of contents\n\n")

    groups = CSV.read('tables/catalogs.csv', headers: true).group_by do |row|
      if row['Geographic code'].size == 2
        row['Geographic name']
      else
        abbreviations.fetch(row['Geographic name'].match(/\((.*)\)/)[1])
      end
    end

    groups.each do |group, _|
      f.write("* [#{group}](##{group.downcase.gsub(' ', '-')})\n")
    end

    groups.each do |group, items|
      f.write("\n## #{group}\n\n")

      items.sort! do |a,b|
        # Sort by level and then by population.
        a_size = a['Geographic code'].size
        b_size = b['Geographic code'].size

        if a_size == b_size
          b['Population, 2011'].to_i <=> a['Population, 2011'].to_i
        else
          a_size <=> b_size
        end
      end

      items.each do |row|
        line = "* [#{row[0].sub(%r{ *[(/].+\z}, '')}](#{row['Catalog URL']})\n"

        if row['License URL']
          line += "  * [License](#{row['License URL']})\n"
        end
        if row['Policy URL']
          line += "  * [Policy](#{row['Policy URL']})\n"
        end
        if row['Generic contact']
          line += "  * [Contact](#{row['Generic contact']['@'] ? "mailto:#{row['Generic contact']}" : row['Generic contact']})\n"
        end
        if row['Twitter']
          line += "  * [@#{row['Twitter'].sub('https://twitter.com/', '')}](#{row['Twitter']})\n"
        end
        if row['Contact email']
          line += "  * People: "
          contact_names = row['Contact name'].to_s.split("\n")
          line += row['Contact email'].split("\n").map.with_index do |email,i|
            "[#{contact_names[i] || email}](mailto:#{email})"
          end.join(", ")
          line += "\n"
        end

        f.write(line)
      end
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
