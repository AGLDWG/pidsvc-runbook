# Run Book / System Operation Manual
This repository contains information about how to operate the Ausralian Government Linked Data Working Group' Persistent ID Service (PID Svc) managing subdomains at *.data.gov.au, for example environment.data.gov.au.


## Role
The PID Svc is a proxy server with a graphical user interface enabling people to edit proxy rules (redirects and proxying actions) without having to code the system. It is used in GA to manade the pid.geoscience.gov.au persisternt domain. That domain is used to identify datasets, vocabularies, terms within vocabularies, services and other information resources.


## Governance
**Custodian**: The Australian Government Linked Data Working Group (<http://linked.data.gov.au>). Solutions subgroup convenor, Nicholas Car <nicholas.car@ga.gov.au>  
**Sponsor**: Chair of the Australian Government Linked Data Working Group, Dr Armin Haller <armin.haller@anu.edu.au>
**Point of Contact**: for technical matters, Nicholas Car   
**Creator**: SISS Stack team at CSIRO Perth, particularly Pavel Golodoniuc (<pavel.golodoniuc@csiro.au>)


## Operational Risks
The major operational risks and mitigation strategies are:

Risk | Description | Indicator | Likelyhood | Severity | Mitigation | Remediation | Worst case
---- | ----------- | --------- | ---------- | -------- | ---------- | ----------- | ----------
Misconfiguration | Upon reconfiguration due to an infrastructure change, the server is misconfigured | Patterns stop resolving | Low | Moderate | The server is established behind an Elastic IP, which can be re pointed to another server. Install scripts for this specific PID Svc are ready to go: contained within the pidsvc-installation repository. Backups of the data in this PID Svc are ready to be used for reload: within the pidsvc-backups repository | Directly fix server via console or reinstall a new server | 1/2 a day down time after discovery
Overload | The server bocomes overloaded due to many requests (it is not in an auto-scaling group) | degraded performance | Low | Moderate | Server is stand-alone and can be managed directly | The server can be stopped and re-started to break any long-runnign sessions and have resources increased during restart | Hours down time after discovery  
Malicious attack (overload) | as per Overload | as per Overload | Very Low | High | as per Overload | as per Overload | as per Overload 
Overiding pattern | A user accidentally overwrites existing patterns with new ones | some/all patterns resolve incorrectly | Moderate | Very Low | Limit pattern makers, instill pattern testing before implementation | Fix patterns in GUI | Minutes of down time after discovery (depending on pattern author availability) with new patterns requiring re-implementation
Lack of technical managers | No staff in GA can manage the PID Svc | inability to add/change patterns | Very Low | Low | train additional GA staff in management | Ask Web Ops or Cloud Enablement for technical assistance, call PID Svc creators (CSIRO) for help | Inability to make changes until staff trained


## System components

![AGLDWG PID Service runbook component diagram]
(AGLDWG-PID-Service-runbook-component-diagram.png)  

Figure 1: GA PID Service runbook component diagram  

The PID Svc system consists of the following major components:

* A **DNS entry** for *x.data.gov.au*
    * pointing via an A Record to a floating IP address
* A **floating IP address**
    * in the form of an AWS Elastic IP product
    * address: 203.100.30.55
    * pointing, via an AWS association, to a server
* A **server**
    * a Nectar Cloud (<http://www.nectar.org.au/>) Ubuntu Linux 16.04 instance, hosted by QCIF (<https://www.qcif.edu.au>)


The server consists of the following internal components:  

* **Apache Web Server Program**
    * version 2.4
    * all custom configuration in the VHosts files in /etc/apache2/sites-enabled/
        * copies of these files are backed up to the backup repo
* **Tomcat Web Server Program**
    * version 7
    * the PID Svc application itself is a WAR file application (a Tomcat site), multiple instances of which are run, one for each AGIFT-based subdomain
    * connection details for the PID Svc applications to the Postgres DBs are contained in %TOMCAT_HOME%/Catalina/localhost/
* **PID Svc WAR** 
    * this is the PID Svc application created by CSIRO, online at <https://cgsrv1.arrc.csiro.au/swrepo/PidService/jenkins/trunk/pidsvc-latest.war>
    * hosted by the Tomcat Server program
* **Postgres Database Server program**
    * version 9.3 (default for the Linux server version)
* **Postgres DB**
    * databases hosted by the Postgres Database Server program
    * constructed using the CSIRO build script <https://www.seegrid.csiro.au/subversion/PID/trunk/pidsvc/src/main/db/postgresql.sql>
    

## Systems access
The following systems are required for PID Svc operation:

* **DNS**
    * the domain name x.data.gov.au is governed by the Department of Finance
    

* **NECTAR account**
    * Where: the majority of the PID Svc systems sits within the 'AGLDWG' NECTAR account
    * Who: the account is managed by the AGLDWG. Access accounts held by Nicholas Car & Armin Haller


* **PID Service instance** -- at <http://pid.data.gov.au>
    * Where: hosted on a single NECTAR Virtual Machine, machine name 'pid-prod7', Elastic IP address: 52.63.174.100
	* Who: the machine can be accessed by the Catalogue team or Cloud Enablement
	* How: access details:
	    * PID Svc GUI
		    * UI webpage: <http://pid.data.gov.au/pidsvc>
		    * user: pidsvcadmin
		    * pwd: (see custodian)
        * pid-prod VM via SSH
            * user: ubuntu
            * pwd: access key 'nectar.ppk' (see custodian)


## Installation / Documentation
The installation of this PID Service follows the CSIRO documentation with changes to allow multiple instances of the PID Svc to be installed on a single Virtual Machine. This is a common Tomcat multiple container deployment. 

* CSIRO PID Svc installation documentation: <https://www.seegrid.csiro.au/wiki/Siss/PIDService#Installation>

The current set of commands (Linux shell) used to install the server are captured in the file /home/ubuntu/install.log which is also in this repository as install.log