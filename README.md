# puppet-validation-tools

The following is a general summary of code validation "self-service" tools/scripts which can be used to troubleshoot and validate your Puppet code. It is by no means intended as an exhaustive list.  Feel free to contact me/submit a pull request for any inclusions you might like to see added to this list :)

**Note: Remember to use the `--noop` flag if you don't want your changes immediately applied.**
  
1. `puppet help` -- Not to be overlooked, this command is often the best starting place for troubleshooting a significant number of issues.
    
2. `puppet apply -e` -- For example, you might test a hiera lookup with, `sudo puppet apply --environment=target environment -e "notice(hiera('message'))"` . 
Also, just to give you an idea of how the interpolation works, here's an example using the exec resource:
      
     `puppet apply -e 'exec { "/usr/bin/cowsay $::fqdn > /etc/motd": }'`
      
      An example using an include:
      `puppet apply -e "include pe_repo::platform::aix_71_power"`
      
      How to use an `inline_epp` template to access elements in an array without a key:
     
```
puppet apply -e 'notice(inline_epp("<%= \$processors[models][0] %>"))'
```
Same thing, but with a ruby `inline_template` (and a few more options):
```
puppet apply -e 'notice(inline_template("<%= @processors[\"models\"][0] %>"))'
puppet apply -e 'notice(inline_template("<%= @processors[\"models\"].at(0) %>"))'
puppet apply -e 'notice(inline_template("<%= @processors[\"models\"].first %>"))'
puppet apply -e 'notice(inline_template("<%= @processors[\"models\"].last %>"))'
```
Here is a link to the Type reference: <a href="https://docs.puppet.com/puppet/latest/type.html">https://docs.puppet.com/puppet/latest/type.html</a> . Certainly a great way to learn about the various Types in Puppet, and how best to leverage them.
    
3. `hiera_explain` -- Use of this tool is going to vary based on how you have your code and data structured in your Puppet implementation. Here's a link that provides a good starting place for experimenting/learning about how to interact with this module: <a href="https://github.com/binford2k/hiera_explain">https://github.com/binford2k/hiera_explain</a>
      
4. `puppet-syntax` -- A module that is worth considering for any CI/CD workflow. More info available here: <a href="https://github.com/voxpupuli/puppet-syntax">https://github.com/voxpupuli/puppet-syntax</a>. 

5. `puppet parser validate` Although many may be already familiar with it, it bears inclusion in this list -- as well as perhaps defining as a specific function in `.zshrc/.bashrc` ;)

6. `puppet-lint` -- Another tried and true method for keeping things orderly :)

7. The Puppet Enterprise Client Tools Suite. <a href="https://docs.puppet.com/pe/latest/install_pe_client_tools.html">https://docs.puppet.com/pe/latest/install_pe_client_tools.html</a> -- I highly recommend getting familiar with this suite of tools, as it makes much easier work of interacting with various PE services, previously relegated to lengthy curl commands ran against the various API endpoints (usually as root), and/or logging in directly to the console.

8. Increasing the default logging level for your Puppet agents. -- While not specifically a "tool" for code validation, increasing the debug log level can be incredibly useful for troubleshooting issues where agents are inexplicably hanging, etc.. The default logging level is typically too low to determine why an agent run may be intermittently hanging. 

To turn this on, edit the `/etc/sysconfig/puppet` file and uncomment the `PUPPET_EXTRA_OPTS` line, changing it to: `PUPPET_EXTRA_OPTS="--debug --trace --logdest=/var/log/puppetlabs/puppet/puppet-debug.log"`

Then create the log file with: `$touch /var/log/puppetlabs/puppet/puppet-debug.log`. Restart Puppet with `service puppet restart`. When the problem re-surfaces, look at the `/var/log/puppetlabs/puppet/puppet-debug.log` file for clues. This is basically the equivalent of running the following from the command line:
```
puppet agent -t --debug --trace 2>&1 | tee trace.txt
```

8. Beginning with Puppet 4.4, the `puppet lookup` command may be used: https://docs.puppet.com/puppet/4.4/lookup_quick.html">https://docs.puppet.com/puppet/4.4/lookup_quick.html (this is now simply referred to as "Hiera 5", and is documented here: <a href="https://docs.puppet.com/puppet/4.9/hiera_intro.html">https://docs.puppet.com/puppet/4.9/hiera_intro.html</a> and here: <a href="https://support.puppet.com/hc/en-us/articles/115005435168">https://support.puppet.com/hc/en-us/articles/115005435168</a>

9. `r10k puppetfile install` -- This will look for a `Puppetfile` in the current working directory, and if found, will install all modules as listed in the file, pulling in all relevant dependencies. This is an indispensable tool when you need to test code, i.e. where you keep getting dependency related errors when using `puppet apply`.

10. `puppet module install` -- This will install a module and bring in all dependencies. While the `r10k puppetfile install` is fine if you are testing on a PE Master, you might want/need to test some code on a Linux client system where you have your control repo, but aren't managing your module code there directly -- again to avoid dependency issue when using `puppet apply`. You can use `puppet module install` with the `--modulepath` flag to install these modules, again automatically bringing in all dependencies.

11. While you really should be using <a href="https://github.com/dylanratcliffe/onceover">https://github.com/dylanratcliffe/onceover</a>, here is a somewhat primitive bash `for loop` to quickly validate `.eyaml/.yaml` (haven't tested this fully with .eyaml, but it should at least catch those pesky "hard tabs"):
```
for x in `find . -regex ".*[eyaml][yaml]?" -print`; do ruby -e "require 'yaml'; YAML.load_file('$x')"; echo $x; done 2>&1 |tee /dev/tty >> outfile.txt
```
Here's a script to retrieve all the parameters for the Puppet_enterprise class from the most recently compiled catalog on the agent
```
for x in `find $(puppet agent --configprint client_datadir) -name "*.json" -print`; do echo $x; jq '.resources[] | select(.type == "Class" and .title == "Puppet_enterprise").parameters' $x >> ~/pe_class_params.txt; done
```
Here's a command to get all the classes that were applied on the last run:
```
# cat $(puppet agent --configprint classfile)
pe_repo
pe_repo::platform::el_7_x86_64
puppet_enterprise
puppet_enterprise::profile::agent
puppet_enterprise::profile::amq::broker
puppet_enterprise::profile::certificate_authority
puppet_enterprise::profile::console
puppet_enterprise::profile::master
puppet_enterprise::profile::master::mcollective
[...]
```
And here's a `grep` command to grab something more specific:
```
# grep docker $(puppet agent --configprint classfile)
docker::params
docker::systemd_reload
docker
docker::repos
docker::install
docker::config
docker::service
```
This lists all resource titles enforced on the last run:
```
# cat $(puppet agent --configprint resourcefile)
service[postgresql]
file[/etc/puppetlabs/puppet/routes.yaml]
ini_setting[puppetdbserver]
service[pe-httpd]
firewall[5432 accept - postgres]
puppetdb_conn_validator[puppetdb_conn]
package[postgresql-server]
exec[/sbin/iptables-save > /etc/sysconfig/iptables]
[...]
```
Identify which yum repo's are being managed:
```
# grep yumrepo $(puppet agent --configprint resourcefile)
yumrepo[docker]
yumrepo[epel-testing]
yumrepo[epel-testing-debuginfo]
yumrepo[epel-testing-source]
yumrepo[epel]
yumrepo[epel-debuginfo]
yumrepo[epel-source]
```
12. If you are looking to implement something a bit more sophisticated than the "one-off" command-line validation suggestions above, you might consider implementing **rspec testing of your Puppet modules,** as discussed in the excellent blog post by Joseph Oaks available here:<span style="color: rgb(34,34,34);"> </span> <a href="https://puppet.com/blog/unit-testing-rspec-puppet-for-beginners">https://puppet.com/blog/unit-testing-rspec-puppet-for-beginners</a> and another post by Nick Walker here: <a href="https://puppet.com/blog/use-onceover-start-testing-rspec-puppet">https://puppet.com/blog/use-onceover-start-testing-rspec-puppet</a>

13. The `validate_cmd` Resource Types: https://puppet.com/docs/puppet/5.5/types/file.html#file-attribute-validate_cmd. These are great resource types to use in validating a file’s syntax before replacing it.

The above list should offer a reasonable collection of tools for detecting errors in advance or validating your Puppet code prior to pushing this to production, as well as possibly clarifying various "points in the decision tree" within Hiera.  For a more thorough explanation of how to best leverage each of the above suggestions, you should consult the relevant pages on <a href="http://docs.puppet.com/">docs.puppet.com</a>, or try searching support.puppet.com for more specific information.
