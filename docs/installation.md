## Installing Rail-RNA

Make sure you have a recent (>= 2009) OS X or Linux box with at least 8 GB of RAM and Python 2.7.x. For a no-fuss install, enter
```
(INSTALLER=/var/tmp/$(cat /dev/urandom | env LC_CTYPE=C tr -cd 'a-f0-9' | head -c 32);
curl http://verve.webfactional.com/rail -o $INSTALLER; python $INSTALLER -m || true;
rm -f $INSTALLER)
```
in a Unix shell (which you open by running the `Terminal` app). This works only if `python` points to a Python 2.7 executable, and you may have to replace `python` in the command above with a path to one. Otherwise, download the latest version of Rail-RNA [here](https://github.com/nellore/rail/raw/master/releases/install_rail-rna-0.2.4b). If for some reason you need an older version `V`, you can visit [the `releases` subdirectory](https://github.com/nellore/rail/tree/master/releases) of the repo, click on the file `install_rail-rna-V`, and click the `Raw` button to download it.

The file you download will be in the format `install_rail-rna-V`. In a Unix shell, navigate to the directory in which you downloaded the installer. This typically means entering `cd ~/Downloads`. Now enter `chmod +x install_rail-rna-V` to make the installer executable.

### Options

Installation options may now be viewed by entering `./install_rail-rna-V`. They include

* `-i/--install-dir <dir>`: This is the directory in which Rail-RNA should be installed. The default is "/usr/local/raildotbio" if you install for all users, and "~/raildotbio" if you install for just the current user.

* `-n/--no-dependencies`: This installs Rail-RNA without any of its dependencies. Rail-RNA wraps [Bowtie 1](http://bowtie-bio.sourceforge.net/), [Bowtie 2](http://bowtie-bio.sourceforge.net/bowtie2/), [SAMTools](https://samtools.github.io/), [bedGraphToBigWig](http://hgdownload.cse.ucsc.edu/admin/exe/), the [AWS CLI](http://aws.amazon.com/cli/) (in `elastic` mode, to use Rail-RNA on Amazon Elastic MapReduce), and [IPython Parallel](http://ipython.org/ipython-doc/dev/parallel/) (in `parallel` mode, for using Rail-RNA on more than one machine in various distributed computing environments). You can avoid installing them, and Rail-RNA will then look for the appropriate dependencies, but this is not recommended.

* `-p/--prep-dependencies`: This installs Rail-RNA with only dependencies required for its preprocess job flow. The option isn't that useful: Rail-RNA invokes it in a bootstrap script in `elastic` mode for job flows that preprocess input data. This avoids its having to install unnecessary dependencies on cloud computing clusters. The option is overrided by `--no-dependencies`.

* `-y/--yes`: Ordinarily, the user is asked several questions during installation. Invoking this option answers "yes" to all such questions, which means Rail-RNA is installed for all users, the default installation directory `/usr/local/raildotbio` is overwritten if it already exists, which means you upgrade, and the AWS CLI and IPython Parallel are installed if you don't have them already.

* `-s/--symlink-dependencies` You may have already installed versions of Rail's dependencies besides the AWS CLI and IPython Parallel, like SAMTools. By default, Rail-RNA does not change your system's default executables for these dependencies, and you can go on using this software the way you always did. However, if you are installing on a new system for all users and would like to expose Rail-RNA's dependencies, invoke this option to place symlinks to dependencies in /usr/local/bin.

* `--curl <exe>`: The installer downloads with [cURL](http://curl.haxx.se/docs/manpage.html), which tends to come with Linux distributions and Mac OS. It's rarely necessary to specify the path to cURL directly, but that's what this option is for.

You can install Rail-RNA for all users or just for the current user. If you have root access and would like to install for all users, enter
```
sudo ./install_rail-rna-V
```
, enter your password, and proceed. To install for just you, enter
```
./install_rail-rna-V
```
Follow the prompts. Rail-RNA has several dependencies: [Bowtie 1](http://bowtie-bio.sourceforge.net/), [Bowtie 2](http://bowtie-bio.sourceforge.net/bowtie2/), [SAMTools](https://samtools.github.io/), and [bedGraphToBigWig](http://hgdownload.cse.ucsc.edu/admin/exe/) are the critical ones, and they are installed especially for Rail in `~/raildotbio` (single-user install) or `/usr/local/raildotbio` (all-user install) by default. (`-d` may be used to specify a different destination directory, as described above). You may have some of the critical dependencies already, but the installer will copy specific versions to the destination directory anyway to ensure reproducibility of results across installations of a given version of Rail-RNA. Don't worry: nothing happens to your original configuration, and the default versions of the dependencies you already have remain default---that is, unless you invoke the `-s` parameter described above. Optional dependencies are the [AWS CLI](http://aws.amazon.com/cli/), which you'll need to Rail-RNA on Amazon Elastic MapReduce, and [IPython Parallel](http://ipython.org/ipython-doc/dev/parallel/), which you'll need to run Rail-RNA over a conventional computer cluster. The installer will check to see if the optional dependencies are already present. If they're not, it will ask you if you want to install them.

The output of a full installation for all users looks like this.
```
dhcp-pool-133:Downloads testuser$ sudo ./install_rail-rna-0.1.8b 
∀ Rail-RNA v0.1.8b Installer
Rail-RNA can be installed for all users or just the current user.
    * Install for all users? [y/n]: y
Installed Rail-RNA.
IPython is not installed but required for Rail-RNA to work in its
"parallel" mode.
    * Install IPython now? [y/n]: y
AWS CLI is not installed but required for Rail-RNA to work in its
"elastic" mode, on Amazon Elastic MapReduce.
    * Install AWS CLI now? [y/n]: y
Installation log may be found at
/usr/local/raildotbio/rail-rna_installer.log. Configure the
AWS CLI by running "aws configure". Afterwards, run
"aws emr create-default-roles" to use default IAM roles on Amazon
Elastic MapReduce. To learn more about IAM roles, visit
http://docs.aws.amazon.com/IAM/latest/UserGuide/roles-toplevel.html .
Start using Rail by entering "rail-rna".
```
If you plan to use Rail-RNA in its `elastic` mode, on Amazon Elastic MapReduce, you must additionally perform the steps in the following section.

## Setting up Amazon Elastic MapReduce

First off, you need an account with Amazon Web Services (AWS). Sign up [here](http://aws.amazon.com/). If you're new to AWS, we highly recommend following [this](https://github.com/griffithlab/rnaseq_tutorial/wiki/Intro-to-AWS-Cloud-Computing) tutorial by the [Griffith Lab](http://genome.wustl.edu/people/groups/detail/griffith-lab/) at [Wash U](http://wustl.edu/) to get a feel for its use. But if you just want to set up an account, you need only read sections 4-6 contained there.

If you've just installed the AWS CLI along with Rail-RNA, you must configure it. Obtain your AWS Access Key ID and Secret Access Key by following the instructions [here](http://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html).

### A note on security

<span style="color: red; font-size: 250%">Don't put your credentials in a place where anyone else can find them, especially online. Hackers will [steal them](http://readwrite.com/2014/04/15/amazon-web-services-hack-bitcoin-miners-github) and for example mine cryptocurrency on your dollar.</span> If you accidentally put your credentials in a GitHub repo, [scrub it](https://help.github.com/articles/remove-sensitive-data/) and [get new AWS keys](http://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html) immediately.

### Configuring the AWS CLI

Now enter
```
aws configure
```
at the shell prompt. You will be prompted to enter your Access Key ID, your Secret Access Key, a default region name, and a default output format. `us-east-1` is the standard region for US customers, and it spans data centers on the East and West Coasts. However, if you or your input data live elsewhere, you may want to default to one of the other regions listed [here](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html). For example, we find that Rail is fastest at downloading and preprocessing data hosted by the [European Nucleotide Archive](http://www.ebi.ac.uk/ena) in the `eu-west-1` region, which is in Ireland. Finally, hitting enter to accept the default output format is fine.

### Creating default roles

You must also set up [IAM](http://docs.aws.amazon.com/IAM/latest/UserGuide/roles-toplevel.html) roles for use with Elastic MapReduce. A discussion of IAM roles for Elastic MapReduce is available [here](http://docs.aws.amazon.com/ElasticMapReduce/latest/DeveloperGuide/emr-iam-roles.html). IAM is the way AWS securely gives users and services permission to access resources. The default set of permissions tends to work fine; if they don't for you, then you probably know enough about IAM to set up roles for Elastic MapReduce yourself. The typical way to get setting up roles over with is by entering
```
aws emr create-default-roles
```
, but it's possible you're managing a lab whose members are IAM users attached to your AWS account, and you've already given them permissions. In this case, you will need to make sure your lab members have the `iam:GetInstanceProfile`, `iam:GetRole`, and `iam:PassRole` permissions. We also recommend you give them the `iam:ListRoles` permission; otherwise, they won't be able clone clusters on Elastic MapReduce. This is sometimes useful if their job flows fail because their bid prices on the [spot market](http://aws.amazon.com/ec2/purchasing-options/spot-instances/) were too low, and they want to restart job flows easily. With the appropriate permissions, lab members can install Rail-RNA and configure the AWS CLI as described above. This includes their running `aws emr create-default-roles`, which will retrieve the default roles you created for them. Learn more about working with IAM users [here](http://docs.aws.amazon.com/IAM/latest/UserGuide/Using_WorkingWithGroupsAndUsers.html). This step, and if you find yourself having trouble, ask for help in our [Gitter](https://gitter.im/nellore/rail).

## For hackers

### How the installer works

The Rail-RNA installer is nothing but a ZIP archive. When it is executed by Python while it's still compressed, the installer is run. If you unpack the archive first and then execute the directory containing `__main__.py`, Rail-RNA is run, and it will complain if it can't find dependencies. If you're a Python developer who needs to write a custom installer because your software is difficult to package with something like `pip`, you might want to try this approach. A helpful starting point is [this page](https://blogs.gnome.org/jamesh/2012/05/21/python-zip-files/).

### Rolling your own installer

We've made it easy for you to release your own custom version of Rail-RNA.

1. Clone the source at a shell prompt with

        git clone https://www.github.com/nellore/rail.git
. We assume you've cloned to `/home/testuser/rail` below.

2. Edit `src/version.py`, which looks like this:

        !/usr/bin/env python
        """
        version.py
        Part of Rail-RNA

        Stores version number of Rail-RNA as a string.
        """

        version_number = 'devel'
Change "devel" to some version number that diverges from the Rail-RNA versioning scheme. Perhaps you'll use a prefix "C" for "custom," as in "C0.1.0". We call your custom version C below.

3. Edit source files in `src/` however you want. You'll probably want to focus on the files in `src/rna/steps`, which contains a script per step of the Rail-RNA pipeline. To test your changes, rather than starting a command with `rail-rna`, use
        python /home/testuser/rail/src
.

4. Once you're done hacking Rail, run

        sh /home/testuser/rail/make_it_rail.sh
A new installer `install_rail-rna-C` will appear in the `releases/` directory. Run it to install your modified version locally.
