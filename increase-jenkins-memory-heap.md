# Increase Jenkins Memory Heap

To add more Java heap space for Jenkins, you can modify the `JAVA_OPTS` environment variable in the Jenkins systemd service file. Since you're using systemd to manage Jenkins, you can follow these steps:

## Debian System

1. Open the Jenkins systemd service file for editing:
```bash
sudo vi /lib/systemd/system/jenkins.service
```

2. Add the following lines to the file:

```bash
[Service]
Environment="JAVA_OPTS=-Djava.awt.headless=true -Xmx2g"  # Adjust the heap size (e.g., -Xmx2g for 2 GB)
```
Replace *`-Xmx2g`* with the desired heap size. For example, if you want to allocate 4 GB of heap space, you can use *`-Xmx4g`*.

3. Save the file and exit the editor.

4. Reload the systemd configuration to apply the changes:
```bash
sudo systemctl daemon-reload
```

5. Restart the Jenkins service for the changes to take effect:
```bash
sudo systemctl restart jenkins
```

These steps will manually create the override file for Jenkins, allowing you to specify the *JAVA_OPTS* environment variable to adjust the Java heap size.

## References
(Jenkins Systemd Services)[https://www.jenkins.io/doc/book/system-administration/systemd-services/]
