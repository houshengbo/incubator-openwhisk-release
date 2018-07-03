<!--
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
-->

# Download OpenWhisk

The OpenWhisk source code may be downloaded from https://dist.apache.org/repos/dist/dev/incubator/openwhisk/apache-openwhisk-0.9.0-incubating-rc2. The current release is version 0.9.0, and the artifact for this OpenWhisk source code release is called `"openwhisk-0.9.0-incubating-sources.tar.gz"`.

```
# download from your terminal with wget
wget https://dist.apache.org/repos/dist/dev/incubator/openwhisk/apache-openwhisk-0.9.0-incubating-rc2/openwhisk-0.9.0-incubating-sources.tar.gz
```

# Verify the SHA-512 checksums, and signature

You need to install `gpg` on your local machine.

- For Ubuntu user, run the following command:
  ```
  sudo apt-get install gnupg
  ```

- For Mac user, run the following command:
  ```
  brew install gpg
  ```

Before using `gpg` to verify the OpenWhisk release integrity, you should [verify `gpg`'s own integrity](https://gnupg.org/download/integrity_check.html).

The public key used to verify the OpenWhisk checksums can be found [here](https://dist.apache.org/repos/dist/dev/incubator/openwhisk/KEYS). Download the key and import it on your local machine.

```
gpg --import <key_file>
```

The parameter <key_file> is the file, where the public key is saved.

To generate the SHA512 checksum:

```
gpg --print-md SHA512 <artifact>
```

The parameter <artifact> is the file of the artifact `"openwhisk-0.9.0-incubating-sources.tar.gz"`. Compare the content with the [SHA512 file](https://dist.apache.org/repos/dist/dev/incubator/openwhisk/apache-openwhisk-0.9.0-incubating-rc2/openwhisk-0.9.0-incubating-sources.tar.gz.sha512).

Download the [signature](https://dist.apache.org/repos/dist/dev/incubator/openwhisk/apache-openwhisk-0.9.0-incubating-rc2/openwhisk-0.9.0-incubating-sources.tar.gz.asc), and verify it with the command:

```
gpg --verify openwhisk-0.9.0-incubating-sources.tar.gz.asc openwhisk-0.9.0-incubating-sources.tar.gz
```

# Unpack the release

Once you have verified the artifact checksums, unzip and extract the source code from the file `"openwhisk-0.9.0-incubating-sources.tar.gz"`. This will yield a directory called `"incubator-openwhisk-0.9.0-incubating"` on your local machine.

You can use the following command to untar the package:

```
tar -xvzf openwhisk-0.9.0-incubating-sources.tar.gz 
```

# Verify the Apache license header

OpenWhisk uses a tool called [`scanCode`](https://github.com/apache/incubator-openwhisk-utilities/tree/master/scancode) to check the license header of each source file included in the release. Please check the [`scanCode` tutorial](https://github.com/apache/incubator-openwhisk-utilities) for instructions to download and run the tool and scan all code in the release.

# Building OpenWhisk

The following instructions walk you through the steps to build OpenWhisk 0.9.0 for the supported operation systems which are Ubuntu and Mac OS.

## Prerequisites

We recommend you set the environment variable `$OPENWHISK_HOME` on your local machine to the extracted directory `incubator-openwhisk-0.9.0-incubating`.
It is an _error_ to set `$OPENWHISK_HOME` to an invalid or incorrect path (including an empty string). If you do not wish to use the environment variable, make sure it is not defined in your environment at all. The rest of the instructions assume you have defined this environment variable.

### Ubuntu users

If you are a Ubuntu user, our suggested version is between 14.04 and 16.04. Open a terminal, go to the directory of `"incubator-openwhisk-0.9.0-incubating"`, and run the script `"all.sh"` under `tools/ubuntu-setup`:

```
cd $OPENWHISK_HOME
./tools/ubuntu-setup/all.sh
```

Then, you need to add the permission for the current Ubuntu user in Docker:

```
sudo usermod -a -G docker $USER
```

Install Gradle 4.8.1 or the latest version with the following commands:

```
sudo add-apt-repository ppa:cwchien/gradle
sudo apt-get update
sudo apt-get install gradle
```

### Mac users

If you are a Mac user, please run the following script:

```
echo '
# install homebrew
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
# install cask
brew tap caskroom/cask
# install for finding alternative versions (java8)
brew tap caskroom/versions
# install java 8
brew cask install java8
# install scala
brew install scala
# install pip
sudo easy_install pip
# install script prerequisites
sudo -H pip install docker==2.2.1 ansible==2.5.2 jinja2==2.9.6 couchdb==1.1 httplib2==0.9.2 requests==2.10.0' | bash
```

Then, activate docker0 network with the following command:

```
sudo ifconfig lo0 alias 172.17.0.1/24
```

Install Gradle 4.8.1 or the latest version with the following commands:

```
brew install gradle
```

After running the script above, you should have all you need to install OpenWhisk. Normally there should be no error, but it
some error messages pop-up, please [log an issue](https://github.com/apache/incubator-openwhisk/issues) for the OpenWhisk community to assist you.


## Build the source code

Open the terminal, and go to the directory $OPENWHISK_HOME. Install the Gradle wrapper with the command:

```
gradle wrapper
```

Then, run the following gradlew command to build the source code:

```
./gradlew distDocker
```

Once the build is complete, you will have Docker images necessary to run OpenWhisk. If your build fails,
first make sure `docker` is running correctly by verifying with the command `docker images`. If you continue
to encounter build problems, please seek help by opening [an issue](https://github.com/apache/incubator-openwhisk/issues) for the OpenWhisk community to assist you.


## Deploy OpenWhisk

Make sure you are in the `$OPENWHISK_HOME/ansible` directory, and run the following ansible
scripts one after the other. If any of these commands fail, the rest of the deployment will
not work correctly and you should [seek help](https://github.com/apache/incubator-openwhisk/issues).

```
ansible-playbook -i environments/local setup.yml
ansible-playbook -i environments/local couchdb.yml
ansible-playbook -i environments/local initdb.yml
ansible-playbook -i environments/local wipe.yml
ansible-playbook -i environments/local openwhisk.yml
ansible-playbook -i environments/local postdeploy.yml
ansible-playbook -i environments/local apigateway.yml
```

### Troubleshooting

There are several know issues you may encounter:

- The API gateway packages may not install correctly on some systems. You can try to recover, and then proceed with the rest of the playbooks by running `ansible-playbook -i environments/local routemgmt.yml`. See [this issue](https://github.com/apache/incubator-openwhisk/issues/3804) for more details. If you are not interested in the API gateway, you can ignore this error (and also skip the last playbook).

- If the playbook fail, sometimes tearing down the (partially) deployed system and retrying is sufficient to recover. To clean your local environment, run the following commands:
  ```
  ansible-playbook -i environments/local apigateway.yml -e mode=clean
  ansible-playbook -i environments/local openwhisk.yml -e mode=clean
  ansible-playbook -i environments/local couchdb.yml -e mode=clean
  ```
  Then, run the ansible playbooks again to see if there are still failures. If you still fail to deploy OpenWhisk,
please [log an issue](https://github.com/apache/incubator-openwhisk/issues) for the OpenWhisk community to assist you.

- Since this release does not ship the code for test cases, any error regarding test cases can be ignored.

## Run OpenWhisk

The easiest way to try out OpenWhisk is to use OpenWhisk CLI. Please find the configuration [here](https://github.com/apache/incubator-openwhisk/blob/master/docs/cli.md).
For example, you can configure you CLI with the following command, if you have deployed OpenWhisk locally:

```
wsk property set --apihost 172.17.0.1 --auth $(cat ${OPENWHISK_HOME}/ansible/files/auth.guest)
```

Run the following command to echo an input message:

```
bin/wsk -i action invoke /whisk.system/utils/echo -p message hello --result
```

Output:

```json
{
  "message": "hello"
}
```

If you can get the above message, you have successfully deployed OpenWhisk on your local machine.

If you want to learn how to use OpenWhisk in a more comprehensive way, please visit the [OpenWhisk website](https://openwhisk.apache.org/) or
[OpenWhisk repository](https://github.com/apache/incubator-openwhisk/blob/master/README.md) for detailed information.
