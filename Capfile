# Capistrano 2.x recipe file
#
# Requirement cap 2.2
#
# Execute with:
# cap -S stage=<production|test> task

require 'tempfile'

set :application, "openaustralia"

# default_run_options[:pty] = true

set :use_sudo, false

set :scm, :git
set :repository, "git://github.com/mlandauer/openaustralia.git"
set :git_enable_submodules, true
set :deploy_via, :remote_cache

set :stage, "test" unless exists? :stage

set :user, "matthewl"

ssh_options[:port] = 2506
role :web, "www.openaustralia.org"

if stage == "production"
  set :deploy_to, "/www/www/#{application}"
elsif stage == "test"
  set :deploy_to, "/www/test/#{application}"
  set :branch, "test"
end

load 'deploy' if respond_to?(:namespace) # cap2 differentiator

# Using Chef (http://wiki.opscode.com/display/chef/Home) configure the server to
# have all the software we need. WORK IN PROGRESS
task :chef do
  run "rm -rf /tmp/chef"
  upload("chef", "/tmp/chef")
  # Using "sudo -E" to ensure that environment variables are propogated to new environment
  # so that pkg_add knows to use passive ftp. What a PITA.
  sudo "-E chef-solo -l debug -c /tmp/chef/config/solo.rb -j /tmp/chef/config/dna.json"
  #sudo "-E chef-solo -c /tmp/chef/config/solo.rb -j /tmp/chef/config/dna.json"
end

namespace :deploy do
	# Restart Apache because for some reason a deploy can cause trouble very occasionally (which is fixed by a restart). So, playing safe
	task :restart do
	  sudo "apachectl restart"
	end

	task :finalize_update do
		run "chmod -R g+w #{latest_release}" if fetch(:group_writable, true)
	end

	desc "After a code update, we link the images directories to the shared ones"
	task :after_update_code do
		links = {
			"#{release_path}/searchdb"                                => "../../shared/searchdb",

			"#{release_path}/openaustralia-parser/configuration.yml"  => "../../../shared/parser_configuration.yml",

			"#{release_path}/twfy/conf/general"                       => "../../../../shared/general",
			"#{release_path}/twfy/scripts/alerts-lastsent"            => "../../../../shared/alerts-lastsent",

			"#{release_path}/twfy/www/docs/.htaccess"                 => "../../../../../shared/root_htaccess",
			"#{release_path}/twfy/www/docs/sitemap.xml"               => "../../../../../shared/sitemap.xml",
			"#{release_path}/twfy/www/docs/sitemaps"                  => "../../../../../shared/sitemaps",

		  "#{release_path}/twfy/www/docs/images/mps"                => "../../../../../../shared/images/mps",
			"#{release_path}/twfy/www/docs/images/mpsL"               => "../../../../../../shared/images/mpsL",
			"#{release_path}/twfy/www/docs/regmem/scan"               => "../../../../../../shared/regmem_scan",
			"#{release_path}/twfy/www/docs/rss/mp"                    => "../../../../../../shared/rss/mp",
			"#{release_path}/twfy/www/docs/debates/debates.rss"       => "../../../../../../shared/rss/debates.rss"
		}
		# First copy any images that have been checked into the repository to the shared area
		run "cp #{release_path}/twfy/www/docs/images/mps/* #{shared_path}/images/mps"
		run "cp #{release_path}/twfy/www/docs/images/mpsL/* #{shared_path}/images/mpsL"
		# HACK: Remove directories next because they have files in them
		run "rm -rf #{release_path}/twfy/www/docs/images/mps #{release_path}/twfy/www/docs/images/mpsL #{release_path}/twfy/www/docs/rss/mp"
		# "ln -sf <a> <b>" creates a symbolic link but deletes <b> if it already exists
		run links.map {|a| "ln -sf #{a.last} #{a.first}"}.join(";")
		# Now compile twfy/scripts/run-with-lockfile.c
		run "gcc -o #{release_path}/twfy/scripts/run-with-lockfile #{release_path}/twfy/scripts/run-with-lockfile.c"
	end
	
	desc "Upload member images from local machine"
	task :images do
		put_directory "../pwdata/images/mps", "#{shared_path}/images/mps"
		put_directory "../pwdata/images/mpsL", "#{shared_path}/images/mpsL"
	end	
end
