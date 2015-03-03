# CI

# Build targets
* Both i586 and x64
* Linux
  * semistatic
  * Debian Stable (might need backported gcc)
  * Fedora latest
  * CentOS latest
* FreeBSD
  * 9
  * 10

# Technologies
  * Ansible
  * BuildBot
  * Docker
  * Jails

# Goals
  * fast builds (under 10 minutes end to end preferably)
  * flexibility (easily add another variant, like musl)

# Bonus goals
  * safe pull request building, including resulting packages

# Caveats
  * recursor testing wants a subnet on lo, this is tricky in docker (needs cap_net_admin)
