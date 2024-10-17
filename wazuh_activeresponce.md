# **Wazuh Active response Setup**

### **Introduction**

Wazuh has an active response module that helps security teams automate response actions based on specific triggers, enabling them to effectively manage security incidents. Automating response actions ensures that high-priority incidents are addressed and remediated in a timely and consistent manner.

### **Steps**

The active response allows OSSEC to run commands on an agent in response to certain triggers. we need to edit **/var/ossec/etc/ossec.conf** in the Wazuh manager.

Then we have to add the following changes.

**1\. command**

```
<command>
    <name>firewall-drop</name>
    <executable>firewall-drop.sh</executable>
    <expect>srcip</expect>
    <timeout_allowed>yes</timeout_allowed>
</command>
```
In the command section,we use the command which we want to execute. In our case we use **firewall-drop**, the function of this command is to execute when the active responses are triggered in the agent. This command will block IP In the **local firewall**end. in the section we must specify the source IP in except and also we can specify a particular time out.

**2\. Active response.**

```
<active-response>
    <command>firewall-drop</command>
    <location>defined_agent</location>
    <agent_id>002</agent_id>
    <rules_id>31151</rules_id>
    <timeout>3600</timeout>
</active-response>
```

In the active response, the field specifies the required command(**firewall-drop**) and location use **Defined agent**. Specifying the **agent ID** also specifies the rule we want to use.Here I have used **31151** which is multiple failed attempts from the same IP. Time out is defined as time to block the malicious IP.3600s=1hr.

The default rules list can be seen from **/var/ossec/ruleset/rules** we have used **31151** rule from **0245-nginx_rules.xml**.There are many predefined rules can be seen from there.

**3\. White list**

We can also set a list of IP addresses that should never be blocked by the active response. In global section of ossec.conf in the Manager,use the field white_list.It allows IP address or netblock.

```
<ossec_config>
  <global>
    <jsonout_output>yes</jsonout_output>
    <email_notification>no</email_notification>
    <logall>yes</logall>
    <white_list>18.214.194.50</white_list>
  </global>
```

**4\. Adding mail alert for ip blocking.**

**601** rule id (**0015-ossec_rules.xml**) is the triggered rule when an ip blocked by active response,by default it’s alert level is **3**,change it to greater than **8**,here we have used this alert level as **9**.Otherwise mail alert will **not trigger**.

Then add the following in ossec.conf

```
<email_alerts>
  <email_to>email@email.com</email_to>
  <rule_id>601,602</rule_id>
  <do_not_delay />
</email_alerts>
```

After completing the configuration, run the following commands.
```
xmllint ossec.conf
```
Purpose of this command is to verify that configuration files have no errors.

If the command is not found, install the following package.
```
apt install libxml2-utils
systemctl restart wazuh-manager
```
From the following log file we can see the active response which is triggered by the agent.
```
tail -f /var/ossec/logs/active-responses.log
```
The file active-responses.log does not exist by default when OSSEC is installed,so it will not read it. You must create the file active-responses.log at /var/ossec/logs/ folder in agent. Then, restart the Manager and agents to apply the changes.

The following example shows the active response log in agent

```
root@server1:~# tail -f /var/ossec/logs/active-responses.log 
Sun May  7 12:45:26 UTC 2024 /var/ossec/active-response/bin/firewall-drop.sh add - 162.216.47.32 1620563486.6306696 31151
Sun May  7 12:55:27 UTC 2024 /var/ossec/active-response/bin/firewall-drop.sh delete - 162.216.47.32 1620563486.6306696 31151***
```

In case you want to unblock the blocked ip run the following command.
```
iptables -D INPUT -s IP -j DROP
service iptables save
```
**⚠ WARNING: Never copy-paste.** 


Never copy paste anything in to the ossec.conf, because it is xml file and xml is highly sensitive.If you copy paste it may cause syntax error.So always type what you want to add in to it.Agent Every agent has a log file where the active response activities are registered.

### **Outcome**[](https://stage.dinoct.com/operations/dinoct_wiki/runbook/wazuh_active_response_setup/#outcome)

Wazuh successfully blocked ip within seconds and alerts were sent to the email.
