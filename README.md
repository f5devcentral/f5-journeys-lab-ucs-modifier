# f5-journeys-lab-ucs-modifier

----
## Contents:
- [Description](#description)
- [Quick start](#quick-start)
- [Installation](#installation)
- [Preparation of the destination platform](#preparation-of-the-destination-platform)
- [Usage](#usage)
- [Available arguments](#available-arguments)
- [Details of the changes made](#details-of-the-changes-made)
- [Contributing](#contributing)

----
## Description
This tool modifies (sanitizes) UCS from source BIG-IP (11.x and later) to be restored onto laboratory BIG-IP that simplifies moving the configuration to laboratory and makes it easier to reproduce issues or test upgrades without affecting production.
Main advantage is that it does not require access to source BIG-IP master key, certificates, keys or passwords.

## Quick start

```
docker pull f5devcentral/f5-journeyslab-ucsmodifier:v1.0.3
docker run -v <local_directory_with_UCS>:/UCS -it f5devcentral/f5-journeyslab-ucsmodifier:v1.0.3
ucs-modifier -u <UCS_FILE_NAME>.ucs -m <IP> -p '<PASSWORD>'
```

## Installation

Download the docker image:
```
docker pull f5devcentral/f5-journeyslab-ucsmodifier:v1.0.3
```

### Optional step (if the tool is to be run on an offline system):

Save and compress the image:
```
docker save f5devcentral/f5-journeyslab-ucsmodifier:v1.0.3 | gzip > f5-journeyslab-ucsmodifier_v1.0.3.tar.gz
```

Transfer archive to the offline system

Load image from the archive:
```
docker load < f5-journeys-lab-ucs-modifier_v1.0.3.tar.gz
```

## Preparation of the destination platform

Destination BIG-IP (lab):
+ must have sufficient resources (e.g. RAM for BIG-IP VE with multiple modules)
+ must be properly licensed (for provisioned modules) before loading a modified UCS

## Usage

1. Run the image in the container interactively:
```
docker run -v <local_directory_with_UCS>:/UCS -it f5devcentral/f5-journeyslab-ucsmodifier:v1.0.3
```

2. Execute ucs-modifier in the container:
```
ucs-modifier -u /UCS/<UCS_FILE_NAME>.ucs -m <MGMT_IP> -p '<PASSWORD>'
```
> - Destination BIG-IP lab should be prepared (and ready for ssh connection) before executing the "ucs-modifier..." command as it will connect to the target BIG-IP (<MGMT_IP>) to get some basic config (passwords, management IP, gateway - [Details of the changes made](#details-of-the-changes-made)).
> - By defualt modified UCS file (<UCS_FILE_NAME>_modified.ucs) is saved to the same directory as the original UCS file provided (<UCS_FILE_NAME>.ucs).
> - Only unencrypted UCS files (with .ucs extension) are supported.

3. Transfer <UCS_FILE_NAME>_modified.ucs to destination BIG-IP (lab)
```
scp /UCS/<UCS_FILE_NAME>_modified.ucs <MGMT_IP>:/var/local/ucs/
```

4. Load UCS:

+ on the same platform type without the license:
```
tmsh load sys ucs <UCS_FILE_NAME>_modified.ucs no-license
```
> From version 1.0.1 the original bigip.license file is removed from UCS (avoiding license errors if "no-license" parameter is omitted).

+ on a different platform type:
```
tmsh load sys ucs <UCS_FILE_NAME>_modified.ucs platform-migrate
```
> With platform-migrate option, license is excluded by default.

## Available arguments
```
usage: ucs-modifier [-h] -u UCS -m MGMT_IP [-p 'PASSWORD']
                    [--no-replace-cert] [-o OUTPUT] [-d]

Modifies the specified ucs file, removing any sensitive data using values and
files from destination BIG-IP

optional arguments:
  -h, --help            show this help message and exit
  -u UCS, --ucs UCS     Ucs file to modify
  -m MGMT_IP, --mgmt-ip MGMT_IP
                        Management IP of the target BIG-IP
  -p 'PASSWORD', --password 'PASSWORD'
                        Root password of the target BIG-IP in single quotes ('default' by
                        default)
  --no-replace-cert     Skip replacing the certificates
  -o OUTPUT, --output OUTPUT
                        Target output UCS file name
  -d, --debug           Enable debug logging
```

## Details of the changes made
+ Replaces User IDs and passwords with the ones from target big-ip
+ Replaces MGMT IP and default gateway with the ones from target big-ip
+ Replaces encrypted passwords and secrets with a clear text that will not allow for decryption but will allow for configuration load. Example: password replaced on monitor object will not allow for backend communication but will allow for configuration load.
+ Removes restrictions on SSHD and HTTPD access
+ Disables failsafe functionality; it is not triggered in the environment for which was not configured
+ Remove traps and users sections from 'sys snmp' section
+ Deletes obsolete EPSEC files (all but most recent)
+ Removes interface configuration (they will be have default config)
+ Restores local authentication
+ Disables secure password enforcement
+ Removes 'security datasync' sections referencing non-existing files
+ Disables persistence cookie encryption
+ Removes trunks from ha-group configuration
+ Removes remote syslog addresses
+ Generates new certificates matching the ciphers configuration
+ Generates html files showing the changes made: <work_dir_in_container>/config_comparison/
+ Removes the license file from UCS

## Contributing

### Bug reporting

Let us know if something went wrong. By reporting issues, you support development of this project and get a chance of having it fixed soon.
Please use bug template available [here](https://github.com/f5devcentral/f5-journeys-lab-ucs-modifier/issues/new?assignees=&labels=&template=bug_report.md&title=%5BBUG%5D).

### Feature requests

Ideas for enhancements are welcome [here](https://github.com/f5devcentral/f5-journeys-lab-ucs-modifier/issues/new?assignees=&labels=&template=feature_request.md&title=%5BFEAT%5D)
