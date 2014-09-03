# vagrant-netflix-ice

This is a Vagrant project for deploying Netflix's [Ice](https://github.com/Netflix/ice). Ice attempts to provide a birds-eye view of your Amazon Web Services (AWS) usage and spend. Therefore, in order to use Ice, you'll need an AWS account with [detailed billing reports](http://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/detailed-billing-reports.html) enabled.

## Configuration

This project provisions Ice inside of a Vagrant virtual machine using Ansible. All of the required configuration attributes can be found inside `group_vars/all.sample`.

In order to get started, make a copy of `group_vars/all.sample`:

```bash
$ cp group_vars/all.sample group_vars/all
```

 Now, update the following attributes:

- `aws_access_key` - AWS access key ID
- `aws_secret_key` - AWS secret key
- `ice_billing_buckets` - S3 bucket housing the detailed billing reports
- `ice_work_bucket` - S3 bucket where Ice deposits processed billing information
- `ice_aws_account_mapping` - Mapping of AWS account ID and name

**Note**: Ensure that the account associated with your `aws_access_key` and `aws_secret_key` can read from the `ice_billing_buckets`, read and write to the `ice_work_bucket`, and make `DescribeReservedInstancesOfferings` EC2 API calls. `ice-iam-policy.json` provides a sample IAM policy.

## Usage

After modifying all of the required configuration attributes within `group_vars/all`, launch the Vagrant virtual machine:

```bash
$ vagrant up
```

This step launches the Vagrant virtual machine and triggers the Ansible provisioner. Ansible installs and configures Ice within Apache Tomcat.

Ice itself is composed of a processor and a reader. The processor needs to run first in order to arrange your billing data. After the processor is done processing, the reader can be used to view results. In order to follow what the processor is doing, you can tail the Tomcat logs from within the virtual machine:

```bash
$ vagrant ssh
$ sudo tail -f /var/log/tomcat7/catalina.out
2014-08-25 02:21:43,826 [localhost-startStop-1] INFO  BootStrap  - Starting ice...
```

**Note**: `data/ice_processor` and `data/ice_reader` are Ice's working directories and are mapped into the virtual machine by Vagrant. This is done so that working data persists across instances of the virtual machine.
