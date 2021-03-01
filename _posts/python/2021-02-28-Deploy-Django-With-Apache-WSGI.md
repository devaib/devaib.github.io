---
layout: post
title: Deploy Django with Apache and mod_wsgi
description: Deploy Django with Apache and mod_wsgi.
summary: Deploy Django with Apache and mod_wsgi.
tags: [django, apache, mod_wsgi, deploy, python]
permalink: "/python/DeployDjangoWithApacheWSGI"
---

### Reference

- [How to use Django with Apache and mod_wsgi - django documentation](https://docs.djangoproject.com/en/3.1/howto/deployment/wsgi/modwsgi/)
- [Quick Installation Guide - mod_wsgi](https://modwsgi.readthedocs.io/en/develop/user-guides/quick-installation-guide.html)
- [Python Django Tutorial: Deploying Your Application (Option #1) - Deploy to a Linux Server](https://www.youtube.com/watch?v=Sa_kQheCnds&ab_channel=CoreySchafer)

### Main Steps

1. Ip Address and SSH Credentials
2. Root Connection to the Server
3. Setting Host Name
    - `hostnamectl set-hostname django-server`
4. Setting Host File
    - /etc/hosts -> "<ip> django-server"
5. Adding Limited User
6. Setting Up SSH Key Based Authentication
7. Forbiding Root Login & Password Authentication
8. Setting Up a Firewall
    - ufw
9. Putting Django Application on Webserver
10. Generating requirements.txt File
11. Copying Django Application on to the Webserver
12. Creating Virtual Environment on the Server
13. Installing Dependencies
14. Changing Django Settings for Testing the Application on Django Server
15. Collecting Static Files 
    - `python manage.py collectstatic`
16. Testing Application
17. Installing Apache & ModWSGI 
    - `sudo apt install apache2`
    - build mod_wsgi source code(the libapache2-mod-wsgi-py3 lib is outdated)
18. Configuring Apache Webserver
    - https://github.com/CoreyMSchafer/code_snippets/blob/master/Django_Blog/snippets/django_project.conf
19. Enabling Site Through Apache
    - `sudo a2ensite django_project`
    - `sudo a2dissite 000-default.conf`
20. Setting Up File Permissions
    - project directory, db files
21. Creating Configuration File for Hiding Sensitive Information
22. Updating Project Settings File
23. Allowing http Traffic
    - `sudo ufw allow http/tcp`
24. Restarting the Server & Running the Site
    - `sudo service apache2 restart`

### Additional Apache Configuration

```
# /etc/apache2/apache2.conf

LoadFile /<path>/<to>/miniconda3/envs/django/lib/libpython3.8.so.1.0
LoadModule wsgi_module /usr/lib/apache2/modules/mod_wsgi.so

WSGIPythonHome /<path>/<to>/miniconda3/envs/django
WSGIPythonPath /<path>/<to>/<project>
```
 
