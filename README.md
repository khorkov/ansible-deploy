Deploy on production with ansible:

Notice:

Testing only on Ubuntu 16.04.

Include:

1. nginx
2. postgresql
3. puma
4. rbenv
5. redis
6. sphinx
7. exim4
8. monit

Install ansible:

Ubuntu:

apt-get install ansible

sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible

Setup:

Before run change your ip address in hosts file.

ansible-playbook python.yml server.yml -i hosts

Deploy:

In Gemfile:

	group :development do
	  gem 'capistrano', require: false
	  gem 'capistrano-bundler', require: false
	  gem 'capistrano-rails', require: false
	  gem 'capistrano-rbenv', require: false
	  gem 'capistrano3-puma', github: "seuros/capistrano-puma", require: false
	  gem 'capistrano-sidekiq', require: false
	end


Perform the installation of all new gems and the installation of capistrano:

bundle install
cap install

Change in file config/deploy.rb: 

	lock '~> 3.11.0'

	set :application, 'usagle'
	set :repo_url, 'git@github.com:usagle/usagle_app.git'

	set :branch, 'master'
	set :deploy_user, 'deployer'

	set :deploy_to, '/home/deployer/applications/usagle'

	set :linked_files, %w{config/database.yml .env.production}

	set :linked_dirs, %w{log tmp/pids tmp/cache tmp/sockets public/system public/uploads}

	set :rbenv_type, :user
	set :rbenv_ruby, '2.4.1'
	set :rbenv_prefix, "RBENV_ROOT=#{fetch(:rbenv_path)} RBENV_VERSION=#{fetch(:rbenv_ruby)} #{fetch(:rbenv_path)}/bin/rbenv exec"
	set :rbenv_roles, :all

	set :default_shell, '/bin/bash -l'

	set :puma_init_active_record, true

	namespace :deploy do
	  desc 'Restart application'
	  task :restart do
	    on roles(:app), in: :sequence, wait: 5 do
	    	invoke 'puma:restart'
	    end
	  end

	end
	
Change in file config/deploy/production.rb:

role :app, %w[deployer@your ip]
role :web, %w[deployer@your ip]
role :db,  %w[deployer@your ip]

set :rails_env, :production
set :stage, :production

server 'your ip', user: 'deployer', roles: %w{web app db}, primary: true

set :ssh_options, {
   keys: %w[/home/user_name/.ssh/id_rsa],
   forward_agent: true,
   auth_methods: %w(publickey password),
   port: 22
}

Change your ip in file config/deploy/production.rb

Also, don't forget to tweak Capfile to connect with capistrano related gems that we added a couple of minutes ago in the Gemfile.

Now that the installation is complete, the application itself can be covered.

cap production deploy

Finish!