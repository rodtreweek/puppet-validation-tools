# puppet-validation-tools
Various bits and pieces for troubleshooting Puppet issues.

The following is a general summary of code validation self service tools/scripts. It is by no means intended as an exhaustive list.  Feel free to add your own suggestions for inclusion to this list in the comments below <ac:emoticon ac:name="smile"/>

Note: Remember to use the `--noop` flag if you don't want your changes immediately applied.
  
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
Here is a link to the Type reference: <a href="https://docs.puppet.com/puppet/latest/type.html">https://docs.puppet.com/puppet/latest/type.html</a> . Certainly a great way to learn about the various Types in Puppet, and how best to leverage them.</p>
    </li>
    <li>
      <p>
        <code>hiera_explain</code>. Use of this tool is going to vary based on how you have your code and data structured in your Puppet implementation. Here's a link that provides a good starting place for experimenting/learning about how to interact with this module: <a href="https://github.com/binford2k/hiera_explain">https://github.com/binford2k/hiera_explain</a>
      </p>
    </li>
    <li>
      <p>
        <code>puppet-syntax</code>. A module that is worth considering for any CI/CD workflow. More info available here: <a href="https://github.com/voxpupuli/puppet-syntax">https://github.com/voxpupuli/puppet-syntax</a>. </p>
    </li>
    <li>
      <p>
        <code>puppet parser validate</code> . Although many may be familiar with it, it bears inclusion in this list --as well as defining as a function in .bashrc ;)</p>
    </li>
    <li>
      <p>
        <code>puppet-lint</code>. Another tried and true method for keeping things orderly :)</p>
    </li>
    <li>
      <p>The Puppet Enterprise Client Tools Suite. <a href="https://docs.puppet.com/pe/latest/install_pe_client_tools.html">https://docs.puppet.com/pe/latest/install_pe_client_tools.html</a> --I <span>highly</span> recommend getting familiar with this suite of tools, as it makes much easier work of interacting with various PE services, previously relegated to curl commands ran against the api endpoints (usually as root), and/or logging in directly to the console.</p>
    </li>
    <li>
      <p>Increasing the default logging level for your Puppet agents. While not specifically a tool for code validation, increasing the debug log level can be incredibly useful for troubleshooting issues where agents are inexplicably hanging. The default logging level is typically too low to determine why an agent run may be intermittently hanging. To turn this on, edit the <code>/etc/sysconfig/puppet</code> file and uncomment the <code>PUPPET_EXTRA_OPTS</code> line, changing it to:</p>
      <ac:structured-macro ac:macro-id="6435147a-6d3f-407b-8809-3e9d72c64a30" ac:name="code" ac:schema-version="1">
        <ac:parameter ac:name="language">bash</ac:parameter>
        <ac:plain-text-body><![CDATA[PUPPET_EXTRA_OPTS="--debug --trace --logdest=/var/log/puppetlabs/puppet/puppet-debug.log" ]]></ac:plain-text-body>
      </ac:structured-macro>
      <p>Then create the log file with:</p>
      <ac:structured-macro ac:macro-id="5a2f8507-cb6d-4c06-aa06-76e081c340b4" ac:name="code" ac:schema-version="1">
        <ac:parameter ac:name="language">bash</ac:parameter>
        <ac:plain-text-body><![CDATA[$touch /var/log/puppetlabs/puppet/puppet-debug.log]]></ac:plain-text-body>
      </ac:structured-macro>
      <p>Restart Puppet with <code>service puppet restart</code>. When the problem resurfaces, look at the <code>/var/log/puppetlabs/puppet/puppet-debug.log</code> file for clues. This is basically the equivalent of running this on the command line:</p>
      <ac:structured-macro ac:macro-id="f28ad77c-d2d0-449c-800e-95c1c60bf09b" ac:name="code" ac:schema-version="1">
        <ac:parameter ac:name="language">bash</ac:parameter>
        <ac:plain-text-body><![CDATA[puppet agent -t --debug --trace 2>&1 | tee trace.txt]]></ac:plain-text-body>
      </ac:structured-macro>
      <p>
        <br/>
      </p>
      <p>
        <br/>
      </p>
    </li>
    <li>
      <p>Beginning with Puppet 4.4, the <code>puppet lookup</code> command: <a href="https://docs.puppet.com/puppet/4.4/lookup_quick.html">https://docs.puppet.com/puppet/4.4/lookup_quick.html</a> (this is now simply referred to as "Hiera 5", and is documented here: <a href="https://docs.puppet.com/puppet/4.9/hiera_intro.html">https://docs.puppet.com/puppet/4.9/hiera_intro.html</a> and here: <a href="https://support.puppet.com/hc/en-us/articles/115005435168">https://support.puppet.com/hc/en-us/articles/115005435168</a>
      </p>
    </li>
    <li>
      <p>
        <code>r10k puppetfile install</code>: This will look for a Puppetfile in the current working directory, and if found, will install all modules as listed in the file, pulling in all relevant dependencies. This is an indispensable tool when you need to test code, and keep getting dependency related errors when using <code>puppet apply.</code>
      </p>
    </li>
    <li>
      <p>
        <code>puppet module install</code>: This will install a module and bring in all dependencies. While the <code>r10k puppetfile install</code> is fine if you are testing on a PE Master, you might want/need to test some code on a Linux client system where you have your control repo, but aren't managing your module code directly. To avoid dependency issue when using <code>puppet apply.</code> you can use <code>puppet module install</code> with the <code>--modulepath</code> flag to install these modules, automatically bringing in all dependencies.</p>
    </li>
    <li>
      <p>While you really should be using <a href="https://github.com/dylanratcliffe/onceover">https://github.com/dylanratcliffe/onceover</a>, here is a (pretty hacky) for loop to quickly validate .eyaml/.yaml (haven't tested this fully with .eyaml, but it should at least catch those pesky "hard tabs"):</p>
      <ac:structured-macro ac:macro-id="7dfbfcc3-ff67-4f70-915f-f08ea232bf06" ac:name="code" ac:schema-version="1">
        <ac:parameter ac:name="language">bash</ac:parameter>
        <ac:plain-text-body><![CDATA[for x in `find . -regex ".*[eyaml][yaml]?" -print`; do ruby -e "require 'yaml'; YAML.load_file('$x')"; echo $x; done 2>&1 |tee /dev/tty >> outfile.txt]]></ac:plain-text-body>
      </ac:structured-macro>
      <p>
        <br/>
        <br/>
      </p>
    </li>
    <li>
      <p>Script to retrieve all the parameters for the Puppet_enterprise class from the most recently compiled catalog on the agent:</p>
      <ac:structured-macro ac:macro-id="dff44e18-a2ac-41f1-b380-b72bea1b2549" ac:name="code" ac:schema-version="1">
        <ac:parameter ac:name="language">bash</ac:parameter>
        <ac:plain-text-body><![CDATA[for x in `find $(puppet agent --configprint client_datadir) -name "*.json" -print`; do echo $x; jq '.resources[] | select(.type == "Class" and .title == "Puppet_enterprise").parameters' $x >> ~/pe_class_params.txt; done]]></ac:plain-text-body>
      </ac:structured-macro>
      <p>Here's a command to get all the classes that were applied on the last run:</p>
      <ac:structured-macro ac:macro-id="b28a94b1-bc03-4d80-a537-136688a8e68c" ac:name="code" ac:schema-version="1">
        <ac:parameter ac:name="language">bash</ac:parameter>
        <ac:plain-text-body><![CDATA[[root@rtreweek ~]# cat $(puppet agent --configprint classfile)
pe_repo
pe_repo::platform::el_7_x86_64
puppet_enterprise
puppet_enterprise::profile::agent
puppet_enterprise::profile::amq::broker
puppet_enterprise::profile::certificate_authority
puppet_enterprise::profile::console
puppet_enterprise::profile::master
puppet_enterprise::profile::master::mcollective
[...]]]></ac:plain-text-body>
      </ac:structured-macro>
      <p>
        <span>And here's how to validate that a class is being enforced:</span>
      </p>
      <ac:structured-macro ac:macro-id="b7098965-b621-403e-9178-da1c71e08558" ac:name="code" ac:schema-version="1">
        <ac:parameter ac:name="language">bash</ac:parameter>
        <ac:plain-text-body><![CDATA[[root@rtreweek ~]# grep docker $(puppet agent --configprint classfile)
docker::params
docker::systemd_reload
docker
docker::repos
docker::install
docker::config
docker::service ]]></ac:plain-text-body>
      </ac:structured-macro>
      <p>
        <span>Lists all resource titles enforced on last run:</span>
      </p>
      <ac:structured-macro ac:macro-id="ef3b59ed-d6e5-41ac-8535-3d78ba9964cc" ac:name="code" ac:schema-version="1">
        <ac:parameter ac:name="language">bash</ac:parameter>
        <ac:plain-text-body><![CDATA[[root@rtreweek state]# cat $(puppet agent --configprint resourcefile)
service[postgresql]
file[/etc/puppetlabs/puppet/routes.yaml]
ini_setting[puppetdbserver]
service[pe-httpd]
firewall[5432 accept - postgres]
puppetdb_conn_validator[puppetdb_conn]
package[postgresql-server]
exec[/sbin/iptables-save > /etc/sysconfig/iptables]
[...] ]]></ac:plain-text-body>
      </ac:structured-macro>
      <div class="page" title="Page 61">
        <div class="layoutArea">
          <div class="column">
            <p>Identify which yum repo's are being managed:</p>
            <ac:structured-macro ac:macro-id="690c41d9-60d3-4a20-8342-dd2966f33820" ac:name="code" ac:schema-version="1">
              <ac:parameter ac:name="language">bash</ac:parameter>
              <ac:plain-text-body><![CDATA[[root@rtreweek state]# grep yumrepo $(puppet agent --configprint resourcefile)
yumrepo[docker]
yumrepo[epel-testing]
yumrepo[epel-testing-debuginfo]
yumrepo[epel-testing-source]
yumrepo[epel]
yumrepo[epel-debuginfo]
yumrepo[epel-source]]]></ac:plain-text-body>
            </ac:structured-macro>
          </div>
        </div>
      </div>
    </li>
    <li>
      <p>If you are looking to implement something a bit more sophisticated than the "one-off" command-line validation suggestions above, you might consider implementing <strong>rspec testing of your Puppet modules</strong>, as discussed in the excellent blog post by Joseph Oaks available here:<span style="color: rgb(34,34,34);"> </span> <a href="https://puppet.com/blog/unit-testing-rspec-puppet-for-beginners">https://puppet.com/blog/unit-testing-rspec-puppet-for-beginners</a> and another post by Nick Walker here: <a href="https://puppet.com/blog/use-onceover-start-testing-rspec-puppet">https://puppet.com/blog/use-onceover-start-testing-rspec-puppet</a>
      </p>
    </li>
    <li>
      <p>The <span style="color: rgb(34,34,34);">
          <code>validate_cmd</code> and <code>validate_cmd</code> Resource Types: <a href="https://docs.puppet.com/puppet/latest/type.html#file-attribute-validate_cmd">https://docs.puppet.com/puppet/latest/type.html#file-attribute-validate_cmd</a> .These are great resource types to use in<span> validating a file’s syntax before replacing it.</span> </span>
      </p>
    </li>
  </ol>
  <br/>
  <p>
    <br/>
  </p>
  <p>
    <span>The above list should offer a reasonable collection of tools for detecting errors/validating code as well as possibly clarifying the points in the "decision tree" within Hiera.  For a more thorough explanation of how to best leverage each of the above suggestions, you should consult the relevant pages on </span> <a href="http://docs.puppet.com/">docs.puppet.com</a> <span>, or try searching support.puppet.com for more specific information.</span>
  </p>
</div>


