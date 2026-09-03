
[Infra](../Infra.md)

# Servers

# Servers Summary

| **server\_nickname** | **use\_case** | **location** | **public\_IP** | **needs\_VPN** | **connection\_methods** | **known\_users** |
| --- | --- | --- | --- | --- | --- | --- |
| cqt-paul-server | access to nqch quantum computer | NUS local machine | 10.246.80.228 | yes | public key, password | eortiz, nqch-deploy, sergi |
| cqt-ale-server | All kinds of compute | AWS ap-southeast-1 | 54.251.1.211 | no | public key | benchmarking, elis |
| cqt-ale-server | flask server hosting Tanviruls postgresql DB | AWS ap-southeast-1 | 54.169.91.191 | no | public key | alecqt, ubuntu |

# Instances in AWS CQT account

| Instance | Public IP | Current main purpose |
| --- | --- | --- |
| Compute server (EC2) | 54.251.1.211 | Runs benchmarking reporting pipeline |
| DB server (EC2) | 54.169.91.191 | Runs the Flask server + stores the calibrations and experiments for benchmarking |

To connect to the compute instance:

```java
ssh -i ~/.ssh/id_ed25519_github1 {user_name}@ec2-54-251-1-211.ap-southeast-1.compute.amazonaws.com
```

To connect to the DB server instance

```java
ssh -i ~/.ssh/id_ed25519_github1 {user_name}@ec2-54-169-91-191.ap-southeast-1.compute.amazonaws.com
```

You can always make your life easier by setting up a config shortcut in your local machine to speed up how you connect to places. It’d be something like this:

```java
cd ~/.ssh/
nano config

#Then add and save something on the lines of:
Host cqt-ale-server # you can change this nickname 
    HostName ec2-54-251-1-211.ap-southeast-1.compute.amazonaws.com
    User {user name} # eg. elis
    IdentityFile ~/.ssh/{your github ssh private key file} # eg. id_255..etc
```

To run after setting up config shortcut you just type: `ssh cqt-ale-server`
