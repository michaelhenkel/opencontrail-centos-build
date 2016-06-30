# opencontrail-centos-build

```
yum install -y epel-release
yum install -y rpm-build scons git python-lxml wget gcc patch make unzip flex bison gcc-c++ openssl-devel autoconf automake vim python-devel python-setuptools protobuf protobuf-devel protobuf-compiler net-snmp-python bzip2 boost-devel tbb-devel  libcurl-devel libxml2-devel zlib-devel cppunit-devel cyrus-sasl-devel.x86_64 cyrus-sasl-lib.x86_64 openssl-devel  cyrus-sasl.x86_64
eval "$(ssh-agent -s)"
ssh-add ${SSH_KEY:-"$HOME/.ssh/id_rsa"}
wget --no-check-certificate -O /usr/bin/repo https://storage.googleapis.com/git-repo-downloads/repo
chmod +x /usr/bin/repo
CONTRAIL_VNC_REPO=git@github.com:Juniper/contrail-vnc.git
CONTRAIL_BRANCH=R3.0
CONTRAIL_VERSION=3.0.2
mkdir -p ~/contrail/build/libs
cd ~/contrail/build
grep github ~/.ssh/known_hosts || ssh-keyscan github.com >> ~/.ssh/known_hosts
git config --global user.email "michael.henkel@gmail.com"
repo init -u $CONTRAIL_VNC_REPO -b $CONTRAIL_BRANCH
repo sync
cd libs
wget http://sourceforge.net/projects/libipfix/files/libipfix/libipfix_110209.tgz
wget http://downloads.datastax.com/cpp-driver/centos/7/dependencies/libuv/v1.8.0/libuv-1.8.0-1.el7.centos.x86_64.rpm
wget http://downloads.datastax.com/cpp-driver/centos/7/dependencies/libuv/v1.8.0/libuv-devel-1.8.0-1.el7.centos.x86_64.rpm
wget http://downloads.datastax.com/cpp-driver/centos/7/cassandra/v2.4.2/cassandra-cpp-driver-2.4.2-1.el7.centos.x86_64.rpm
wget http://downloads.datastax.com/cpp-driver/centos/7/cassandra/v2.4.2/cassandra-cpp-driver-devel-2.4.2-1.el7.centos.x86_64.rpm
rpm -i libuv-devel-1.8.0-1.el7.centos.x86_64.rpm libuv-1.8.0-1.el7.centos.x86_64.rpm cassandra-cpp-driver-devel-2.4.2-1.el7.centos.x86_64.rpm cassandra-cpp-driver-2.4.2-1.el7.centos.x86_64.rpm
git clone https://github.com/edenhill/librdkafka/
tar zxvf libipfix_110209.tgz
cd libipfix_110209
./configure
make && make install
cd ../librdkafka/
./configure
make && make install
cd ../
git clone https://github.com/michaelhenkel/zookeeper-el7-rpm
mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}
cd zookeeper-el7-rpm
wget http://apache.cs.uu.nl/zookeeper/zookeeper-3.4.8/zookeeper-3.4.8.tar.gz
mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}
cp -r * ~/rpmbuild/SOURCES/
rpmbuild -ba zookeeper.spec
rpm -i ~/rpmbuild/RPMS/x86_64/libzookeeper-3.4.8-1.x86_64.rpm ~/rpmbuild/RPMS/x86_64/libzookeeper-devel-3.4.8-1.x86_64.rpm ~/rpmbuild/RPMS/x86_64/python-zookeeper-3.4.8-1.x86_64.rpm
cd ~/contrail/build/third_party
python fetch_packages.py
cd ..
export sbtop=~/contrail/build/
cd tools/packages/rpm/contrail
rpmbuild -ba --define "_sbtop $sbtop" contrail.spec

```
