# Cockpit
**A sysadmin login session in a web browser**

[cockpit-project.org](https://cockpit-project.org/)

Cockpit is an interactive server admin interface. It is easy to use and very lightweight.
Cockpit interacts directly with the operating system from a real Linux session in a browser.


## TODO

I want to add my own pages of some features which works for router with this GUI, and the OS 
of router could be such as Ubuntu, based on embedded terminal.

  * 1. route system information
  * 2. WAN configuration, ethernet port, cellular, WiFi
  * 3. LAN configuration, ethernet port, WiFi
  * 4. Firewall configuration
  * 5. Port forwarding configuration
  * 6. keep alive configuraion
  * 7. router account management
  * 8. configuration management
  * 9. firmware upgrade / OTA
  * 10. maintenance configuraion
  * 11. others...


### Using Cockpit

You can [install Cockpit](https://cockpit-project.org/running.html) on many Linux operating
systems including Debian, Fedora and RHEL.

Cockpit makes Linux discoverable, allowing sysadmins to easily perform tasks such as starting
containers, storage administration, network configuration, inspecting logs and so on.

Jumping between the terminal and the web tool is no problem. A service started via Cockpit
can be stopped via the terminal. Likewise, if an error occurs in the terminal, it can be seen
in the Cockpit journal interface.

You can also easily add other machines that have Cockpit installed and are accessible via SSH and jump
between these hosts.

### Development

 * [Making changes to Cockpit](HACKING.md)
 * [How to contribute, developer documentation](https://github.com/cockpit-project/cockpit/wiki/Contributing)
 * IRC Channel: #cockpit on [libera.chat](https://libera.chat/)
 * [Mailing List](https://lists.fedorahosted.org/admin/lists/cockpit-devel.lists.fedorahosted.org/)
 * [Guiding Principles](https://cockpit-project.org/ideals.html)
 * [Release Notes](https://cockpit-project.org/blog/category/release.html)
 * [Privacy Policy](https://cockpit-project.org/privacy.html)
