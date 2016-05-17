# Linux Server Configuration

### Overview

This project takes a baseline installation of a Linux distribution on a virtual machine and prepares it to host a web application by installing updates, securing it from a number of attack vectors, and installing/configuring web and database servers.

### HTTP

The web application, [Treasure Trove](https://github.com/D-O-M/P3-Catalog-Web-App-With-OAuth2/tree/master/catalog), is hosted via HTTP at both: 

- [http://ec2-52-33-104-21.us-west-2.compute.amazonaws.com/](http://ec2-52-33-104-21.us-west-2.compute.amazonaws.com/) and

- [52.33.104.21](http://52.33.104.21)

### SSH

To gain entry into the server, you can SSH into the server by executing the command:

```
ssh -i path/to/RSA_key_file -p 2200 grader@52.33.104.21
```

Ensure the RSA file is in the right place and enter the password provided.