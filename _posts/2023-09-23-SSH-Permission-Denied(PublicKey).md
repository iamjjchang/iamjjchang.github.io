---
title: Zerotrust Managed Network
date: 2023-09-23 12:00:00 -500
categories: [SSH, Ubuntu, macOS]
tags: [Network]     # TAG names should always be lowercase
---

1. Incorrect SSH Key Pair
The most common cause of the error is an incorrect SSH key pair. The user might be using a different key pair than the one associated with the instance, or the public key might not be added to the authorized_keys file on the server.

2. Incorrect File Permissions
The file permissions of the SSH key pair, authorized_keys file, or the .ssh directory might not be set correctly. The private key file should have permission 600, the authorized_keys file should have permission 644, and the .ssh directory should have permission 700.

3. Incorrect SSH Configuration
The SSH client configuration might not be set up correctly. For example, the user might be trying to connect to the wrong port or using the wrong username.

Resolving the “Permission Denied (publickey)” Error

```sh
$ chmod 600 <private-key-file>
$ chmod 644 <authorized_keys-file>
$ chmod 700 ~/.ssh
```