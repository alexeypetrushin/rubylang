# Simple Cluster Management Tool

It may be **usefull if Your claster has about 1-10 boxes**, and tools like Chef, Puppet, Capistrano
are too complex and proprietary for your needs.
**It's easy**, there are only a 2 concept - Box and Service.

Usage:

- package installation, dependencies, versioning
- process management, start/stop services
- deplyment

It's designed to be used with [Virtual File System][vfs], [Virtual Operation System][vos] and Rake, but
it's not required, You can use other tools also.

## Core Concepts

- Box - a PC with remote access.
- Service - some abstract thing (a package, running process, app, ...) that operates on 1..n
boxes. It can perform install/update/start/stop/.. operations and request other services to perform
things it needs.
- Deployment Scheme - defines how services are distributed among boxes.

## Deployment Scheme

Let's suppose that we want to deploy our App on a cluster of 3 boxes using the following scheme.

![](simple-cluster-management-tool/deployment-scheme.png)

**Tags** are used to define connections (N to M, althouth in example below it's 1 to 1) between
**Boxes** and **Services**.

Deployment scheme defined via box tags (in config) and service tags, see below.

## Boxes

Boxes are defined in config:

``` YAML
handy_scheme:
  'web1.app.com': ['web']
  'web2.app.com': ['web']
  'db.app.com': ['db']
```

## Services

Dependencies are defined by calling another services, it's easy to use and understand and allows high
flexibility in configuration.

And, **it's 'smart'**, in sample below the App::deploy method is smart ennought to figure out that it
needs the Ruby and MySQL Services and it will call for them to apply before itself.

You can specify that the package should be applied only once (see :apply_once), and use versioning
(see :version) - change the version and it will be reapplied.

It supports iterative development and can figure out what services are missing and needs to be applied,
You don't have to write all the config at once, do it by small steps, adding one package after another.

Below are our Services:

``` Ruby
class Services::Ruby < Service
  tag :web
  version 4

  def install
    # it will be called only once, it will be called next time only
    # if You change the version
    apply_once :install do |box|
      logger.info "installing :#{service_name}"
      box.fake_bash "apt-get install ruby"
    end
  end
end

class Services::Thin < Service
  tag :web

  def restart
    logger.info "restarting :#{service_name}"
    boxes.each{|box| box.fake_bash 'thin restart'}
  end
end

class Services::MySql < Service
  tag :db
  def started
    logger.info "ensuring mysql is running"
    box.fake_bash 'mysql start' unless box.fake_bash('ps -A') =~ /mysql/
  end
end

class Services::App < Service
  tag :web

  def install
    services.ruby.install
    apply_once :install do |box|
      logger.info "installing :#{service_name}"
      box.fake_bash "cd #{config.app_path} && git clone app"
    end
  end

  def update
    install
    logger.info "updating :#{service_name}"
    boxes.each{|box| box.fake_bash "cd #{config.app_path} && git pull app"}
  end

  def deploy
    update
    logger.info "deploying :#{service_name}"
    services.my_sql.started
    services.thin.restart
  end
end
```

## Rake

``` Ruby
desc 'deploy to cluster'
task :deploy do
  cluster.services.app.deploy
end
```

Note: You don't have install services before deployment, the **App::deploy will resolve all dependencies
automatically and fully configure all boxes from clean state** - it will install packages, ensure all
needed services are running and only then will start deployment.

Now, type:

``` Bash
$ rake deploy
```

You'll se something like this:

``` Bash
installing :ruby
   => bash: 'apt-get install ruby'
installing :app
   => bash: 'cd /tmp/cm_example_app && git clone app'
updating :app
   => bash: 'cd /tmp/cm_example_app && git pull app'
deploying :app
ensuring mysql is running
   => bash: 'ps -A'
   => bash: 'mysql start'
restarting :thin
   => bash: 'thin restart'
```

Deploy one more time, notice now there's no installation of Ruby and App:

``` Bash
updating :app
   => bash: 'cd /tmp/cm_example_app && git pull app'
deploying :app
ensuring mysql is running
   => bash: 'ps -A'
   => bash: 'mysql start'
restarting :thin
   => bash: 'thin restart'
```

## Installation

``` Bash
$ gem install cluster_management
```

## Examples

Clone [cluster_management][cluster_management] and go to [example][example] folder, there are full
example, type 'rake deploy' and look at the output.

For simplicity it uses the 'localhost' instead of 3 remote boxes and 'fake_bash' that just prints command
to console (because we don't want to actually alter our localhost).
But You can easily define actual remote PCs in config and replace 'fake_bash' with 'bash' and see it in
the real action.

You can also see 'real' configuration I use to manage this site, [my_cluster][my_cluster].

Tags : Administration, SSH
Date : 2011/5/1

[my_cluster]: http://github.com/alexeypetrushin/my_cluster/tree/master/lib/packages
[vos]: http://github.com/alexeypetrushin/vos
[vfs]: http://github.com/alexeypetrushin/vfs
[example]: https://github.com/alexeypetrushin/cluster_management/tree/master/example
[cluster_management]: https://github.com/alexeypetrushin/cluster_management