# pcs-toolbox

Prisma Cloud utility scripts, and a Python SDK for Prisma Cloud APIs.

## Table of Contents

* [Support](#Support)
* [Setup](#Setup)
* [Configuration](#Configuration)
* [Script Usage](#Script-Usage)
    * [CSPM Scripts](#CSPM-Scripts)
    * [CWP Scripts](#CWP-Scripts)

## Support

These scripts have been developed by Prisma Cloud SEs, they are not Supported by Palo Alto Networks.
Nevertheless, the maintainers will make a best-effort to address issues, and (of course) contributors are encouraged to submit issues and pull requests.

## Setup

These scripts are written and tested in Python 3.x.
If you need to install Python 3, you can get more information at [Python's Home Page](https://www.python.org/) ... and you will also need [PIP](https://pypi.python.org/pypi/pip). 

These scripts require the Python packages documented in requirements.txt.
To check and install these packages, execute:

```
pip3 install -r requirements.txt
```

These scripts require the included `pc_lib` library directory to be in the same directory as the script itself.

## Configuration

Configuration for these scripts can be specified each time on the command-line, or can be saved to a configuration file.

Use the `pcs_configure.py` script to save a configuration file. Configuration options include:

- `-u / --username` (REQUIRED) Prisma Cloud Username, or Access Key generated by your Prisma Cloud User
- `-p / --password` (REQUIRED) Password associated with your Prisma Cloud Username, or Secret Key associated with your Access Key
- `--api`           (OPTIONAL) Prisma Cloud API/UI Base URL used to access Prisma Cloud (`app*.prismacloud.*` ... or you can specify a direct `api*.prismacloud.*` URL). 
- `--api_compute`   (OPTIONAL) Prisma Cloud Compute API Base URL used to access Prisma Cloud Compute (For SaaS, use Compute > Manage > System > Downloads: Path to Console). 
- `--config_file`   (OPTIONAL) File containing your Prisma Cloud API configuration settings. Default: `pc-settings.conf`

An Access Key/Secret Key is preferable to using a Username/Password, and Access Keys must be created by a Prisma Cloud User with the permissions required by the script(s) being executed.

Configuration is saved as cleartext JSON, by default in the same directory as the scripts themselves, unless you specify `--config_file`.

### CSPM vs CWP

The `--api` parameter is required for scripts (such as `pcs_alerts_read.py`) that use the Prisma Cloud CSPM API.

The `--api_compute` parameter is required for scripts (such as `pcs_images_packages_read.py`) that use the Prisma Cloud Compute (CWP) API.

For use with On-Premise/Self-Hosted Prisma Cloud Compute, `--username` is your Prisma Cloud Compute User, `--password` is your password or your active bearer token, and `--api` must not be specified.

### References

https://prisma.pan.dev/api/cloud/

https://docs.paloaltonetworks.com/prisma/prisma-cloud/prisma-cloud-admin/manage-prisma-cloud-administrators/create-access-keys.html

https://docs.paloaltonetworks.com/prisma/prisma-cloud/prisma-cloud-admin/manage-prisma-cloud-administrators/prisma-cloud-admin-permissions.html

### Examples:

```
python3 pcs_configure.py --username "Example Access Key" --password "Example Secret Key" --api "app.prismacloud.io"

python3 pcs_configure.py --username "Example Access Key" --password "Example Secret Key" --api "app.prismacloud.io" --config_file ~/example-pc-settings.conf

python3 pcs_configure.py --username "Example Username" --password "Example Password" --api_compute "onpremise.example.com"
```

Run `pcs_configure.py` specifying nothing (other than the optional `--config_file`) to output your current configuration file.

## Script Usage

For detailed documentation of each script's parameters, specify `-h` or `--help` when executing the script.


### CSPM Scripts

#### pcs_policy_set_status.py

Use this script to enable or disable Policies globally for an account (filtered by Policy Type or Compliance Standard).
This is primarily used to set up a new environment with every Policy enabled, or to update an environment after a large number of new Policies have been released.

Example:

```
python3 pcs_policy_set_status.py --policy_type config enable
```

Use this script to enable Policies that are associated with a specific Compliance Standard (or Compliance Standards).

Example:

```
python3 pcs_policy_status.py --policy_type all disable

python3 pcs_policy_status.py --compliance_standard "GDPR" enable
```

#### pcs_user_import.py

Use this script to import a list of Users from a CSV file, assigning the imported Users to the specified Role.
It will check for duplicates before importing.

Example:

```
python3 pcs_user_import.py "example-import-users.csv" "Example Prisma Cloud Role to assign to the imported Users"
```

#### pcs_policy_custom_export.py

Use this script to export custom Policies to a file, for backup ... or to import into another tenant.

Example:

```
python3 pcs_policy_custom_export.py "example-custom-policies.json"
```

#### pcs_policy_custom_import.py

Use this to script import custom Policies.
By default, imported Policies will be disabled, to maintain the status of imported Policies, specify `--maintain_status`
It will check for duplicates before importing.

Example:

```
python3 pcs_policy_custom_import.py "example-custom-policies.json"
```

#### pcs_compliance_export.py

Use this script to export an existing Compliance Standard (and its Requirements and Sections) to a file, for backup ... or to import into another tenant.

Example:

```
python3 pcs_compliance_export.py "GDPR" "example-compliance-standard.json"
```

#### pcs_compliance_import.py

Use this to script import an exported Compliance Standard (and its Requirements and Sections) into a new Compliance Standard.
To import the associated Policy mappings, specify the `--policy` parameter. 
To associate custom Policies requires first running `pcs_policy_custom_export.py` to generate a mapping file (and `pcs_policy_custom_import.py` when importing into another tenant).
It will check for duplicates before importing.

Example:

```
python3 pcs_compliance_import.py "example-compliance-standard.json" "GDPR Imported" --policy
```


### CWP Scripts

#### pcs_images_packages_read.py

Use this script to inspect the packages in all of the container images (or one image, specified by `--image_id`) that have been scanned by Prisma Cloud Compute.

Example:

```
python3 pcs_images_packages_read.py

python3 pcs_images_packages_read.py --image_id "sha256:c004737361182d3cd7f38e6d9ce4a44f2a349b8dc996834e2cba0defcd0cb522"
```

An alternate usage is to specify a package via `--package_id` to search for containers with a specific package, and optionally a version.

Example:

```
python3 pcs_images_packages_read.py --package_type jar --package_id log4j

python3 pcs_images_packages_read.py --package_type jar --package_id log4j:2.14.1 --version_comparison lt --output_to_csv True
```


### Other CSPM and CWP Scripts

#### pcs_cloud_account_import_azure.py (in progress)**

This is the framework for importing a CSV (template in the `templates` directory) with a list of Azure accounts into Prisma Cloud.
Note: This is still a work in progress: the basic import framework is running, but validation of CSV and duplicate name checking has not been implemented yet.

Example:

```
python3 pcs_cloud_account_import_azure.py prisma_cloud_account_import_azure_template.csv
```

#### pcs_posture_endpoint_client.py

This is a generic tool for prototyping with the Cloud Security Posture API.
It sends output to stdout (and optionally to file) and errors/info sent to STDERR, so that it works in a pipeline which makes it `jq` friendly.
Please note this tool is not intended as a replacement for better well-formed scripts and functions.

Example 1: GET request

```
python3 pcs_posture_endpoint_client.py GET /v2/policy
```

Example 2: POST request

```
cat > body.json <<EOF
{ "name": "test standard", "description":"blah" }
EOF
python3 pcs_posture_endpoint_client POST /compliance --request_body body.json
```

#### pcs_compute_endpoint_client.py

This is identical to `pcs_posture_endpoint_client.py`, except it uses the CSPM API rather than the CWP API.
 
