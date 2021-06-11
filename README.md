# CSR-creation-using-TPM
## Step 1: TPM Software Stack
We must install and configure the entire software stack needed for the TPM. First, we have to
install the preconditions and dependences:
```
# apt update
# apt upgrade
# apt -y install autoconf automake libtool pkg-config gcc libssl-dev \
libcurl4-gnutls-dev libdbus-1-dev libglib2.0-dev autoconf-archive \
libcmocka0 libcmocka-dev net-tools build-essential git pkg-config gcc g++ \
m4 libtool automake libgcrypt20-dev libssl-dev uthash-dev autoconf doxygen \
pandoc libsqlite3-dev python-yaml p11-kit opensc gnutls-bin libp11-kit-dev \
python3-yaml cscope libjson-c-dev python3-pyasn1-modules
# apt install -y libengine-pkcs11-openssl1.1
```
Download repositories:
```
$ git clone https://github.com/tpm2-software/tpm2-tss
$ git clone https://github.com/tpm2-software/tpm2-tools
$ git clone https://github.com/tpm2-software/tpm2-abrmd
$ git clone https://github.com/tpm2-software/tpm2-pkcs11
$ git clone https://github.com/OpenSC/libp11.git
$ git clone --depth=1 http://www.github.com/tpm2-software/tpm2-tss-engine
$ git clone https://github.com/yaml/libyaml
```
Install libp11:
```
$ cd libp11
$ git checkout 4084f83
$ ./bootstrap
$ ./configure
$ make -j4
# make install
$ cd ..
```
Install tpm2-tss:
```
$ cd tpm2-tss
$ git checkout 15073cc
$ ./bootstrap -I m4
$ ./configure --with-udevrulesdir=/etc/udev/rules.d --with-udevrulesprefix=70-
$ make -j4
# make install
# useradd --system --user-group tss
# udevadm control --reload-rules && sudo udevadm trigger
# ldconfig
$ cd ..
```
Install tpm2-abrmd:
```
$ cd tpm2-abrmd
$ git checkout 159f503
$ ./bootstrap -I m4
$ ./configure --with-dbuspolicydir=/etc/dbus-1/system.d \
--with-systemdsystemunitdir=/lib/systemd/system \
--with-systemdpresetdir=/lib/systemd/system-preset \
--datarootdir=/usr/share
$ make -j4
# make install
# ldconfig
# pkill -HUP dbus-daemon
# systemctl daemon-reload
# systemctl enable tpm2-abrmd.service
# systemctl start tpm2-abrmd.service
$ dbus-send --system --dest=org.freedesktop.DBus --type=method_call \
--print-reply /org/freedesktop/DBus org.freedesktop.DBus.ListNames \
| grep "com.intel.tss2.Tabrmd" || echo "ERROR: abrmd was not installed
correctly!"
$ cd ..
```
Install tpm2-tools:
```
$ cd tpm2-tools
$ git checkout a8a902c
$ ./bootstrap -I m4
$ ./configure
$ make -j4
# sudo make install
$ cd ..
```
Install LibYAML
```
$ cd libyaml
$ ./bootstrap
$ ./configure
$ make
# make install
$ cd ..
```
Install tpm2-pkcs11:
```
$ cd tpm2-pkcs11
$ git checkout 1b5d296
$ ./bootstrap -I m4
$ ./configure --enable-esapi-session-manage-flags
$ sed -i 's/EVP_PKEY_verify_recover(pkey_ctx, data, data_len,/EVP_PKEY_verify_recover(pkey_ctx, data, (unsigned int*)data_len,/g' ./src/lib/ssl_util.c
$ make -j4
# make install
$ cd ..
```
Install tpm2-tss-engine:
```
$ cd tpm2-tss-engine
$ ./bootstrap
$ ./configure
$ make -j$(nproc)
$ sudo make install
$ cd ..
```
## Step 2: CSR creation
Finally we can create a private key inside the TPM 2.0 and a CSR using this key.

Key creation:
```
$ mkdir key_management
$ cd key_management
$ tpm2tss-genkey -a ecdsa mykey
```
CSR creation:
```
$ openssl req -new -sha256 -engine tpm2tss -keyform engine -key mykey -out my.csr
```
Fill the sections as you need to create the Certificate Signing Request,

Example to Renault SAS:
```
Common Name (CN)	Sensor with Optiga TPM N: 0001
Organizational Unit (OU)	DI-RS CyberSecurity
Organization (O)	Renault SAS
Locality (L)	Paris
State (ST)	Paris
Country (C)	FR
```
