# Study Guide for DEPLOYING COOKBOOKS BADGE CERTIFICATION EXAM

> The outline is copied from [training.chef.io - Deploying Cookbooks](https://training.chef.io/static/Deploying_Cookbooks.pdf). All the information is copied&pasted from [docs.chef.io documentation portal](https://docs.chef.io).

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
Run the chef-client in local mode. This allows all commands that work against the Chef server to also work against the local chef-repo.
_Local mode is a way to run the chef-client against the chef-repo on a local machine as if it were running against the Chef server. Local mode relies on chef-zero, which acts as a very lightweight instance of the Chef server. chef-zero reads and writes to the chef_repo_path, which allows all commands that normally work against the Chef server to be used against the local chef-repo._

`--audit-mode`
_The chef-client may be run in audit-mode. Use audit-mode to evaluate custom rules—also referred to as audits—that are defined in recipes_

`--fips`
_Federal Information Processing Standards (FIPS) is a United States government computer security standard that specifies security requirements for cryptography_

_The chef-client can be run as a daemon. Use the chef-client cookbook to configure the chef-client as a daemon. Add the default recipe to a node’s run-list, and then use attributes in that cookbook to configure the behavior of the chef-client._

- Configuring chef-client
[client.rb](https://docs.chef.io/config_rb_client.html)

_A client.rb file is used to specify the configuration details for the chef-client.

This file is loaded every time this executable is run
On UNIX- and Linux-based machines, the default location for this file is /etc/chef/client.rb; on Microsoft Windows machines, the default location for this file is C:\chef\client.rb; use the --config option from the command line to change this location
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

> Note: Use the chef_gem resource to install gems that are needed by the chef-client during the execution phase.

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

_The authentication process ensures the Chef server responds only to requests made by trusted users. Public key encryption is used by the Chef server. When a node and/or a workstation is configured to run the chef-client, both public and private keys are created. The public key is stored on the Chef server, while the private key is returned to the user for safe keeping. (The private key is a .pem file located in the .chef directory or in /etc/chef.)

Both the chef-client and knife use the Chef server API when communicating with the Chef server. The chef-validator uses the Chef server API, but only during the first chef-client run on a node.

Each request to the Chef server from those executables sign a special group of HTTP headers with the private key. The Chef server then uses the public key to verify the headers and verify the contents._

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
_Libraries -> Attributes -> LWRP -> Definitions -> Recipes
Libraries are loaded first to ensure that all language extensions and Ruby classes are available to all resources. Next, attributes are loaded, followed by lightweight resources, and then all definitions (to ensure that any pseudo-resources within definitions are available). Finally, all recipes are loaded in the order specified by the expanded run-list._

## CONVERGENCE

- What happens during the execute phase of the chef-client run?

[Compile vs Execute - Common functionality](https://docs.chef.io/resource_common.html)
_Each resource is executed in the order identified by the run-list, and then by the order in which each resource is listed in each recipe. Each resource in the resource collection is mapped to a provider. The provider examines the node, and then does the steps necessary to complete the action. And then the next resource is processed. Each action configures a specific part of the system. This process is also referred to as convergence. This is also referred to as the “execution phase”._

- The test/repair model

_Each resource in the resource collection is mapped to a provider. The provider examines the node (tests), and then does the steps necessary to complete the action (repairs)_

- When do notifications get invoked?
[Common resources](https://docs.chef.io/resource_common.html#notifications)

_A notification is a property on a resource that listens to other resources in the resource collection and then takes actions based on the notification type (notifies or subscribes)_

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

_-W, --why-run
Run the executable in why-run mode, which is a type of chef-client run that does everything except modify the system. Use why-run mode to understand why the chef-client makes the decisions that it makes and to learn more about the current and proposed state of the system._

- What are limitations of doing a why run?

_If the action is :start and the service is not running, then start the service (if it is not running) and do nothing (if it is running). What about a service that is installed from a package? The chef-client cannot check to see if the service is running until after the package is installed. A simple question that why-run mode can answer is what the chef-client would say about the state of the service after installing the package because service actions often trigger notifications to other resources. So it can be important to know in advance that any notifications are being triggered correctly._

# ENVIRONMENTS

## WHAT IS AN ENVIRONMENT/USE CASES
- What is the purpose of an Environments?
- What is the '\_default' environment?
- What information can be specified in an Environment?
- What happens if you do not specify an Environment?
- Creating environments in Ruby
- Creating environments in JSON
- Using environments within a search

## ATTRIBUTE PRECEDENCE AND COOKBOOK CONSTRAINTS
- What attribute precedence levels are available for Environments
- Overriding Role attributes
- Syntax for setting cookbook constraints.
- How would you allow only patch updates to a cookbook within an environment?

## SETTING AND VIEWING ENVIRONMENTS
- How can you list Environments?
- How can you move a node to a specific Environment?
- Using `knife exec` to bulk change Environments.
- Using 'chef_environment' global variable in recipes
- Environment specific knife plugins, e.g. `knife flip`
- Bootstrapping a node into a particular Environment

# ROLES

## USING ROLES
- What is the purpose of a Role?
- Creating Roles
- Role Ruby & JSON DSL formats
- Pros and Cons of Roles
- The Role cookbook pattern
- Creating Role cookbooks
- Using Roles within a search

## SETTING ATTRIBUTES AND ATTRIBUTE PRECEDENCE
- What attribute precedence levels are available for Roles
- Setting attribute precedence

## BASE ROLE & NESTED ROLES
- What are nested roles?
- Whats the purpose of a ‘base’ role?

## USING KNIFE
- The `knife role` command
- How would you set the run_list for a node using `knife`?
- Listing nodes
- View role details
- Using Roles within a search

# UPLOADING COOKBOOKS TO CHEF SERVER

## USING BERKSHELF
- The Berksfile & Berksfile.lock
- `berks install` and `berks upload`
- Where are dependant cookbooks stored locally?
- Limitations of using knife to upload cookbooks
- Listing cookbooks on Chef Server using knife
- What happens if you try to upload the same version of a cookbook?
- How can you upload the same version of a cookbook?
- Downloading cookbooks from Chef Server
- Bulk uploading cookbooks using knife

# USING KNIFE

## BASIC KNIFE USAGE
- How does knife know what Chef Server to interact with?
- How does knife authenticate with Chef Server
- How/When would you use `knife ssh` & `knife winrm`?
- Verifying ssl certificate authenticity using knife
- Where can/should you run the `knife` command from?

## KNIFE CONFIGURATION
- How/where do you configure knife?
- Common options - cookbook_path, validation_key, chef_server_url, validation_key
- Setting the chef-client version to be installed during bootstrap
- Setting defaults for command line options in knife’s configuration file
- Using environment variables and sharing knife configuration file with your team
- Managing proxies

## KNIFE PLUGINS
- Common knife pluginsWhat
- What is 'knife ec2' plugin
- What is 'knife windows' plugin
- What is ‘knife block’ plugin?
- What is ‘knife spork’ plugin?
- Installing knife plugins

## TROUBLESHOOTING
- Troubleshooting Authentication
- Using `knife ssl check` command
- Using `knife ssl fetch` command
- Using '-VV' flag
- Setting log levels and log locations

# BOOTSTRAPPING

## USING KNIFE
- Common ‘knife bootstrap’ options - UserName, Password, RunList, and Environment
- Using `winrm` & `ssh`
- Using knife plugins for bootstrap - `knife ec2 ..`, `knife bootstrap windows ...`

## BOOTSTRAP OPTIONS
- Validator' vs 'Validatorless' Bootstraps
- Bootstrapping in FIPS mode
- What are Custom Templates

## UNATTENDED INSTALLS
- Configuring Unattended Installs
- What conditions must exists for unattended install to take place?

## FIRST CHEF-CLIENT RUN
- How does authentication work during the first chef-client run?
- What is ‘ORGANIZATION-validator.pem’ file and when is it used?
- What is the ‘first-boot.json’ file?

# POLICY FILES

## BASIC KNOWLEDGE AND USAGE
- What are policy files, and what problems do they solve?
- Policy file use cases?
- What can/not be configured in a policy file?
- Policy files and Chef Workflow

# SEARCH

## BASIC SEARCH USAGE
- What information is indexed and available for search?
- Search operators and query syntax
- Wildcards, range and fuzzy matching

## SEARCH USING KNIFE AND IN A RECIPE
- Knife command line search syntax
- Recipe search syntax

## FILTERING RESULTS
- How do you filter on Chef Server
- Selecting attributes to be returned

# CHEF SOLO

## WHAT CHEF SOLO IS
- Advantages & disadvantages of Chef-solo vs Chef Server
- Chef-solo executable and options
- Cookbooks, nodes and attributes
- Using Data Bags, Roles & Environments
- Chef-solo run intervals
- Retreiving cookbooks from remote locations
- Chef-solo and node object

# DATA BAGS

## WHAT IS A DATA_BAG
- When might you use a data_bag?
- Indexing data_bags
- What is a data_bag?
- Data_bag and Chef-solo

## DATA_BAG ENCRYPTION
- How do you encrypt a data_bag
- What is Chef Vault

## USING DATA_BAGS
- How do you create a data_bag?
- How can you edit a data_bag
