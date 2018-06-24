# Study Guide for DEPLOYING COOKBOOKS BADGE CERTIFICATION EXAM

> The outline is copied from [training.chef.io - Deploying Cookbooks](https://training.chef.io/static/Deploying_Cookbooks.pdf). All the information is copied&pasted from [docs.chef.io documentation portal](https://docs.chef.io).
> Have a look to Annie's awesome work: https://github.com/anniehedgpeth/chef-certification-study-guides/tree/master/deploying-cookbooks

_To obtain the Deploying Cookbooks badge you must show a proficiency around deploying Chef recipes and managing nodes._

 _You should be able to:_

- upload cookbooks to the Chef server

```
knife cookbook upload cookbook_name
```

- manage all cookbook dependencies using Berkshelf

_Add `depends 'cookbook_name', 'version_number'` to metadata.rb. Then run:_

```
berks install
```

- bootstrap a node, and describe in detail what happens during this process, be able to discuss other methods for bringing a node under the control of Chef, e.g. using `knife plugins` (e.g. `knife ec2`), and be able to discuss and use the `first-boot.json`.

[Bootstrap a node](https://docs.chef.io/install_bootstrap.html)

[About run-lists](https://docs.chef.io/run_lists.html)

[About knife cloud plugins](https://docs.chef.io/plugin_knife.html)

```
knife ec2 server create
```

_`first-boot.json` looks like this:_

```
{
  "run_list": ["recipe[foo]"],
  "container_service": {
    "chef-init-test": {
      "command": "/opt/chef/bin/chef-init-test"
    }
  }
}
```

_Here’s another example, for a minimal bootstrap:_

```
{
  "run_list": ["recipe[company_base::default]"]
}
```

`chef-client -j first-boot.json`

_On UNIX- and Linux-based machines: The second shell script executes the chef-client binary with a set of initial settings stored within `first-boot.json` on the node. `first-boot.json` is generated from the workstation as part of the initial knife bootstrap subcommand.

On Microsoft Windows machines: The batch file that is derived from the `windows-chef-client-msi.erb` bootstrap template executes the chef-client binary with a set of initial settings stored within `first-boot.json` on the node. `first-boot.json` is generated from the workstation as part of the initial knife bootstrap subcommand._

- manage and configure Roles and Environments

[About roles](https://docs.chef.io/roles.html)

[About environments](https://docs.chef.io/environments.html)

- manage and configure multiple levels of attribute precedence

[About attributes](https://docs.chef.io/attributes.html)

- familiar with the internal workings and configuration of chef-client

[Chef client overview](https://docs.chef.io/chef_client_overview.html)

[Chef-client executable](https://docs.chef.io/ctl_chef_client.html)

- familiar with the two-phases of operation

[Compile vs Execute - Common functionality](https://docs.chef.io/resource_common.html)

[Chef client overview](https://docs.chef.io/chef_client_overview.html)

- familiar with the authentication model

https://docs.chef.io/auth.html

- familiar with the use cases for Chef Solo

[About Chef solo](https://docs.chef.io/chef_solo.html)

[Chef solo executable](https://docs.chef.io/ctl_chef_solo.html)

# Badge Scope

- Anatomy of a Chef Run
- Environments
- Roles
- Data Bags
- Uploading Cookbooks to Chef Server
- Using Knife
- Bootstrapping
- Policy Files
- Search
- Chef Solo

# Learning Resources

To prepare for this assessment, we encourage you to go through the following online tutorials:

- [Infrastructure Automation](https://learn.chef.io/tracks/infrastructure-automation)
- [Continuous Automation](https://learn.chef.io/tracks/continuous-automation)
- [Writing Cookbooks](https://learn.chef.io/tracks/writing-cookbooks)
- [Administering a Chef Installation](https://learn.chef.io/tracks/administering-chef-installation)

The Deploying Cookbooks badge is awarded when someone proves that they understand
how to use Chef server to manage nodes and ensure they're in their expected state.
Candidates must show:

- An understanding of the chef-client run phases.
- An understanding of using Roles and Environments.
- An understanding of maintaining cookbooks on the Chef server.
- An understanding of using knife.
- An understanding of using how bootstrapping works.
- An understanding of what policy files are.
- An understanding of what Chef solo is.
- An understanding of using search.
- An understanding of databags.

Here is a detailed breakdown of each area.

# ANATOMY OF A CHEF RUN

## CHEF-CLIENT OPERATION
[Chef Client Overview](https://docs.chef.io/chef_client_overview.html)

- chef-client modes of operation

[chef-client (executable)]https://docs.chef.io/ctl_chef_client.html
Local mode, Audit mode, FIPS mode, as a Service

`-z`, `--local-mode`
_Run the chef-client in local mode. This allows all commands that work against the Chef server to also work against the local chef-repo._

_Local mode is a way to run the chef-client against the chef-repo on a local machine as if it were running against the Chef server. Local mode relies on chef-zero, which acts as a very lightweight instance of the Chef server. chef-zero reads and writes to the chef_repo_path, which allows all commands that normally work against the Chef server to be used against the local chef-repo._

`-W`, `--dry-run` mode
_Run the executable in why-run mode, which is a type of chef-client run that does everything except modify the system. Use why-run mode to understand why the chef-client makes the decisions that it makes and to learn more about the current and proposed state of the system._


`--audit-mode`
_The chef-client may be run in audit-mode. Use audit-mode to evaluate custom rules—also referred to as audits—that are defined in recipes_

`--fips`
_Federal Information Processing Standards (FIPS) is a United States government computer security standard that specifies security requirements for cryptography_

_The chef-client can be run as a daemon. Use the chef-client cookbook to configure the chef-client as a daemon. Add the default recipe to a node’s run-list, and then use attributes in that cookbook to configure the behavior of the chef-client._

- Configuring chef-client
[client.rb](https://docs.chef.io/config_rb_client.html)

_A client.rb file is used to specify the configuration details for the chef-client._

_On UNIX- and Linux-based machines, the default location for this file is /etc/chef/client.rb; on Microsoft Windows machines, the default location for this file is C:\chef\client.rb; use the --config option from the command line to change this location
This file is not created by default
When a client.rb file is present in the default location, the settings contained within that client.rb file will override the default configuration settings_

_A sample client.rb file that contains the most simple way to connect to_ https://manage.chef.io

```
log_level        :info
log_location     STDOUT
chef_server_url  'https://api.chef.io/organizations/<orgname>'
validation_client_name '<orgname>-validator'
validation_key '/etc/chef/validator.pem'
client_key '/etc/chef/client.pem'
```

## COMPILE VS EXECUTE
[Compile vs Execute - Common functionality](https://docs.chef.io/resource_common.html)
_The chef-client processes recipes in two phases:_

_First, each resource in the node object is identified and a resource collection is built. All recipes are loaded in a specific order, and then the actions specified within each of them are identified. This is also referred to as the “compile phase”._

_Next, the chef-client configures the system based on the order of the resources in the resource collection. Each resource is mapped to a provider, which then examines the node and performs the necessary steps to complete the action. This is also referred to as the “execution phase”._

_Typically, actions are processed during the execution phase of the chef-client run. However, sometimes it is necessary to run an action during the compile phase. For example, a resource can be configured to install a package during the compile phase to ensure that application is available to other resources during the execution phase._

- What happens during the 'compile' phase?

_The chef-client identifies each resource in the node object and builds the resource collection. Libraries are loaded first to ensure that all language extensions and Ruby classes are available to all resources. Next, attributes are loaded, followed by lightweight resources, and then all definitions (to ensure that any pseudo-resources within definitions are available). Finally, all recipes are loaded in the order specified by the expanded run-list. This is also referred to as the “compile phase”._

- What happens during the 'execute' phase?

_The chef-client configures the system based on the information that has been collected. Each resource is executed in the order identified by the run-list, and then by the order in which each resource is listed in each recipe. Each resource in the resource collection is mapped to a provider. The provider examines the node, and then does the steps necessary to complete the action. And then the next resource is processed. Each action configures a specific part of the system. This process is also referred to as convergence. This is also referred to as the “execution phase”._

- What happens when you place some Ruby at the start of a recipe?

> Ruby code outside of the ruby_block is executed during the compile phase.

- What happens when you place some Ruby at the end of a recipe?

> Ruby code outside of the ruby_block is executed during the compile phase.

- When are attributes evaluated?

> Attributes are evaluated during the compile phase

- What happens during a 'node.save' operation?

_When all of the actions identified by resources in the resource collection have been done, and when the chef-client run finished successfully, the chef-client updates the node object on the Chef server with the node object that was built during this chef-client run. (This node object will be pulled down by the chef-client during the next chef-client run.) This makes the node object (and the data in the node object) available for search._

[chef node class and node.save method](https://www.rubydoc.info/github/opscode/chef/Chef/Node)
> If you want to save a node object immediately then you should invoke node.save command in your recipe code

## RUN_STATUS
[Run status object](https://docs.chef.io/handlers.html#run-status-object)

- How can you tap into the chef-client run?

_Use a handler to identify situations that arise during a chef-client run, and then tell the chef-client how to handle these situations when they occur._

- What is ‘run_status’?

_The run_status object is initialized by the chef-client before the report interface is run for any handler. The run_status object keeps track of the status of the chef-client run and will contain some (or all) of the following properties:_

```
all_resources
backtrace
elapsed_time
end_time
exception
failed?
node
run_context
start_time
success?
updated_resources
```

- What is ‘run_state’?

 [About recipes](https://docs.chef.io/recipes.html#node-run-state)

_Use node.run_state to stash transient data during a chef-client run. This data may be passed between resources, and then evaluated during the execution phase. run_state is an empty Hash that is always discarded at the end of the chef-client run._

- What is the ‘resource collection’?

https://github.com/chef/chef/blob/master/lib/chef/resource_collection.rb
_ResourceCollection currently handles two tasks:_

_1) Keeps an ordered list of resources to use when converging the node_

_2) Keeps a unique list of resources (keyed as `type[name]`) used for notifications_

## AUTHENTICATION
- How does the chef-client authenticate with the Chef Server?

[Authentication, authorization](https://docs.chef.io/auth.html)

_The authentication process ensures the Chef server responds only to requests made by trusted users. Public key encryption is used by the Chef server. When a node and/or a workstation is configured to run the chef-client, both public and private keys are created. The public key is stored on the Chef server, while the private key is returned to the user for safe keeping. (The private key is a .pem file located in the .chef directory or in /etc/chef.)_

_Both the chef-client and knife use the Chef server API when communicating with the Chef server. The chef-validator uses the Chef server API, but only during the first chef-client run on a node._

_Each request to the Chef server from those executables sign a special group of HTTP headers with the private key. The Chef server then uses the public key to verify the headers and verify the contents._

- Authentication and using NTP

_Debugging authentication problems: The system clock has drifted from the actual time by more than 15 minutes. This can be fixed by syncing the clock with an Network Time Protocol (NTP) server._

## CHEF COMPILE PHASE
- Do all cookbooks always get downloaded?

_The chef-client asks the Chef server for a list of all cookbook files (including recipes, templates, resources, providers, attributes, libraries, and definitions) that will be required to do every action identified in the run-list for the rebuilt node object. The Chef server provides to the chef-client a list of all of those files. The chef-client compares this list to the cookbook files cached on the node (from previous chef-client runs), and then downloads a copy of every file that has changed since the previous chef-client run, along with any new files._

- What about dependencies, and their dependencies?

_When a chef-client is run, it will perform all of the steps that are required to bring the node into the expected state, including synchronizing cookbooks and compiling the resource collection by loading each of the required cookbooks, including recipes, attributes, and all other dependencies._

- What order do the following get loaded - libraries, attributes, resources/providers,
definitions, recipes?

[Chef Client Overview](https://docs.chef.io/chef_client_overview.html)

_Libraries -> Attributes -> LWRP -> Definitions -> Recipes_

_Libraries are loaded first to ensure that all language extensions and Ruby classes are available to all resources. Next, attributes are loaded, followed by lightweight resources, and then all definitions (to ensure that any pseudo-resources within definitions are available). Finally, all recipes are loaded in the order specified by the expanded run-list._

## CONVERGENCE

- What happens during the execute phase of the chef-client run?

[Compile vs Execute - Common functionality](https://docs.chef.io/resource_common.html)

_Each resource is executed in the order identified by the run-list, and then by the order in which each resource is listed in each recipe. Each resource in the resource collection is mapped to a provider. The provider examines the node, and then does the steps necessary to complete the action. And then the next resource is processed. Each action configures a specific part of the system. This process is also referred to as convergence. This is also referred to as the “execution phase”._

- The test/repair model

[Learn the Chef basics](https://learn.chef.io/modules/learn-the-basics#/)

_Chef applies changes only when they are necessary._

_Each resource in the resource collection is mapped to a provider. The provider examines the node (tests), and then does the steps necessary to complete the action (repairs)_

- When do notifications get invoked?

[Common resources](https://docs.chef.io/resource_common.html#notifications)

_A notification is a property on a resource that listens to other resources in the resource collection and then takes actions based on the notification type (notifies or subscribes)._

_A resource may notify another resource to take action when its state changes._

_By default, notifications are :delayed, that is they are queued up as they are triggered, and then executed at the very end of a chef-client run. To run an action immediately, use :immediately:_

- What happens if there are multiple start notifications for a particular resource?

[StackOverFlow qa](https://stackoverflow.com/questions/28409969/chef-regarding-multiple-calls-to-the-same-resource-and-about-services)

_In this case the notifications are added to the notification stack, not the resource stack. The code to add to the notification stack will first look for an existing notification. If it finds one, then it will NOT add another._

- When is node data sent to the Chef Server?

_When all of the actions identified by resources in the resource collection have been done, and when the chef-client run finished successfully, the chef-client updates the node object on the Chef server with the node object that was built during this chef-client run. (This node object will be pulled down by the chef-client during the next chef-client run.) This makes the node object (and the data in the node object) available for search._

> node.save can be used to send the node data immediately

## WHY-RUN

[Chef Client Overview](https://docs.chef.io/chef_client_overview.html)

- What is the purpose of ‘why-run’

_why-run mode is a way to see what the chef-client would have configured, had an actual chef-client run occurred. This approach is similar to the concept of “no-operation” (or “no-op”): decide what should be done, but then don’t actually do anything until it’s done right. This approach to configuration management can help identify where complexity exists in the system, where inter-dependencies may be located, and to verify that everything will be configured in the desired manner._

- How do you invoke a ‘why-run’

[chef-client.rb (executable)](https://docs.chef.io/ctl_chef_client.html)

`-W`, `--why-run`
_Run the executable in why-run mode, which is a type of chef-client run that does everything except modify the system. Use why-run mode to understand why the chef-client makes the decisions that it makes and to learn more about the current and proposed state of the system._

- What are limitations of doing a why run?

_why-run mode is not a replacement for running cookbooks in a test environment that mirrors the production environment. Chef uses why-run mode to learn more about what is going on, but also Kitchen on developer systems, along with an internal OpenStack cloud and external cloud providers to test more thoroughly._

_If the action is :start and the service is not running, then start the service (if it is not running) and do nothing (if it is running). What about a service that is installed from a package? The chef-client cannot check to see if the service is running until after the package is installed. A simple question that why-run mode can answer is what the chef-client would say about the state of the service after installing the package because service actions often trigger notifications to other resources. So it can be important to know in advance that any notifications are being triggered correctly._

# ENVIRONMENTS

[About environments](https://docs.chef.io/environments.html)

## WHAT IS AN ENVIRONMENT/USE CASES

- What is the purpose of an Environments?

_An environment is a way to map an organization’s real-life workflow to what can be configured and managed when using Chef server. Every organization begins with a single environment called the `_default` environment, which cannot be modified (or deleted). Additional environments can be created to reflect each organization’s patterns and workflow. For example, creating `production`, `staging`, `testing`, and `development` environments. Generally, an environment is also associated with one (or more) cookbook versions._

- What is the `_default` environment?

_Every organization must have at least one environment. Every organization starts out with a single environment that is named _default, which ensures that at least one environment is always available to the Chef server. The _default environment cannot be modified in any way. Nodes, roles, run-lists, cookbooks (and cookbook versions), and attributes specific to an organization can only be associated with a custom environment._

- What information can be specified in an Environment?

_Nodes, roles, run-lists, cookbooks (and cookbook versions), and attributes specific to an organization can only be associated with a custom environment._

- What happens if you do not specify an Environment?

_It is set to `_default`. Every organization starts out with a single environment that is named `_default`_

- Creating environments in Ruby

```

name 'environment_name'
description 'environment_description'
cookbook OR cookbook_versions  'cookbook' OR 'cookbook' => 'cookbook_version'
default_attributes 'node' => { 'attribute' => [ 'value', 'value', 'etc.' ] }
override_attributes 'node' => { 'attribute' => [ 'value', 'value', 'etc.' ] }

```

- Creating environments in JSON

```

{
  "name": "dev",
  "default_attributes": {
    "apache2": {
      "listen_ports": [
        "80",
        "443"
      ]
    }
  },
  "json_class": "Chef::Environment",
    "description": "",
    "cookbook_versions": {
    "couchdb": "= 11.0.0"
  },
  "chef_type": "environment"
  }

  ```

- Using environments within a search

```
knife search node "chef_environment:QA AND platform:centos"
```

_Or, to include the same search in a recipe, use a code block similar to:_

```
qa_nodes = search(:node,"chef_environment:QA")
qa_nodes.each do |qa_node|
    # Do useful work specific to qa nodes only
end
```

## ATTRIBUTE PRECEDENCE AND COOKBOOK CONSTRAINTS

- What attribute precedence levels are available for Environments

[About environments](https://docs.chef.io/roles.html#attribute-precedence)

_`default` and `override`_

- Overriding Role attributes

`override`

_Applying environment override attributes after role override attributes allows the same role to be used across multiple environments, yet ensuring that values can be set that are specific to each environment (when required)._

- Syntax for setting cookbook constraints.

```
  "cookbook_versions": {
    "couchdb": "= 11.0.0"
  },
  ```

- How would you allow only patch updates to a cookbook within an environment?

```
  "cookbook_versions": {
    "couchdb": "~> 11.0.0"
  },
  ```

## SETTING AND VIEWING ENVIRONMENTS

[About environments](https://docs.chef.io/roles.html#attribute-precedence)

[knife environment](https://docs.chef.io/knife_environment.html)

- How can you list Environments?

```
$ knife environment list
```

- How can you move a node to a specific Environment?

```
$ knife node environment_set NODE_NAME ENVIRONMENT_NAME (options)
```

- Using `knife exec` to bulk change Environments.

```
knife exec -E "nodes.transform(“chef_environment:dev“) \
  {|n| puts n.run_list.remove(“recipe[chef-client::upgrade]“); n.save }"
```

- Using 'chef_environment' global variable in recipes

```
if node.chef_environment == "dev"
  # stuff
end
```

- Environment specific knife plugins, e.g. `knife flip`

[Community plugins](https://docs.chef.io/plugin_community.html)

`knife-flip` - _A knife plugin to move a node, or all nodes in a role, to a specific environment_

`knife-bulkchangeenv` - _A plugin for Chef::Knife which lets you move all nodes in one environment into another._

`knife-env-diff` - _Adds the ability to diff the cookbook versions for two (or more) environments._

`knife-set-environment` - _Adds the ability to set a node environment._

`knife-spork` - _Adds a simple environment workflow so that teams can more easily work together on the same cookbooks and environments._

`knife-whisk` - _Adds the ability to create new servers in a team environment._

- Bootstrapping a node into a particular Environment

```
knife bootstrap <IPorFQDN> --run-list 'cookbook::default' --environment 'dev' -x 'annie' -i '~/.ssh/id_rsa' --
sudo -N 'prep-node'
```

# ROLES

[Roles](https://docs.chef.io/roles.html)

## USING ROLES

- What is the purpose of a Role?

_A role is a way to define certain patterns and processes that exist across nodes in an organization as belonging to a single job function. Each role consists of zero (or more) attributes and a run-list. Each node can have zero (or more) roles assigned to it. When a role is run against a node, the configuration details of that node are compared against the attributes of the role, and then the contents of that role’s run-list are applied to the node’s configuration details. When a chef-client runs, it merges its own attributes and run-lists with those contained within each assigned role._

- Creating Roles

_There are several ways to manage roles:_

_knife can be used to create, edit, view, list, tag, and delete roles._

_The Chef management console add-on can be used to create, edit, view, list, tag, and delete roles. In addition, role attributes can be modified and roles can be moved between environments._

_The chef-client can be used to manage role data using the command line and JSON files (that contain a hash, the elements of which are added as role attributes). In addition, the run_list setting allows roles and/or recipes to be added to the role._

_The open source Chef server can be used to manage role data using the command line and JSON files (that contain a hash, the elements of which are added as role attributes). In addition, the run_list setting allows roles and/or recipes to be added to the role._

_The Chef server API can be used to create and manage roles directly, although using knife and/or the Chef management console is the most common way to manage roles._

_The command line can also be used with JSON files and third-party services, such as Amazon EC2, where the JSON files can contain per-instance metadata stored in a file on-disk and then read by chef-solo or chef-client as required._

- Role Ruby & JSON DSL formats

```
name "webserver"
description "The base role for systems that serve HTTP traffic"
run_list "recipe[apache2]", "recipe[apache2::mod_ssl]", "role[monitor]"
env_run_lists "prod" => ["recipe[apache2]"], "staging" => ["recipe[apache2::staging]"], "_default" => []
default_attributes "apache2" => { "listen_ports" => [ "80", "443" ] }
override_attributes "apache2" => { "max_children" => "50" }
```

```
{
  "name": "webserver",
  "chef_type": "role",
  "json_class": "Chef::Role",
  "default_attributes": {
    "apache2": {
      "listen_ports": [
        "80",
        "443"
      ]
    }
  },
  "description": "The base role for systems that serve HTTP traffic",
  "run_list": [
    "recipe[apache2]",
    "recipe[apache2::mod_ssl]",
    "role[monitor]"
  ],
  "env_run_lists" : {
    "production" : [],
    "preprod" : [],
    "dev": [
      "role[base]",
      "recipe[apache]",
      "recipe[apache::copy_dev_configs]",
    ],
    "test": [
      "role[base]",
      "recipe[apache]"
    ]
  },
  "override_attributes": {
    "apache2": {
      "max_children": "50"
    }
  }
}
```

- Pros and Cons of Roles

_PROS: A role is a way to define certain patterns and processes that exist across nodes in an organization as belonging to a single job function_

[Policyfile](https://docs.chef.io/policyfile.html)

_CONS: NO versioning for Roles. When running Chef without Policyfile roles are global objects. Changes to roles are applied immediately to any node that contains that role in its run-list. This can make it hard to update roles and in some use cases discourages using roles at all._

_Policyfile effectively replaces roles. Policyfile files are versioned automatically and new versions are applied to systems only when promoted._

- The Role cookbook pattern

[Writing wrapper cookbooks](https://blog.chef.io/2017/02/14/writing-wrapper-cookbooks/)

- Creating Role cookbooks

_Eventually you will have multiple base cookbooks and you may want to combine them into a single logical unit, so that can be tested together. Take for example a cookbook called role_my_company_website. This cookbook’s default recipe might look like the following:_

```
include_recipe 'my_company_windows_base::default'
include_recipe 'my_company_audit::default'
include_recipe 'my_company_iis::default'
include_recipe 'my_company_website::default'
```

- Using Roles within a search

```
knife search role '*:*'
```

## SETTING ATTRIBUTES AND ATTRIBUTE PRECEDENCE

- What attribute precedence levels are available for Roles

[About roles](https://docs.chef.io/roles.html#attribute-precedence)

_`default` and `override`_

- Setting attribute precedence

_The attribute precedence order for roles and environments is reversed for `default` and `override` attributes. The precedence order for `default` attributes is environment, then role. The precedence order for `override` attributes is role, then environment. Applying environment `override` attributes after role `override` attributes allows the same role to be used across multiple environments, yet ensuring that values can be set that are specific to each environment (when required). For example, the role for an application server may exist in all environments, yet one environment may use a database server that is different from other environments._

## BASE ROLE & NESTED ROLES

- What are nested roles?

[Nested roles](http://blog.afistfulofservers.net/post/2011/03/16/a-brief-chef-tutorial-from-concentrate/)

> Nested role is a role with another role included into its run_list

```
role[demo]
  role[base]                   <---- nested role
  recipe[foo::server]
  recipe[foo::muninplugin]
```

- Whats the purpose of a `base` role?

_Base role can be used as a starting point to create another run_list_

## USING KNIFE

- The `knife role` command

[knife role](https://docs.chef.io/knife_role.html)

_The knife role subcommand is used to manage the roles that are associated with one or more nodes on a Chef server._

- How would you set the run_list for a node using `knife`?

```
knife node run_list add NODE_NAME RUN_LIST_ITEM (options)
```

- Listing nodes

```
knife node list (options)
```

- View role details

```
knife role show ROLE_NAME
```

- Using Roles within a search

```
knife search node 'platform:windows AND roles:jenkins'
```

_To search a top-level run-list for a role named load_balancer use the knife search subcommand from the command line or the search method in a recipe. For example:_

```
$ knife search node role:load_balancer
```

and from within a recipe:

```
search(:node, 'role:load_balancer')
```

_To search an expanded run-list for all nodes with the role load_balancer use the knife search subcommand from the command line or the search method in a recipe. For example:_

```
$ knife search node roles:load_balancer
```

_and from within a recipe:_

```
search(:node, 'roles:load_balancer')
```

# UPLOADING COOKBOOKS TO CHEF SERVER

## USING BERKSHELF

[About Berkshelf](https://docs.chef.io/berkshelf.html)

- The Berksfile & Berksfile.lock

_A Berksfile describes the set of sources and dependencies needed to use a cookbook. It is used in conjunction with the berks command._

_A Berksfile is a Ruby file, in which sources, dependencies, and options may be specified. Berksfiles are modelled closely on Bundler’s Gemfile. The syntax is as follows:_

```
source "https://supermarket.chef.io"
metadata
cookbook "NAME" [, "VERSION_CONSTRAINT"] [, SOURCE_OPTIONS]
```

_Running the install command also creates a `Berksfile.lock`, which represents exactly which cookbook versions Berkshelf installed. This file ensures that someone else can check the cookbook out of git and get exactly the same dependencies as you._

_Use `berks install` to install cookbooks into the cache. This command generates the Berkshelf lock file that ensures consistency._

- `berks install` and `berks upload`

_When you run `berks install`, cookbooks mentioned as `depends` will be downloaded from Supermarket into the cache_

_You can upload all cookbooks to your Chef server with berks upload:_

```
$ berks upload
```

- Where are dependant cookbooks stored locally?

```
~/.berkshelf/cookbooks/
```

- Limitations of using knife to upload cookbooks



- Listing cookbooks on Chef Server using knife

```
$ knife cookbook list (options)
```

_This argument has the following options:_

`-a`, `--all`
_Return all available versions for every cookbook._

`-w`, `--with-uri`
_Show the corresponding URIs._

- What happens if you try to upload the same version of a cookbook?

[StackOverFlow QA](https://stackoverflow.com/questions/28957578/chef-cookbook-version-delete-or-update-specific-version)

_When you run knife cookbook upload it will update whatever version is listed in your local metadata.rb file. (unless it is frozen). If you use Berks, you'll need to add the --force option to overwrite an existing version._

- How can you upload the same version of a cookbook?

```
$ knife cookbook upload --all --cookbook-path cookbooks
Uploading build-essential [1.3.4]
Uploading chef-server    [2.0.0]
Uploading python         [1.4.1]
Uploading yum            [2.1.0]
ERROR: Version 1.3.4 of cookbook build-essential is frozen. Use --force to override.
WARNING: Not updating version constraints for some cookbooks in the environment as the cookbook is frozen.
Uploaded all cookbooks.
```

- Downloading cookbooks from Chef Server

```
$ knife cookbook download COOKBOOK_NAME [COOKBOOK_VERSION] (options)
```

_This argument has the following options:_

`-d DOWNLOAD_DIRECTORY`, `--dir DOWNLOAD_DIRECTORY`
_The directory in which cookbooks are located._

`-f`, `--force`
_Overwrite an existing directory._

`-N`, `--latest`
_Download the most recent version of a cookbook._

- Bulk uploading cookbooks using knife

```
$ knife cookbook upload [COOKBOOK_NAME...] (options)
```

`-a`, `--all`
_Upload all cookbooks._

_Upload the /cookbooks directory. Browse to the top level of the chef-repo and enter:_

```
$ knife upload cookbooks
```

_or from anywhere in the chef-repo, enter:_

```
$ knife upload /cookbooks
```

# USING KNIFE

## BASIC KNIFE USAGE

- How does knife know what Chef Server to interact with?

_A knife.rb file is used to specify configuration details for knife._

`chef_server_url`
_The URL for the Chef server. For example:_

```
chef_server_url 'https://localhost/organizations/ORG_NAME'
```

- How does knife authenticate with Chef Server

[Authentication, Authorization](https://docs.chef.io/auth.html#api-requests)

_During the initial chef-client run, the chef-client will register with the Chef server using the private key assigned to the chef-validator, after which the chef-client will obtain a client.pem private key for all future authentication requests to the Chef server._

_RSA public key-pairs are used to authenticate knife with the Chef server every time knife attempts to access the Chef server. This ensures that each instance of knife is properly registered with the Chef server and that only trusted users can make changes to the data._

`validation_key`
_The location of the file that contains the key used when a chef-client is registered with a Chef server. A validation key is signed using the validation_client_name for authentication. Default value: `/etc/chef/validation.pem`. For example:_

```
validation_key '/etc/chef/validation.pem'
```

`client_key`
_The location of the file that contains the client key. Default value: `/etc/chef/client.pem`. For example:_

```
client_key '/etc/chef/client.pem'
```

- How/When would you use `knife ssh` & `knife winrm`?

_Use the knife ssh subcommand to invoke SSH commands (in parallel) on a subset of nodes within an organization, based on the results of a search query made to the Chef server._

```
$ knife ssh "role:webserver" "sudo chef-client"
```

_The knife windows subcommand is used to configure and interact with nodes that exist on server and/or desktop machines that are running Microsoft Windows. Nodes are configured using WinRM, which allows native objects—batch scripts, Windows PowerShell scripts, or scripting library variables—to be called by external applications._

- Verifying ssl certificate authenticity using knife

_The  SSL certificates that are used by the chef-client may be verified by specifying the path to the `client.rb` file. Use the `--config` option (that is available to any knife command) to specify this path:_

```
$ knife ssl check --config /etc/chef/client.rb
```

_Verify an external server’s SSL certificate_

```
$ knife ssl check URL_or_URI
```

_for example:_

```
$ knife ssl check https://www.chef.io
```

- Where can/should you run the `knife` command from?

> From workstation, from your repo directory

## KNIFE CONFIGURATION

- How/where do you configure knife?

`.chef/knife.rb`

- Common options - cookbook_path, validation_key, chef_server_url, validation_key

[config.rb](https://docs.chef.io/config_rb_knife.html)

```
current_dir = File.dirname(__FILE__)
log_level                :info
log_location             STDOUT
node_name                "nodename"
client_key               "#{current_dir}/user.pem"
chef_server_url          "https://api.chef.io/organizations/orgname"
cookbook_path            ["#{current_dir}/../cookbooks"]
```

- Setting the chef-client version to be installed during bootstrap

```
$ knife bootstrap FQDN_or_IP_ADDRESS (options)
```

`--bootstrap-version VERSION` _- The version of the chef-client to install_.

- Setting defaults for command line options in knife’s configuration file

To add settings to the `knife.rb` file, use the following syntax:
```
knife[:setting_name] = value
```
where value may require quotation marks (‘ ‘) if that value is a string. For example:
```
knife[:ssh_port] = 22
knife[:bootstrap_template] = 'ubuntu14.04-gems'
knife[:bootstrap_version] = ''
knife[:bootstrap_proxy] = ''
```

[knife configure](https://docs.chef.io/knife_configure.html)

_Use the knife configure subcommand to create the knife.rb and client.rb files so that they can be distributed to workstations and nodes._

- Using environment variables and sharing knife configuration file with your team

_Add environment variables to the `knife.rb` and share the file among your team. For example:_

```
current_dir = File.dirname(__FILE__)
  user = ENV['OPSCODE_USER'] || ENV['USER']
  node_name                user
  client_key               "#{ENV['HOME']}/chef-repo/.chef/#{user}.pem"
  validation_client_name   "#{ENV['ORGNAME']}-validator"
  validation_key           "#{ENV['HOME']}/chef-repo/.chef/#{ENV['ORGNAME']}-validator.pem"
  chef_server_url          "https://api.opscode.com/organizations/#{ENV['ORGNAME']}"
  syntax_check_cache_path  "#{ENV['HOME']}/chef-repo/.chef/syntax_check_cache"
  cookbook_path            ["#{current_dir}/../cookbooks"]
  cookbook_copyright       "Your Company, Inc."
  cookbook_license         "apachev2"
  cookbook_email           "cookbooks@yourcompany.com"

  # Amazon AWS
  knife[:aws_access_key_id] = ENV['AWS_ACCESS_KEY_ID']
  knife[:aws_secret_access_key] = ENV['AWS_SECRET_ACCESS_KEY']
```

- Managing proxies

_In an environment that requires proxies to reach the Internet, many Chef commands will not work until they are configured correctly. To configure Chef to work in an environment that requires proxies, set the `http_proxy`, `https_proxy`, `ftp_proxy`, and/or `no_proxy` environment variables to specify the proxy settings using a lowercase value._

## KNIFE PLUGINS

- Common knife plugins

_Knife functionality can be extended with plugins, which work the same as built-in subcommands (including common options). Knife plugins have been written to interact with common cloud providers, to simplify common Chef tasks, and to aid in Chef workflows._ `chef gem install PLUGIN_NAME`

```
knife-acl
knife-azure
knife-ec2
knife-eucalyptus
knife-google
knife-linode
knife-lpar
knife-openstack
knife-push
knife-rackspace
knife-vcenter
knife-windows
```

- What is 'knife ec2' plugin?

_The knife ec2 subcommand is used to manage API-driven cloud servers that are hosted by Amazon EC2. For example:_

```
knife ec2 server create -r 'role[webserver]' -I ami-cd0fd6be -f t2.micro --aws-access-key-id 'Your AWS Access Key ID' --aws-secret-access-key "Your AWS Secret Access Key"`
```

- What is 'knife windows' plugin?

_The `knife windows` subcommand is used to configure and interact with nodes that exist on server and/or desktop machines that are running Microsoft Windows. Nodes are configured using WinRM, which allows native objects—batch scripts, Windows PowerShell scripts, or scripting library variables—to be called by external applications. The `knife windows` subcommand supports NTLM and Kerberos methods of authentication. For example:_

```
knife bootstrap windows winrm web1.cloudapp.net -r 'server::web' -x 'proddomain\webuser' -P 'password'
```

- What is ‘knife block’ plugin?

https://github.com/knife-block/knife-block

_The `knife block` plugin has been created to enable the use of multiple knife.rb files against multiple chef servers. The premise is that you have a "block" in which you store all your "knives" and you can choose the one best suited to the task. Knife looks for `knife.rb` in `~/.chef` - all this script does is create a symlink from the required configuration to `knife.rb` so that knife can act on the appropriate server. For example:_

```
knife block use <server_name>
```

- What is ‘knife spork’ plugin?

_The `knife-spork` plugin adds workflow that enables multiple developers to work on the same Chef server and repository, but without stepping on each other’s toes. This plugin is designed around the workflow at Etsy, where several people work in the same repository and Chef server simultaneously. For example:_

```
knife spork bump
```

- Installing knife plugins

```
chef gem install PLUGIN_NAME
```

## TROUBLESHOOTING

- Troubleshooting Authentication

[Troubleshooting](https://docs.chef.io/errors.html)

_There are multiple causes of the Chef 401 “Unauthorized” error, so please use the sections by the link above to find the error message that most closely matches your output._

*Failed to authenticate as ORGANIZATION-validator*

```
INFO: Client key /etc/chef/client.pem is not present - registering
INFO: HTTP Request Returned 401 Unauthorized: Failed to authenticate as ORGANIZATION-validator. Ensure that your node_name and client key are correct.
FATAL: Stacktrace dumped to c:/chef/cache/chef-stacktrace.out
FATAL: Net::HTTPServerException: 401 "Unauthorized"
```

_1) Check if the ORGANIZATION-validator.pem file exists in one of the following locations:_

```
~/.chef
~/projects/current_project/.chef
/etc/chef
```

_2) If there’s no ORGANIZATION-validator.pem file, regenerate it._

*Failed to authenticate to https://api.opscode.com*

```
ERROR: Failed to authenticate to https://api.opscode.com/organizations/ORGANIZATION as USERNAME with key /path/to/USERNAME.pem
Response:  Failed to authenticate as USERNAME. Ensure that your node_name and client key are correct.
```

_1) Verify you have the correct values in your knife.rb file, especially for the node_name and client_key settings._

_2) Check if the file referenced in the client_key setting (usually USER.pem) exists_

_3) If there’s no client.rb file, regenerate it and ensure the values for the node_name and client_key settings are correct._

*Organization not found*

_If you see this error when trying to recreate the ORGANIZATION-validator.pem, it’s possible that the chef-client itself was deleted. In this situation, the ORGANIZATION-validator.pem will need to be recreated_

*Synchronize the clock on your host*

_If the system clock drifts more than 15 minutes from the actual time, the following type of error will be shown:_

```
INFO: Client key /etc/chef/client.pem is not present - registering
INFO: HTTP Request Returned 401 Unauthorized: Failed to authenticate as ORGANIZATION-validator. Synchronize the clock on your host.
FATAL: Stacktrace dumped to /var/chef/cache/chef-stacktrace.out
FATAL: Net::HTTPServerException: 401 "Unauthorized"
```

*All other 401 errors*

_1) Make sure your client.pem is valid._

_2) Make sure to use the same node_name as the initial chef-client run._
_Run chef-client -l debug_

*403 Forbidden*

```
FATAL: Stacktrace dumped to /var/chef/cache/chef-stacktrace.out
FATAL: Net::HTTPServerException: 403 "Forbidden"
```

_In Chef, there are two different types of permissions issues, object specific and global permissions. To figure out which type of permission issue you’re experiencing, run the `chef-client` again using the `-l debug` options to see debugging output._

```
DEBUG: Sending HTTP Request to https://api.opscode.com/organizations/ORGNAME/nodes
ERROR: Running exception handlers
```

_1) Fix the global permissions: Log in to the Chef management console and click on the failing object type (most likely Nodes)_

_2) fix object permissions: Log in to the Chef management console and click on the failing object type (most likely Nodes). Click on the object in the list that is causing the error._

*500 (Unexpected)*

_HTTP 500 is a non-specific error message. The full error message for the error the chef-client is receiving can be found in one of the following log ﬁles:_

```
/var/log/opscode/opscode-account/current
/var/log/opscode/opscode-erchef/current
```

*502 / 504 (Gateway)*

```
$ grep 'HTTP/1.1" 504' /var/log/opscode/nginx/access.log
$ grep 'HTTP/1.1" 504' nginx-access.log | cut -d' ' -f8 | sort | uniq -c | sort
$ tail -10000 nginx-access.log | grep 'HTTP/1.1" 504' | cut -d' ' -f8 | sort | uniq -c | sort
```

*Cannot find config file*

_Work around this issue by supplying the full path to the client.rb file:_

```
$ chef-client -c /etc/chef/client.rb
```

_When the values for certain settings in the client.rb file—node_name and client_key—are incorrect, it will not be possible to authenticate to the Chef server._

- Using `knife ssl check` command

[knife ssl](https://docs.chef.io/knife_ssl_check.html)

_Use the `knife ssl check` subcommand to verify the SSL configuration for the Chef server or a location specified by a URL or URI. Invalid certificates will not be used by OpenSSL. When this command is run, the certificate files (`*.crt` and/or `*.pem`) that are located in the `/.chef/trusted_certs` directory are checked to see if they have valid X.509 certificate properties. A warning is returned when certificates do not have valid X.509 certificate properties or if the `/.chef/trusted_certs` directory does not contain any certificates._

```
knife ssl check (options)
```

- Using `knife ssl fetch` command

_Run the `knife ssl fetch` to download the self-signed certificate from the Chef server to the `/.chef/trusted_certs` directory on a workstation._

- Using '-VV' flag

`-V`, `--verbose` _Set for more verbose outputs. Use `-VV` for maximum verbosity_.

- Setting log levels and log locations

[Log](https://docs.chef.io/resource_log.html)

[Server logs](https://docs.chef.io/server_logs.html)

_Use the log resource to create log entries. The log resource behaves like any other resource: built into the resource collection during the compile phase, and then run during the execution phase. (To create a log entry that is not built into the resource collection, use `Chef::Log` instead of the log resource.)_

```
log 'message' do
  message 'A message add to the log.'
  level :info
end
```

_Log Level	Syntax_

`Fatal`	`Chef::Log.fatal('string')`

`Error`	`Chef::Log.error('string')`

`Warn`	`Chef::Log.warn('string')`

`Info`	`Chef::Log.info('string')`

`Debug`	`Chef::Log.debug('string')`

_Set default logging level_

```
log 'a string to log'
```

_Set debug logging level_

```
log 'a debug string' do
  level :debug
end
```

_Add a message to a log file_

```
log 'message' do
  message 'This is the message that will be added to the log.'
  level :info
end
```

_All logs generated by the Chef server can be found in `/var/log/opscode`. Each service enabled on the system also has a sub-directory in which service-specific logs are located, typically found in `/var/log/opscode/service_name`._

```
/var/log/opscode
/var/log/opscode/service_name
```

_View Log Files_

```
$ chef-server-ctl tail
$ chef-server-ctl tail SERVICENAME
$ tail -50f /var/log/chef-server/opscode-chef/current
```

_opscode-erchef, current, erchef_

```
$ ls -lrt /var/log/opscode/opscode-erchef/erchef.log.*
```

_nginx_

```
/var/log/opscode/nginx/
$ chef-server-ctl tail nginx
```

# BOOTSTRAPPING

## USING KNIFE

- Common ‘knife bootstrap’ options - UserName, Password, RunList, and Environment

[knife bootstrap](https://docs.chef.io/knife_bootstrap.html)

`-x USERNAME`, `--ssh-user USERNAME` - _The SSH user name._

`-P PASSWORD`, `--ssh-password PASSWORD` - _The SSH password. This can be used to pass the password directly on the command line. If this option is not specified (and a password is required) knife prompts for the password._

`-r RUN_LIST`, `--run-list RUN_LIST` - _A comma-separated list of roles and/or recipes to be applied._

`-E ENVIRONMENT`, `--environment ENVIRONMENT` - _The name of the environment. When this option is added to a command, the command will run only against the named environment._

- Using `winrm` & `ssh`

_Use the `knife winrm` argument to create a connection to one or more remote machines. As each connection is created, a password must be provided. This argument uses the same syntax as the search subcommand. WinRM requires that a target node be accessible via the ports configured to support access via HTTP or HTTPS._

```
knife winrm SEARCH_QUERY SSH_COMMAND (options)
```

_Use the `knife ssh` subcommand to invoke SSH commands (in parallel) on a subset of nodes within an organization, based on the results of a search query made to the Chef server._

```
knife ssh SEARCH_QUERY SSH_COMMAND (options)
```

- Using knife plugins for bootstrap - `knife ec2 ..`, `knife bootstrap windows ...`

_Amazon EC2 is a web service that provides resizable compute capacity in the cloud, based on preconfigured operating systems and virtual application software using Amazon Machine Images (AMI). The knife ec2 subcommand is used to manage API-driven cloud servers that are hosted by Amazon EC2._

```
knife ec2 server create
```

_Use the `bootstrap windows winrm` argument to bootstrap chef-client installations in a Microsoft Windows environment, using WinRM and the WS-Management protocol for communication. This argument requires the FQDN of the host machine to be specified. The Microsoft Installer Package (MSI) run silently during the bootstrap operation (using the /qn option). For example:_

```
knife bootstrap windows winrm FQDN
```

## BOOTSTRAP OPTIONS

- Validator' vs 'Validatorless' Bootstraps

_The ORGANIZATION-validator.pem is typically added to the .chef directory on the workstation. When a node is bootstrapped from that workstation, the ORGANIZATION-validator.pem is used to authenticate the newly-created node to the Chef server during the initial chef-client run. Starting with Chef client 12.1, it is possible to bootstrap a node using the USER.pem file instead of the ORGANIZATION-validator.pem file. This is known as a “validatorless bootstrap”._

- Bootstrapping in FIPS mode

_If you have FIPS compliance enabled at the kernel level then chef-client will default to running in FIPS mode. Otherwise you can add `fips true` to the `/etc/chef/client.rb` or `C:\\chef\\client.rb.`_

_Bootstrap a node using FIPS_

```
knife bootstrap 192.0.2.0 -P vanilla -x root -r 'recipe[apt],recipe[xfs],recipe[vim]' --fips
```

- What are Custom Templates

[knife bootstrap](https://docs.chef.io/knife_bootstrap.html#custom-templates)

_The default `chef-full` template uses the omnibus installer. For most bootstrap operations, regardless of the platform on which the target node is running, using the `chef-full` distribution is the best approach for installing the chef-client on a target node. In some situations, a custom template may be required._

_For example, the default bootstrap operation relies on an Internet connection to get the distribution to the target node. If a target node cannot access the Internet, then a custom template can be used to define a specific location for the distribution so that the target node may access it during the bootstrap operation._

_You can use the --bootstrap-template option with the knife bootstrap subcommand to specify the name of your bootstrap template file:_

```
$ knife bootstrap 123.456.7.8 -x username -P password --sudo --bootstrap-template "template"
```

## UNATTENDED INSTALLS

- Configuring Unattended Installs

[Bootstrap a Node](https://docs.chef.io/install_bootstrap.html#bootstrapping-with-user-data)

_When the chef-client is installed using an unattended bootstrap, it may be built into an image that starts the chef-client on boot, or installed using User Data or some other kind of post-deployment script. The type of image or User Data used depends on the platform on which the unattended bootstrap will take place._

_The method used to inject a user data script into a server will vary depending on the infrastructure platform being used. For example, on AWS you can pass this data in as a text file using the command line tool._

_Setting the initial run-list_

_A node’s initial run-list is specified using a JSON file on the host system. When running the chef-client as an executable, use the -j option to tell the chef-client which JSON file to use. For example:_

```
$ chef-client -j /etc/chef/first-boot.json --environment _default
```

- What conditions must exists for unattended install to take place?

_Must be able to authenticate to the Chef server_

_Must be able to configure a run-list_

_May require custom attributes, depending on the cookbooks that are being used_

_Must be able to access the `chef-validator.pem` so that it may create a new identity on the Chef server_

_Must have a unique node name; the chef-client will use the FQDN for the host system by default_

_It is important that settings in the `client.rb` file —`chef_server_url`, `http_proxy`, and so on are used—to ensure that configuration details are built into the unattended bootstrap process._

## FIRST CHEF-CLIENT RUN

[Chef client overview](https://docs.chef.io/chef_client_overview.html)

- How does authentication work during the first chef-client run?

_During the first chef-client run, the private key `/etc/chef/client.pem` does not exist. Instead, the `chef-client` will attempt to use the private key assigned to the `chef-validator`, located in `/etc/chef/validation.pem`. (If, for any reason, the `chef-validator` is unable to make an authenticated request to the Chef server, the initial `chef-client` run will fail.) During the initial `chef-client` run, the `chef-client` will register with the Chef server using the private key assigned to the `chef-validator`, after which the `chef-client` will obtain a `client.pem` private key for all future authentication requests to the Chef server._

- What is ‘ORGANIZATION-validator.pem’ file and when is it used?

_During the first chef-client run, the private key `/etc/chef/client.pem` does not exist. Instead, the `chef-client` will attempt to use the private key assigned to the `chef-validator`, located in `/etc/chef/validation.pem`. (If, for any reason, the `chef-validator` is unable to make an authenticated request to the Chef server, the initial `chef-client` run will fail.) During the initial `chef-client` run, the `chef-client` will register with the Chef server using the private key assigned to the `chef-validator`, after which the `chef-client` will obtain a `client.pem` private key for all future authentication requests to the Chef server._

- What is the ‘first-boot.json’ file?

_On UNIX- and Linux-based machines: The second shell script executes the `chef-client` binary with a set of initial settings stored within `first-boot.json` on the node. `first-boot.json` is generated from the workstation as part of the initial `knife bootstrap` subcommand._

_On Microsoft Windows machines: The batch file that is derived from the `windows-chef-client-msi.erb` bootstrap template executes the `chef-client` binary with a set of initial settings stored within `first-boot.json` on the node. `first-boot.json` is generated from the workstation as part of the initial `knife bootstrap` subcommand._

# POLICY FILES

[About policyfile](https://docs.chef.io/policyfile.html)

## BASIC KNOWLEDGE AND USAGE

- What are policy files, and what problems do they solve?

_A Policyfile is an optional way to manage role, environment, and community cookbook data with a single document that is uploaded to the Chef server. The file is associated with a group of nodes, cookbooks, and settings. When these nodes perform a Chef client run, they utilize recipes specified in the Policyfile run-list._

- Policy file use cases?

_For some users of Chef, Policyfile will make it easier to test and promote code safely with a simpler interface. Policyfile improves the user experience and resolves real-world problems that some workflows built around Chef must deal with. The following sections discuss in more detail some of the good reasons to use Policyfile, including:_

_- Focus the workflow on the entire system_

_- Safer development workflows_

_- Less expensive computation_

_- Code visibility_

_- Role mutability_

_- Cookbook mutability_

_- Replaces Berkshelf and the environment cookbook pattern_

- What can/not be configured in a policy file?

```
name "jenkins-master"
run_list "java", "jenkins::master", "recipe[policyfile_demo]"
default_source :supermarket, "https://mysupermarket.example"
cookbook "policyfile_demo", path: "cookbooks/policyfile_demo"
cookbook "jenkins", "~> 2.1"
cookbook "mysql", github: "chef-cookbooks/mysql", branch: "master"
```

- Policy files and Chef Workflow

_Focused System Workflows: The knife command line tool maps very closely to the Chef server API and the objects defined by it: roles, environments, run-lists, cookbooks, data bags, nodes, and so on. The chef-client assembles these pieces at run-time and configures a host to do useful work._

_Policyfile focuses that workflow onto the entire system, rather than the individual components. For example, Policyfile describes whole systems, whereas each individual revision of the Policyfile.lock.json file uploaded to the Chef server describes a part of that system, inclusive of roles, environments, cookbooks, and the other Chef server objects necessary to configure that part of the system._

_Safer Workflows: Policyfile encourages safer workflows by making it easier to publish development versions of cookbooks to the Chef server without the risk of mutating the production versions and without requiring a complicated versioning scheme to work around cookbook mutability issues. Roles are mutable and those changes are applied only to the nodes specified by the policy. Policyfile does not require any changes to your normal workflows. Use the same repositories you are already using, the same cookbooks, and workflows. Policyfile will prevent an updated cookbook or role from being applied immediately to all machines._

# SEARCH

## BASIC SEARCH USAGE

- What information is indexed and available for search?

_- client_

_- environment_

_- node_

_- role_

_- data bag_

- Search operators and query syntax

_A search query is comprised of two parts: the key and the search pattern. A search query has the following syntax:_ `key:search_pattern`
_where key is a field name that is found in the JSON description of an indexable object on the Chef server (a role, node, client, environment, or data bag) and search_pattern defines what will be searched for, using one of the following search patterns: exact, wildcard, range, or fuzzy matching. Both key and search_pattern are case-sensitive; key has limited support for multiple character wildcard matching using an asterisk (“*”) (and as long as it is not the first character)._

```
knife search INDEX SEARCH_QUERY
```
_defaults to search for node_
```
knife search '*:*' -i
```
_is the same as_
```
$ knife search node '*:*' -i
```

- Wildcards, range and fuzzy matching

_- To search for any node that contains the specified key, enter the following:_

```
knife search node 'foo:*'
```

_- To search using an inclusive range, enter the following:_

```
knife search sample "id:[bar TO foo]"
```

_- To use a fuzzy search pattern enter something similar to:_

```
knife search client "name:boo~"
```

## SEARCH USING KNIFE AND IN A RECIPE

- Knife command line search syntax

```
knife search INDEX SEARCH_QUERY
```

- Recipe search syntax

```
search(:node, "key:attribute")
```

## FILTERING RESULTS

- How do you filter on Chef Server

_Use `:filter_result` as part of a search query to filter the search output based on the pattern specified by a Hash. Only attributes in the Hash will be returned._

_The syntax for the search method that uses :filter_result is as follows:_

```
search(:index, 'query',
  :filter_result => { 'foo' => [ 'abc' ],
                      'bar' => [ '123' ],
                      'baz' => [ 'sea', 'power' ]
                    }
      ).each do |result|
  puts result['foo']
  puts result['bar']
  puts result['baz']
end
```
_where:_

_- `:index` is of name of the index on the Chef server against which the search query will run: `:client`, `:data_bag_name`, `:environment`, `:node`, and `:role`_

_- 'query' is a valid search query against an object on the Chef server_

_- `:filter_result` defines a Hash of values to be returned_

_For example:_

```
search(:node, 'role:web',
  :filter_result => { 'name' => [ 'name' ],
                      'ip' => [ 'ipaddress' ],
                      'kernel_version' => [ 'kernel', 'version' ]
                    }
      ).each do |result|
  puts result['name']
  puts result['ip']
  puts result['kernel_version']
end
```

- Selecting attributes to be returned

_Use :filter\_result as part of a search query to filter the search output based on the pattern specified by a Hash. Only attributes in the Hash will be returned._

# CHEF SOLO

## WHAT CHEF SOLO IS

[About chef solo](https://docs.chef.io/chef_solo.html)

- Advantages & disadvantages of Chef-solo vs Chef Server

_chef-solo is a command that executes chef-client in a way that does not require the Chef server in order to converge cookbooks. chef-solo uses chef-client’s Chef local mode, and does not support the following functionality present in chef-client / server configurations:_

_- Centralized distribution of cookbooks_
_- A centralized API that interacts with and integrates infrastructure components_
_- Authentication or authorization_

- Chef-solo executable and options

```
chef-solo OPTION VALUE OPTION VALUE ...
```

- Cookbooks, nodes and attributes

_chef-solo supports two locations from which cookbooks can be run:

_- A local directory._

_- A URL at which a tar.gz archive is located._

_Unlike chef-client, where the node object is stored on the Chef server, chef-solo stores its node objects as JSON files on local disk. By default, chef-solo stores these files in a nodes folder in the same directory as your cookbooks directory. You can control the location of this directory via the node_path value in your configuration file._

_chef-solo does not interact with the Chef server. Consequently, node-specific attributes must be located in a JSON file on the target system, a remote location (such as Amazon Simple Storage Service (S3)), or a web server on the local network._

- Using Data Bags, Roles & Environments

_chef-solo will look for data bags in /var/chef/data_bags, but this location can be modified by changing the setting in solo.rb._

```
data_bag_path '/var/chef-solo/data_bags'
```

_chef-solo will look for roles in /var/chef/roles, but this location can be modified by changing the setting for role_path in solo.rb._

```
role_path '/var/chef-solo/roles'
```

_chef-solo will look for environments in /var/chef/environments, but this location can be modified by changing the setting for environment_path in solo.rb._

```
environment_path '/var/chef-solo/environments'
```

- Chef-solo run intervals

```
-i SECONDS, --interval SECONDS
```
_The frequency (in seconds) at which the chef-client runs. When the chef-client is run at intervals, `--splay` and `--interval` values are applied before the chef-client run._

 - Retrieving cookbooks from remote locations

```
-r RECIPE_URL, --recipe-url RECIPE_URL
```
_The URL location from which a remote cookbook tar.gz is to be downloaded._

- Chef-solo and node object

_Unlike chef-client, where the node object is stored on the Chef server, chef-solo stores its node objects as JSON files on local disk. By default, chef-solo stores these files in a nodes folder in the same directory as your cookbooks directory. You can control the location of this directory via the node_path value in your configuration file._

# DATA BAGS

## WHAT IS A DATA_BAG

- When might you use a data_bag?

_A data bag is a global variable that is stored as JSON data and is accessible from a Chef server. A data bag is indexed for searching and can be loaded by a recipe or accessed during a search. It is used when different clients need to access the same information._

- Indexing data_bags

_Any search for a data bag (or a data bag item) must specify the name of the data bag and then provide the search query string that will be used during the search. For example, to use knife to search within a data bag named “admin\_data” across all items, except for the “admin\_users” item, enter the following:_

```
knife search admin_data "(NOT id:admin_users)"
```

_Or, to include the same search query in a recipe, use a code block similar to:_

```
search(:admin_data, "NOT id:admin_users")
```

_Data bags can be accessed through the search indexes. Use this approach when more than one data bag item is required or when the contents of a data bag are looped through. The search indexes will bulk-load all of the data bag items, which will result in a lower overhead than if each data bag item were loaded by name._

_To load the secret from a file:_

```
data_bag_item('bag', 'item', IO.read('secret_file'))
```

_To load a single data bag item named admins:_

```
data_bag('admins')
```

_The contents of a data bag item named justin:_

```
data_bag_item('admins', 'justin')
```

- What is a data_bag?

_A data bag is a global variable that is stored as JSON data and is accessible from a Chef server. A data bag is indexed for searching and can be loaded by a recipe or accessed during a search._

- Data_bag and Chef-solo

_chef-solo can load data from a data bag as long as the contents of that data bag are accessible from a directory structure that exists on the same machine as chef-solo. The location of this directory is configurable using the data_bag_path option in the solo.rb file. The name of each sub-directory corresponds to a data bag and each JSON file within a sub-directory corresponds to a data bag item. Search is not available in recipes when they are run with chef-solo; use the data_bag() and data_bag_item() functions to access data bags and data bag items._

## DATA_BAG ENCRYPTION

- How do you encrypt a data_bag

_A data bag item is encrypted using a knife command similar to:_
```
knife data bag create passwords mysql --secret-file /tmp/my_data_bag_key
```
_where “passwords” is the name of the data bag, “mysql” is the name of the data bag item, and “/tmp/my_data_bag_key” is the path to the location in which the file that contains the secret-key is located. knife will ask for user credentials before the encrypted data bag item is saved._

- What is Chef Vault

_chef-vault is a RubyGems package that is included in the Chef development kit. chef-vault allows the encryption of a data bag item by using the public keys of a list of nodes, allowing only those nodes to decrypt the encrypted values. chef-vault uses the knife vault subcommand._

## USING DATA_BAGS

- How do you create a data_bag?

_knife can be used to create data bags and data bag items when the knife data bag subcommand is run with the create argument. For example:_

```
knife data bag create DATA_BAG_NAME (DATA_BAG_ITEM)
```

_knife can be used to update data bag items using the from file argument:_

```
knife data bag from file BAG_NAME ITEM_NAME.json
```

_As long as a file is in the correct directory structure, knife will be able to find the data bag and data bag item with only the name of the data bag and data bag item. For example:_

```
knife data bag from file BAG_NAME ITEM_NAME.json
```

- How can you edit a data_bag

_if you are in the “admins” directory and make changes to the file charlie.json, then to upload that change to the Chef server use the following command:_

```
knife data bag from file admins charlie.json
```
