# Active Directory Certificate Services (ADCS) Deployment Template.

The included config template installs the following features and tools:

<details>
    <summary>ADCS role services configuration in the config file</summary>

* [X] Certification Authority
* [ ] Certificate Enrollment Policy Web Service
* [ ] Certificate Enrollment Web Service
* [X] Certitication Authority Web Enrollment
* [ ] Network Device Enrollment Service
* [ ] Online Responder

</details>

<details>
    <summary>IIS role services configuration in the config file</summary>

* [X] Web Server
    * Common HTTP Features
        * [X] Default Document
        * [X] Directory Browsing
        * [X] HTTP Errors
        * [X] Static Content
        * [X] HTTP Redirection
        * [ ] WebDAV Publishing
    * Health and Diagnostics
        * [X] HTTP Logging
        * [ ] Custom Logging
        * [X] Logging Tools
        * [ ] ODBC Logging
        * [X] Request Monitor
        * [X] Tracing
    * Performance
        * [X] Static Content Compression
        * [ ] Dynamic Content Compression
    * Security
        * [X] Request Filtering
        * [ ] Basic Authentication
        * [ ] Centralized SSL Certificate Support
        * [ ] Client Certificate Mapping Authentication
        * [ ] Digest Authentication
        * [ ] IIS Client Certificate Mapping Authentication
        * [X] IP and Domain Restrictions
        * [X] URL Authentication
        * [X] Windows Authentication
    * Application Development
        * [ ] .NET Extensibility 3.5
        * [ ] .NET Extensibility 4.8
        * [ ] Application Initialization
        * [X] ASP
        * [ ] ASP.NET 3.5
        * [ ] ASP.NET 4.8
        * [ ] CGI
        * [X] ISAPI Extensions
        * [ ] ISAPI Filters
        * [ ] Server Side Includes
        * [ ] WebSocket Protocol
* [ ] FTP Server
* [X] Management Tools
    * [X] IIS Management Console
    * [X] IIS 6 Management Compatibility
        * [X] IIS 6 Metabase Compatibility
        * [ ] IIS 6 Management Console
        * [ ] IIS 6 Scripting Tools
        * [ ] IIS 6 WMI Compatibility
    * [X] IIS Management Scripts and Tools
    * [X] Management Service
</details>
