The Metronome Multi-Tenancy-Infrastructure
==========================================

Because private instances, next to the current public instance, of metronome
were needed it was decided to create a setup that makes creating these instances
and scaling the underlying infrastructure (in the future) simple.

As Metronome is meant to be easy to use and fire-and-forget to configure, the
access to the underlying infrastructure has to be a single point of entry. This
will in the future have a failover etc.

There are 3 elements within the Metronome system:
 - The carbon-protocol listener
 - The HTTP API for querying and adding data
 - The static HTML that serves as the frontend

For each of these elements a different way of differentiating between customers
is needed.

Identifying the carbon data per customer
----------------------------------------
The data in the carbon protocol is unsigned and unauthenticated. The only way
within the protocol to distinguish the source of the data is the name used for
the sent metrics (like `ns1.example.com.pdns.auth.rd-queries`). As this string
is freely choosable by the user setting up the nameserver, this is a horrible
way to identify the customer.

As most PowerDNS customers are telcos or network operators, it was chosen to
use a whitelist of IP addresses and/or netblocks to differentiate between
customer data (and which customer this belongs to) and public data.

Identifying and serving the HTTP interface to the customer
----------------------------------------------------------
For all customer webfrontends, HTTP Basic authentication will be used. Based on
the credentials, the usergroup (the customer) can be deduced. This is then used
to give access to both the api and the web interface with the correct url in the
`local.js` config file

How this is set up
------------------
Initially, everything is hosted on one machine.

```
CARBON DATA    --->   HAProxy:9300 ---> Metronome-127.0.0.1:9300+CUSTOMER_ID


HTTP API REQUEST ---> HAProxy:443  ---> Metronome-127.0.0.1:8000+CUSTOMER_ID
                       ^        \
                      /          \
HTTP NON-API REQUEST -            ---> Nginx-127.0.0.1:18000+CUSTOMER_ID
```

It is clear from the picture that at any point, requests can be sent to other
backend servers, should this be needed for performance or failover reasons.

Tools and technologies
----------------------

This infra will be run on Debian 8.0 "Jessie", which is currently nearing stable
status for 2 important reasons: it has HAProxy 1.5, which supports SSL natively,
and it has systemd. The latter can be used to easily spawn multiple instances
of metronome from a single service file.

To deploy and maintain this infra, Ansible will be used.

ToDo
----
Somewhat in order of execution

* Metronome
  * Package it using fpm (for now)
    * Versioning will be 0.1+$(date +%Y%m%d)-NUM
  * Write systemd service file
* Ansible
  * Evaluate available HAProxy and nginx roles for possible use
  * Write Metronome role
  * Write template for `local.js`
