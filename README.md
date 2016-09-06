Guidelines for Web Application Programming
==========================================

William & Mary IT provides a web hosting platform for any group affiliated with the College,
including administrative groups, academic/research groups, and student organizations.
The purpose of this guide is to help write an application that will run securely on top of the
platform provided by IT.

Because they are limited in scope, but easier to develop on, developing on
[blogs.wm.edu](http://blogs.wm.edu),
[Cascade](http://www.wm.edu/offices/webanddesign/web/cascade/index.php), and
[Tribe Voices](http://www.wm.edu/offices/webanddesign/web/tribevoices/index.php) is not
covered here.

While this document specifies some conventions used by William & Mary, it can be
summarized to say that all code should be developed in a way that it could withstand
scrutiny from developers outside of W&M.

1. [Platform Overview](#platform-overview)
2. [Security Model](#security-model)
3. [Code Style](#code-style)
    - [PHP Style](#php-style)
    - [Python Style](#pyton-style)
    - [Ruby Style](#ruby-style)
    - [Node.js Style](#nodejs-style)
4. [Security Best Practices](#security-best-practices)
    - [PHP Platform Security](#php-platform-security)
5. [Shared Cluster Considerations](#shared-cluster-considerations)
    - [Reverse Proxy](#reverse-proxy)
    - [Shared Storage](#shared-storage)
    - [htaccss Files](#htaccess-files)
6. [Documenting your Application](#documenting-your-application)
7. [Authenticating Users](#authenticating-users)

## Platform Overview

IT's general use hosting platform consists of multiple servers behind a load balancer
that behaves like a reverse proxy. Application files are stored on a highly available
shared filesystem that is mounted on each server via NFS.

A MySQL compatible database is available by request. The database is replicated across
multiple servers. The connection endpoint may be a server or a load balancer.

Because servers are behind a load balancer, developers are reminded that HTTP is a
stateless protocol. Assumptions that work on a single server must not be made when
writing code for a load balanced environment.

## Security Model

IT's web hosting operates under a shared responsibility security model. IT will provide a
platform that's designed with security in mind. The application/site owner is responsible for
developing and maintaining the code to ensure the site is secure.

IT reserves the right to remove sites without notice that are compromised or threaten the
stability and security of the platform.

## Code Style

A style guide should be followed whenever possible to make it as easy as possible for
experienced developers to make sense of code that is new to them. Style refers to the layout
of code, including variable, class, method naming conventions, and sometimes how some
operations are handled.

If a specific framework is used, and specifies how theming, database queries, or code
layout should be used, then it should be followed. For example, writing database queries by hand should
be avoided in [Laravel](https://laravel.com/docs/5.2/queries) (or any framework that
supplies ORM) whenever possible, and database queries shouldn't show up in the view
on any MVC.

### PHP Style

Unless a framework used has its own style guide (e.g.
[WordPress](https://make.wordpress.org/core/handbook/best-practices/coding-standards/php/)), standards
set by the [PHP Framework Interoperability Group](http://www.php-fig.org/) should be used. Specifically
[PSR-1](http://www.php-fig.org/psr/psr-1/) and [PSR-2](http://www.php-fig.org/psr/psr-1/).

### Python Style

Unless a framework being used has its own style guide, PEP8 should be followed. Rules
that are purposely excluded should be noted somewhere obvious.

### Ruby Style

Unless a framework being used has its own style guide, the community maintained
[Ruby style guide](https://github.com/bbatsov/ruby-style-guide) should be used.

### Node.js Style

Unless a framework being used has its own style guide, the community maintained
[None.js Style Guide](https://github.com/felixge/node-style-guide) should be used.


## Security Best Practices

Established best practices must be followed for preventing cross-site scripting attacks (XSS),
SQL/Shell Injections.

**** PROVIDE LINKS TO BEST PRACTICES HERE ****

### PHP Platform Security

#### Disabled Functions

A number of functions are disabled on our PHP servers to protect against accidental information
disclosure and avoid common security issues. They're grouped by family below.

* pcntl_*
* exec
* shell_exec
* phpinfo
* eval
* system
* passthru
* apache_child_terminate
* apache_setenv
* symlink
* chgrp
* chown
* posix_uname
* proc_close, proc_nice, proc_terminate, proc_open

#### Injection Protection

User input must be properly sanitized before passing it to a database or shell. That
a form or handler is behind authentication does not mean that unsanitized input may be used
(See [here](https://xkcd.com/327/)).

If a framework has its own method for escaping input then it should be used before a
custom solution is implemented.

## Shared Cluster Considerations

Because there are multiple servers behind a load balancer, some common single-server
assumptions cannot be made in the W&M environment. The load balancer acts as a reverse
proxy, SSL is not used behind the load balancer, and files are mounted off of a highly available NAS.

### Reverse Proxy

Every HTTP(s) request connects to a load balancer that then makes another request to a
healthy server in the cluster. A developer must not assume that a series of requests
will talk to the same server in the cluster.

The load balancer will set X-Forwarded-For, X-Forwarded-Proto, and X-Forwarded-Port headers
that indicate the IP address, protocol, and port used to connect to the load balancer.

To help keep compatibility with PHP applications that check `$_SERVER['https']`, the
environment variable `HTTPS` will be set to `on` if X-Forwarded-Proto is set to https.

### Shared Storage

One directory level up from the document root is shared across all servers and is available
for libraries and other persistent storage. The system temporary
directory is not, but a different temporary location is available when needed. On PHP systems
the directory returned by `sys_get_temp_dir()` is shared across all servers. For non-PHP
applications, contact IT to find out what to use.

### htaccess Files

As is recommended by the
[Apache documentation](https://httpd.apache.org/docs/current/howto/htaccess.html#when),
.htaccess files are ignored. IT can add any required configuration to the Apache
configuration.

#### Additional Libraries

Extra libraries should be installed within the document root or one level up.

On Python systems [virtualenv](https://virtualenv.pypa.io/en/stable/) should be used.

## Documenting your Application

Projects should be documented in a way that someone intermediately familiar with the
framework and language is able to fix bugs, make functional changes, and take ownership
if needed.

Projects should support a widely used and language-relevant (i.e. PHPDoc shouldn't be used with Ruby)
documentation tool such as [PHPDoc](https://www.phpdoc.org/),
[Sphinx](http://www.sphinx-doc.org/en/stable/), or [RDoc](https://github.com/rdoc/rdoc) whenever
the project is sufficiently complex.

## Authenticating Users

Single sign on (SSO) should be used to authenticate users. Central Authentication
Service (CAS) and Shibboleth/SAML are both available as identity providers. CAS is the
preferred method, and typically is the easiest to setup. SAML may not be made available
in all cases.

### CAS Libraries

There are a number of CAS libraries available for each supported language. There are add-ons
to many popular frameworks that add CAS support to the built in authentication/authorization
mechanisms.

A web search will help you identify which one is right for your use case.
