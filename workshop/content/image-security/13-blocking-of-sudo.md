The `su` command isn't the only way of becoming `root` in Linux. The preferred method is actually to use `sudo`. The `/etc/passwd` file being writable means that access to `sudo` also needs to be locked down.

Let's first verify how you could become `root` using `sudo`.

In the container you already have running, create the copy of the `/etc/passwd` file.

```execute
cp /etc/passwd /tmp/passwd
```

and generate the hashed password value:

```execute
HASHED_PASSWORD=`openssl passwd -1 secret`
```

This time when we update the `/etc/passwd` file with the hashed password, we will do it for the current user. We also change the group ID for the user from group ID 0 to group ID 10, corresponding to the `wheel` group.

```execute
cat /tmp/passwd | sed "s%`whoami`:x:`id -u`:0%`whoami`:${HASHED_PASSWORD}:`id -u`:10%" > /etc/passwd
```

The `wheel` group is used because anyone in the `wheel` group can by default use `sudo`.

Run `id` using `sudo`:

```execute
sudo id
```

When prompted, enter the password:

```execute
secret
```

The result should be:

```
uid=0(root) gid=0(root) groups=0(root)
```

This verifies this also would allow one to become `root`.

Stop the current container by killing it.

```execute-2
docker kill `docker ps -ql`
```

The means of disabling use of `sudo` so this cannot be done is to remove the ability for any user in the `wheel` group to run `sudo`. This can be done from the `Dockerfile` using:

```
RUN sed -i.bak -e 's/^%wheel/# %wheel/' /etc/sudoers
```

To verify this change, switch location to the `~/greeting-v6` sub directory.

```execute
cd ~/greeting-v6
```

View the contents of the `Dockerfile` by running:

```execute
cat Dockerfile
```

Build the container image:

```execute
docker build -t greeting .
```

Start a container with an interactive shell:

```execute
docker run -it --rm greeting bash
```

and go through the steps again to test it.

Make the copy of the `/etc/passwd` file.

```execute
cp /etc/passwd /tmp/passwd
```

Generate a hashed password value, where the value of the password is `secret`.

```execute
HASHED_PASSWORD=`openssl passwd -1 secret`
```

and update the `/etc/passwd` file.

```execute
cat /tmp/passwd | sed "s%`whoami`:x:`id -u`:0%`whoami`:${HASHED_PASSWORD}:`id -u`:10%" > /etc/passwd
```

Try again running `sudo`.

```execute
sudo id
```

When prompted, enter the password:

```execute
secret
```

This time it should fail with the error:

```
default is not in the sudoers file.  This incident will be reported.
```

Stop the container once more.

```execute-2
docker kill `docker ps -ql`
```
