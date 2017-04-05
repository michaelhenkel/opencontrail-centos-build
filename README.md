# opencontrail-centos-build

```
export LC_ALL=C
yum install -y epel-release
yum install -y rpm-build scons git python-lxml wget gcc patch make unzip flex bison gcc-c++ openssl-devel autoconf automake vim python-devel python-setuptools protobuf protobuf-devel protobuf-compiler net-snmp-python bzip2 boost-devel tbb-devel  libcurl-devel libxml2-devel zlib-devel cppunit-devel cyrus-sasl-devel cyrus-sasl-lib openssl-devel  cyrus-sasl python-sphinx.noarch kernel-devel libpcap-devel libpcap libnl-devel libnl3-devel libtool
eval "$(ssh-agent -s)"
ssh-add ${SSH_KEY:-"$HOME/.ssh/id_rsa"}
wget --no-check-certificate -O /usr/bin/repo https://storage.googleapis.com/git-repo-downloads/repo
chmod +x /usr/bin/repo
CONTRAIL_VNC_REPO=git@github.com:Juniper/contrail-vnc.git
#CONTRAIL_BRANCH=R3.0
mkdir -p ~/contrail/build/libs
cd ~/contrail/build
grep github ~/.ssh/known_hosts || ssh-keyscan github.com >> ~/.ssh/known_hosts
git config --global user.email "michael.henkel@gmail.com"
#repo init -u $CONTRAIL_VNC_REPO -b $CONTRAIL_BRANCH
repo init -u $CONTRAIL_VNC_REPO 
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

#cd controller/
#git checkout 09e5d48c3b256d361558413ed9befb2dfee9484c
#cd ../vrouter/
#git checkout 4692505afeb6e968a98fe58bc0b901f5df2bcff3
#cd ../contrail-web-controller/
#git checkout 3fe8b5d4c18a2fd97dc5296144cd242884c8a728
#cd ../openstack/contrail-heat/
#git checkout a5a36a22e6862f9e3c76a9864175815ead0501bd
#cd ../../contrail-web-core
#git checkout 7b592db58b5cc6aba5524e032d151389d4183aec
#cd ../tools/sandesh
#git checkout 440c5a58ad09f613419e1ba67bdd0acb688d21bc
export sbtop=~/contrail/build/
cd ./tools/packages/rpm/contrail

kver=`uname -a |awk '{print $3}'`
sed -i "/3.10.0-327.10.1.el7.x86_64/ s/$/ $kver/" contrail.spec
ln -s /usr/src/kernels/3.10.0-327.22.2.el7.x86_64 /usr/src/kernels/3.10.0-327.el7.x86_64 #check your kernel version in /usr/src/kernels
cat <<EOF > dkms.conf.in
PACKAGE_NAME=vrouter
PACKAGE_VERSION="__VERSION__"
PRE_BUILD="utils/dkms/gen_build_info.sh __VERSION__ $dkms_tree/vrouter/__VERSION__/build"
MAKE[0]="'make' -C . KERNELDIR=/lib/modules/${kernelver}/build"
CLEAN[0]="'make' -C . KERNELDIR=/lib/modules/${kernelver}/build"
BUILT_MODULE_NAME[0]="vrouter"
DEST_MODULE_LOCATION[0]="/kernel/net/vrouter"
AUTOINSTALL="yes"
EOF
sed -i "s#tools/packaging/common/control_files#tools/packages/rpm/contrail#g" contrail.spec
sed -i "s#%{_distrorpmpkgdir}#%{_sbtop}/%{_distrorpmpkgdir}#g" contrail.spec  |grep _distrorpmpkgdir
JOBS=`nproc`
SCONSFLAGS="-j $JOBS -Q debug=1" rpmbuild -ba --define "_sbtop $sbtop" contrail.spec

```
