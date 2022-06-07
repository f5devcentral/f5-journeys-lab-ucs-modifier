# f5-journeys-lab-ucs-modifier

----
## Contents:
- [Description](#description)
- [Usage](#usage)
- [Quick start](#quick-start)
- [Details of the changes made](#details-of-the-changes-made)
- [Contributing](#contributing)

----
## Description
This tool modifies (sanitizes) UCS from source BIG-IP to be restored onto laboratory BIG-IP that simplifies moving the configuration to laboratory and makes it easier to reproduce issues or test upgrades without affecting production.
Main advantage is that it does not require access to source BIG-IP master key, certificates, keys or passwords.

## Usage
```
usage: ucs-modifier [-h] -u UCS -m MGMT_IP [-p PASSWORD] [--no-replace-cert] [-o OUTPUT]
                    [-d]
Modifies the specified ucs file, removing any sensitive data using values and files from
another big-ip

optional arguments:
  -h, --help            show this help message and exit
  -u UCS, --ucs UCS     Ucs file to modify
  -m MGMT_IP, --mgmt-ip MGMT_IP
                        Management IP of the used big-ip
  -p PASSWORD, --password PASSWORD
                        Root password of big-ip ('default' by default)
  --no-replace-cert     Skip replacing the certificates
  -o OUTPUT, --output OUTPUT
                        Target output file name
  -d, --debug           Enable debug logging
```

## Quick start

Download the docker image:
```
docker pull f5devcentral/f5-journeyslab-ucsmodifier:v1.0.0
```
Run the image in the container interactively:
```
docker run -v <local_directory_with_UCS>:/UCS -it f5devcentral/f5-journeyslab-ucsmodifier:v1.0.0
```
Execute ucs-modifier in the container:
```
ucs-modifier -u <UCS_FILE> -m <IP> -p <PASSWORD>
```
> Modified UCS file (<ORIGINAL_UCS_FILENAME>_modified.ucs) is saved to the same directory as the original UCS file provided.

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

## Contributing

### Bug reporting

Let us know if something went wrong. By reporting issues, you support development of this project and get a chance of having it fixed soon.
Please use bug template available [here](https://github.com/f5devcentral/f5-journeys-lab-ucs-modifier/issues/new?assignees=&labels=&template=bug_report.md&title=%5BBUG%5D).

### Feature requests

Ideas for enhancements are welcome [here](https://github.com/f5devcentral/f5-journeys-lab-ucs-modifier/issues/new?assignees=&labels=&template=feature_request.md&title=%5BFEAT%5D)
