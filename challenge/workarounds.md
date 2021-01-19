# Workarounds

## GCP VM instance access via ssh after restart

You might have these kinds of problems while attempting to connect to a GCP VM instance after stopping them (_public ip may change_).

### Remote host identification has changed

Try deleting the host of the $HOME/ssh/known_hosts file with the command:

```bash
ssh-keygen -R <VM instance public ip address>
```

Try to connect again via SSH.

> _Based on [https://www.digitalocean.com/community/questions/warning-remote-host-identification-has-changed](https://www.digitalocean.com/community/questions/warning-remote-host-identification-has-changed)_

### Operation timed out

Go to the VM instance, and Edit it.
Find Custom metadata option and Click Add item and Type ```startup-script``` **as a key** and Copy and past the command ```sudo ufw allow ssh``` **as a value**. This command will enabled port 22 for SSH.

Restart the instance and try to connect again via SSH.

> _Based on [https://serverfault.com/questions/953290/google-compute-engine-ssh-connect-to-host-ip-port-22-operation-timed-out](https://serverfault.com/questions/953290/google-compute-engine-ssh-connect-to-host-ip-port-22-operation-timed-out)_
