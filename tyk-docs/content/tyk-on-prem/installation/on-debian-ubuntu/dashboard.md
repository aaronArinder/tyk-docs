---
date: 2017-03-22T16:08:31Z
Title: Dashboard on Ubuntu
tags: ["Tyk Dashboard", "Self-Managed", "Installation", "Ubuntu", "Debian"]
description: "How to install the Tyk Dashboard as part of the Tyk Stack on Ubuntu or Debian using Ansible or shell scripts"
menu:
  main:
    parent: "On Debian / Ubuntu"
weight: 1
url: /tyk-on-premises/debian-ubuntu/dashboard
aliases:
  - /getting-started/installation/with-tyk-on-premises/on-ubuntu/dashboard/
  - /getting-started/installation/with-tyk-on-premises/on-debian-ubuntu/dashboard/
---
{{< tabs_start >}}
{{< tab_start "Ansible" >}}
<br />
{{< note >}}
**Requirements**

[Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) is required to run the following commands. Instructions on how install Tyk Dashboard with shell is in the <b>Shell</b> tab.
{{< /note >}}

## Getting Started
1. clone the [tyk-ansible](https://github.com/TykTechnologies/tyk-ansible) repositry

```bash
$ git clone https://github.com/TykTechnologies/tyk-ansible
```

2. `cd` into the directory
```.bash
$ cd tyk-ansible
```

3. Run initalization script to initialize environment

```bash
$ sh scripts/init.sh
```

4. Modify `hosts.yml` file to update ssh variables to your server(s). You can learn more about the hosts file [here](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)

5. Run ansible-playbook to install `tyk-dashboard`

```bash
$ ansible-playbook playbook.yml -t tyk-dashboard
```
{{< tab_end >}}
{{< tab_start "Shell" >}}
## <a name="install-tyk-dashboard-ubuntu"></a>Install Tyk Dashboard on Ubuntu

Tyk has its own APT repositories hosted by the kind folks at [packagecloud.io][1], which makes it easy, safe and secure to install a trusted distribution of the Tyk Gateway stack.

This tutorial has been tested on Ubuntu 16.04 & 18.04 with few if any modifications. We will install the Tyk Dashboard with all dependencies locally.

### Prerequisites

- Have MongoDb and Redis installed - see [here][2] for details.
- Ensure port `3000` is available. This is used by the Dashboard to provide the GUI and the Developer Portal.

### Step 1: Set up our APT Repositories

First, add our GPG key which signs our binaries:

```bash
curl -L https://packagecloud.io/tyk/tyk-dashboard/gpgkey | sudo apt-key add -
```

Run update:

```bash
sudo apt-get update
```

Since our repositories are installed via HTTPS, you will need to make sure APT supports this:

```bash
sudo apt-get install -y apt-transport-https
```

Now lets add the required repos and update again (notice the `-a` flag in the second Tyk commands - this is important!):

```bash
echo "deb https://packagecloud.io/tyk/tyk-dashboard/ubuntu/ bionic main" | sudo tee /etc/apt/sources.list.d/tyk_tyk-dashboard.list

echo "deb-src https://packagecloud.io/tyk/tyk-dashboard/ubuntu/ bionic main" | sudo tee -a /etc/apt/sources.list.d/tyk_tyk-dashboard.list

sudo apt-get update
```

**What we've done here is:**

- Added the Tyk Dashboard repository
- Updated our package list

### Step 2: Install the Tyk Dashboard

We're now ready to install the Tyk Dashboard. To install run:

```bash
sudo apt-get install -y tyk-dashboard
```

What we've done here is instructed `apt-get` to install the Tyk Dashboard without prompting. Wait for the downloads to complete.

When the the Tyk Dashboard has finished installing, it will have installed some `init` scripts, but it will not be running yet. The next step will be to setup each application - thankfully this can be done with three very simple commands.

#### Verify the origin key (optional)

Debian packages are signed with the repository keys. These keys are verified at the time of fetching the package and is taken care of by the `apt` infrastructure. These keys are controlled by PackageCloud, our repository provider. For an additional guarantee, it is possible to verify that the package was indeed created by Tyk by verifying the `origin` certificate that is attached to the package.

First, you have to fetch Tyk's signing key and import it.

```bash
wget https://keyserver.tyk.io/tyk.io.deb.signing.key
gpg --import tyk.io.deb.signing.key
```

Then, you have to either,
- sign the key with your ultimately trusted key
- trust this key ultimately

The downloaded package will be available in `/var/cache/apt/archives`. Assuming you found the file `tyk-gateway-2.9.4_amd64.deb` there, you can verify the origin signature.

```bash
gpg --verify d.deb
gpg: Signature made Wed 04 Mar 2020 03:05:00 IST
gpg:                using RSA key F3781522A858A2C43D3BC997CA041CD1466FA2F8
gpg: Good signature from "Team Tyk (package signing) <team@tyk.io>" [ultimate]
```

## Configure Tyk Dashboard

### Prerequisites

You need to ensure the MongoDB and Redis services are running before proceeding.

{{< note success >}}
**Note**  

You need to replace `<hostname>` for `--redishost=<hostname>`, and `<IP Address>` for `--mongo=mongodb://<IP Address>/` with your own values to run this script.
{{< /note >}}


We can set the dashboard up with a helper setup command script. This will get the dashboard set up for the local instance:

```bash
sudo /opt/tyk-dashboard/install/setup.sh --listenport=3000 --redishost=<hostname> --redisport=6379 --mongo=mongodb://<IP Address>/tyk_analytics --tyk_api_hostname=$HOSTNAME --tyk_node_hostname=http://localhost --tyk_node_port=8080 --portal_root=/portal --domain="XXX.XXX.XXX.XXX"
```

{{< note success >}}
**Note**  

Make sure to use the actual DNS hostname or the public IP of your instance as the last parameter.
{{< /note >}}


What we have done here is:

- `--listenport=3000`: Told the Tyk Dashboard (and Portal) to listen on port 3000.
- `--redishost=<hostname>`: The Tyk Dashboard should use the local Redis instance.
- `--redisport=6379`: The Tyk Dashboard should use the default port.
- `--domain="XXX.XXX.XXX.XXX"`: Bind the dashboard to the IP or DNS hostname of this instance (required).
- `--mongo=mongodb://<IP Address>/tyk_analytics`: Use the local MongoDB (should always be the same as the gateway).
- `--tyk_api_hostname=$HOSTNAME`: The Tyk Dashboard has no idea what hostname has been given to Tyk, so we need to tell it, in this instance we are just using the local HOSTNAME env variable, but you could set this to the public-hostname/IP of the instance.
- `--tyk_node_hostname=http://localhost`: The Tyk Dashboard needs to see a Tyk node in order to create new tokens, so we need to tell it where we can find one, in this case, use the one installed locally.
- `--tyk_node_port=8080`: Tell the dashboard that the Tyk node it should communicate with is on port 8080.
- `--portal_root=/portal`: We want the portal to be shown on `/portal` of whichever domain we set for the portal.

### Step 1: Enter your Dashboard License

Add your license in `/opt/tyk-dashboard/tyk_analytics.conf` in the `license` field.

### Step 2: Start the Tyk Dashboard

Start the dashboard service, and ensure it will start automatically on system boot.

```bash
sudo systemctl start tyk-dashboard
sudo systemctl enable tyk-dashboard
```

### Step 3: Install Tyk Gateway

Follow the [Gateway installation instructions](/docs/getting-started/installation/with-tyk-on-premises/on-ubuntu/gateway/) to connect to your Dashboard instance before you continue on to step 4.

### Step 4: Bootstrap the Dashboard with an initial User and Organisation

Go to:

```{copy.Wrapper}
127.0.0.1:3000
```

You should get to the Tyk Dashboard Setup screen:

![Tyk Dashboard Bootstrap Screen][3]

### Step 5 - Create your Organisation and Default User

You need to enter the following:

- Your **Organisation Name**
- Your **Organisation Slug**
- Your User **Email Address**
- Your User **First and Last Name**
- A **Password** for your User
- **Re-enter** your user **Password**

{{< note success >}}
**Note**  

For a password, we recommend a combination of alphanumeric characters, with both upper and lower case
letters.
{{< /note >}}

Click **Bootstrap** to save the details.

### Step 6 - Login to the Dashboard

You can now log in to the Tyk Dashboard from `127.0.0.1:3000`, using the username and password created in the Dashboard Setup screen.

[1]: https://packagecloud.io/tyk
[2]: /docs/getting-started/installation/with-tyk-on-premises/on-ubuntu/#prerequisites
[3]: /docs/img/dashboard/system-management/bootstrap_screen.png
{{< tab_end >}}
{{< tabs_end >}}
