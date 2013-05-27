require 'date'
require 'digest/md5'
require 'fileutils'
require 'nokogiri'

basedir = "."
build   = "#{basedir}/build"
source  = "#{basedir}/library/ImboClientCli"
logs    = "#{build}/logs"

desc "Task used by Jenkins-CI"
task :jenkins => [:prepare, :lint, :installdep, :test, :apidocs, :loc_ci, :cs_ci, :cpd_ci, :pmd_ci]

desc "Task used by Travis-CI"
task :travis => [:installdep, :test]

desc "Default task"
task :default => [:prepare, :lint, :installdep, :test, :apidocs, :loc, :cs, :cpd, :pmd]

desc "Clean up and create artifact directories"
task :prepare do
  FileUtils.rm_rf build
  FileUtils.mkdir build

  ["coverage", "logs", "docs"].each do |d|
    FileUtils.mkdir "#{build}/#{d}"
  end
end

desc "Check syntax on all php files in the project"
task :lint do
  `git ls-files "*.php"`.split("\n").each do |f|
    begin
      sh %{php -l #{f}}
    rescue Exception
      exit 1
    end
  end
end

desc "Install dependencies"
task :installdep do
  if ENV["TRAVIS"] == "true"
    system "composer self-update"
    system "composer --no-ansi install --dev"
  else
    Rake::Task["install_composer"].invoke
    system "php -d \"apc.enable_cli=0\" composer.phar install --dev"
  end
end

desc "Update dependencies"
task :updatedep do
  Rake::Task["install_composer"].invoke
  system "php -d \"apc.enable_cli=0\" composer.phar update --dev"
end

desc "Install/update composer itself"
task :install_composer do
  if File.exists?("composer.phar")
    system "php -d \"apc.enable_cli=0\" composer.phar self-update"
  else
    system "curl -s http://getcomposer.org/installer | php -d \"apc.enable_cli=0\""
  end
end

desc "Run tests"
task :test do
  if ENV["TRAVIS"] == "true"
    begin
      sh %{vendor/bin/phpunit --verbose -c tests/phpunit.xml.travis}
    rescue Exception
      exit 1
    end
  else
    begin
      sh %{vendor/bin/phpunit --verbose -c tests --coverage-html build/coverage --coverage-clover build/logs/clover.xml --log-junit build/logs/junit.xml}
    rescue Exception
      exit 1
    end
  end
end

desc "Generate API documentation"
task :apidocs do
  system "phpdoc -d #{source} -t #{build}/docs"
end

desc "Generate \"lines of code\" logs"
task :loc do
  system "phploc --log-csv #{logs}/phploc.csv --log-xml #{logs}/phploc.xml #{source}"
end

desc "Generate \"lines of code\""
task :loc_ci do
  system "phploc #{source}"
end

desc "Check coding standard"
task :cs do
  system "phpcs --standard=Imbo #{source}"
end

desc "Check coding standard and generate checkstyle logs"
task :cs_ci do
  system "phpcs --report=checkstyle --report-file=#{logs}/checkstyle.xml --standard=Imbo #{source}"
end

desc "Run copy&paste detector"
task :cpd do
    system "phpcpd #{source}"
end

desc "Run copy&paste detector and generate logs"
task :cpd_ci do
    system "phpcpd --log-pmd #{logs}/pmd-cpd.xml #{source}"
end

desc "Run project mess detector"
task :pmd do
    system "phpmd #{source} text #{basedir}/phpmd.xml"
end

desc "Run project mess detector and generate XML and HTML logs"
task :pmd_ci do
    system "phpmd #{source} xml #{basedir}/phpmd.xml --reportfile #{logs}/pmd.xml"
    system "phpmd #{source} html #{basedir}/phpmd.xml --reportfile #{logs}/pmd.html"
end
