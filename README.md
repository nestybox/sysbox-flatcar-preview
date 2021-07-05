# sysbox-flatcar-preview

This is a preview repository for running the
[Sysbox](https://github.com/nestybox/sysbox) container runtime on Kinvolk's [Flatcar OS](https://kinvolk.io/flatcar-container-linux/).

The repo contains Sysbox binaries built specifically for Flatcar, as well as the
Container Linux configuration yaml needed to install Sysbox on Flatcar.

## Why Sysbox on Flatcar

Flatcar is a container-optimized Linux distro, meaning that the OS is designed
to run workloads inside containers securely.

Running Sysbox on Flatcar further increases container security, as Sysbox
enables containers deployed by Docker or Kubernetes to run with stronger
isolation (Linux user-namespace, procfs and sysfs virtualization, initial mount
locking, etc.). In addition, Sysbox enables containers to run most workloads
that run in virtual machines (including systemd, Docker, and even Kubernetes),
thus enabling new powerful use cases for containers beyond microservice
deployment.

## Installing Sysbox on Flatcar

The method of installation depends on whether Sysbox is installed on
a Docker host or a Kubernetes node.

### Installing Sysbox on Flatcar Docker Hosts

To install Sysbox on a host machine (physical or VM) running Flatcar,
simply use the Container Linux configuration file [here](config/config.yaml).

For example, the steps below show deploy Sysbox on a Google Compute Engine (GCE)
virtual machine:

1) Convert the config yaml to an Ignition yaml. This is done using the `ct` tool,
as described [in this Kinvolk doc](https://kinvolk.io/docs/flatcar-container-linux/latest/provisioning/config-transpiler/):

```console
$ ct --platform=gce < config.yaml > config.ign
```

2) Provision the GCE VM and pass the `config.ign` generated in the prior step as "user-data".

```console
$ gcloud compute instances create flatcar-vm --image-project kinvolk-public --image-family flatcar-stable --zone us-central1-a --machine-type n2-standard-4 --metadata-from-file user-data=config-sysbox-flatcar-gce.ign
Created [https://www.googleapis.com/compute/v1/projects/predictive-fx-309900/zones/us-central1-a/instances/flatcar-vm].
NAME        ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP    EXTERNAL_IP    STATUS
flatcar-vm  us-central1-a  n2-standard-4               10.128.15.196  34.132.170.36  RUNNING
```

Once the VM boots, Sysbox will be installled and running. You can verify this as follows:

```console
core@flatcar-vm ~ $ systemctl status sysbox
● sysbox.service - Sysbox container runtime
     Loaded: loaded (/etc/systemd/system/sysbox.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-07-05 19:35:41 UTC; 3h 10min ago
       Docs: https://github.com/nestybox/sysbox
   Main PID: 1064 (sh)
      Tasks: 2 (limit: 19154)
     Memory: 704.0K
     CGroup: /system.slice/sysbox.service
             ├─1064 /bin/sh -c /opt/bin/sysbox/sysbox-runc --version && /opt/bin/sysbox/sysbox-mgr --version && /opt/bin/sysbox/sysbox-fs --version && /bin/sleep infinity
             └─1084 /bin/sleep infinity
```

Sysbox will also load the shiftfs module into the kernel:

```console
core@flatcar-vm ~ $ lsmod | grep shiftfs
shiftfs                32768  0
```

You can now deploy containers with Docker + Sysbox as follows:

```console
core@flatcar-vm ~ $ docker run --runtime=sysbox-runc -it --rm <some-image>
```

For example, to deploy a "VM-like" container that runs Ubuntu Focal + systemd + Docker inside:

```console
core@flatcar-vm ~ $ docker run --runtime=sysbox-runc -it --rm nestybox/ubuntu-focal-systemd-docker
```

The [Sysbox Quickstart Guide](docs/quickstart/README.md) and [Nestybox blog site](https://blog.nestybox.com/)
have many usage examples.

### Installing Sysbox on Flatcar Kubernetes Nodes

This is not yet supported on Flatcar (though it's supported in [other Linux distros](https://github.com/nestybox/sysbox/blob/master/docs/user-guide/install-k8s.md)).

If you are interested in running Sysbox on Flatcar K8s nodes, please [contact us](#contact) and let us know.

## Contact

Slack: [Nestybox Slack Workspace][slack]

Email: contact@nestybox.com

We are available from Monday-Friday, 9am-5pm Pacific Time.

## Thank You

We thank you **very much** for using Sysbox on Flatcar! We hope you find it
interesting and that it helps you use containers in new and more secure and
powerful ways.

[slack]: https://nestybox-support.slack.com/join/shared_invite/enQtOTA0NDQwMTkzMjg2LTAxNGJjYTU2ZmJkYTZjNDMwNmM4Y2YxNzZiZGJlZDM4OTc1NGUzZDFiNTM4NzM1ZTA2NDE3NzQ1ODg1YzhmNDQ#/
