# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License

# Makefile for stout.

AUTOMAKE_OPTIONS = foreign

# NOTE: The stout headers were moved to the Makefile in "include"
# subdirectory to make it easy to preserve the directory structure of
# the header files during installation.
SUBDIRS = . include

include ../versions.am

BOOST = ../boost-$(BOOST_VERSION)
ELFIO = ../elfio-$(ELFIO_VERSION)
GLOG = ../glog-$(GLOG_VERSION)
GMOCK = ../gmock-$(GMOCK_VERSION)
GTEST = $(GMOCK)/gtest
PROTOBUF = ../protobuf-$(PROTOBUF_VERSION)
PICOJSON = ../picojson-$(PICOJSON_VERSION)

# We use "-isystem" instead of "-I" to add Boost to the include search
# path. This disables compiler warnings inside Boost headers since we
# can't easily fix them. See MESOS-3799.
if WITH_BUNDLED_BOOST
BOOST_INCLUDE_FLAGS = -isystem $(BOOST)
endif

if WITH_BUNDLED_ELFIO
ELFIO_INCLUDE_FLAGS = -I$(ELFIO)
endif

if WITH_BUNDLED_GLOG
GLOG_INCLUDE_FLAGS = -I$(GLOG)/src
LIB_GLOG = $(GLOG)/libglog.la
else
LIB_GLOG = -lglog
endif

# TODO(kapil): Handle unbundled GMOCK.
if WITH_BUNDLED_GMOCK
GMOCK_INCLUDE_FLAGS = -I$(GMOCK)/include
GTEST_INCLUDE_FLAGS = -I$(GTEST)/include
LIB_GMOCK = ../libgmock.la
endif

# Enable the INT64 support for PicoJSON.
PICOJSON_INCLUDE_FLAGS =	\
  -DPICOJSON_USE_INT64		\
  -D__STDC_FORMAT_MACROS

if WITH_BUNDLED_PICOJSON
PICOJSON_INCLUDE_FLAGS += -I$(PICOJSON)
endif

if WITH_BUNDLED_PROTOBUF
PROTOBUF_INCLUDE_FLAGS = -I$(PROTOBUF)/src
LIB_PROTOBUF = $(PROTOBUF)/src/libprotobuf.la
else
LIB_PROTOBUF = -lprotobuf
endif

# Tests for stout.
check_PROGRAMS = stout-tests

stout_tests_SOURCES =			\
  tests/adaptor_tests.cpp		\
  tests/base64_tests.cpp		\
  tests/bits_tests.cpp			\
  tests/bytes_tests.cpp			\
  tests/cache_tests.cpp			\
  tests/duration_tests.cpp		\
  tests/dynamiclibrary_tests.cpp	\
  tests/error_tests.cpp			\
  tests/flags_tests.cpp			\
  tests/gzip_tests.cpp			\
  tests/hashmap_tests.cpp		\
  tests/hashset_tests.cpp		\
  tests/interval_tests.cpp		\
  tests/ip_tests.cpp			\
  tests/json_tests.cpp			\
  tests/jsonify_tests.cpp		\
  tests/linkedhashmap_tests.cpp		\
  tests/mac_tests.cpp			\
  tests/main.cpp			\
  tests/multimap_tests.cpp		\
  tests/none_tests.cpp			\
  tests/numify_tests.cpp		\
  tests/option_tests.cpp		\
  tests/os_tests.cpp			\
  tests/os/env_tests.cpp		\
  tests/os/filesystem_tests.cpp		\
  tests/os/process_tests.cpp		\
  tests/os/rmdir_tests.cpp		\
  tests/os/sendfile_tests.cpp		\
  tests/os/signals_tests.cpp		\
  tests/os/socket_tests.cpp		\
  tests/os/strerror_tests.cpp		\
  tests/os/systems_tests.cpp		\
  tests/path_tests.cpp			\
  tests/protobuf_tests.cpp		\
  tests/protobuf_tests.pb.cc		\
  tests/protobuf_tests.pb.h		\
  tests/protobuf_tests.proto		\
  tests/recordio_tests.cpp		\
  tests/result_tests.cpp		\
  tests/set_tests.cpp			\
  tests/some_tests.cpp			\
  tests/strings_tests.cpp		\
  tests/subcommand_tests.cpp		\
  tests/svn_tests.cpp			\
  tests/try_tests.cpp			\
  tests/uuid_tests.cpp			\
  tests/version_tests.cpp

if OS_LINUX
  stout_tests_SOURCES += tests/proc_tests.cpp
endif

# The order of include flags shouldn't matter because we are allowed to disable
# some bundled dependencies in favor of system-installed ones. This would be a
# concern if some headers were to have the same paths.
stout_tests_CPPFLAGS =			\
  -I$(srcdir)/include			\
  $(BOOST_INCLUDE_FLAGS)		\
  $(ELFIO_INCLUDE_FLAGS)		\
  $(GLOG_INCLUDE_FLAGS)			\
  $(GMOCK_INCLUDE_FLAGS)		\
  $(GTEST_INCLUDE_FLAGS)		\
  $(PICOJSON_INCLUDE_FLAGS)		\
  $(PROTOBUF_INCLUDE_FLAGS)		\
  $(AM_CPPFLAGS)

# The order of LDADD flags would matter if the same symbol is provided by
# multiple of these libraries. This is highly unlikely, especially, since we are
# allowed to disable some bundled libraries in lieu of system-installed ones.
stout_tests_LDADD =			\
  $(LIB_GMOCK)				\
  $(LIB_GLOG)				\
  $(LIB_PROTOBUF)			\
  -lsvn_subr-1				\
  -lsvn_delta-1				\
  -lapr-1

# FreeBSD makes the dynamic linking interface available in libc
if !OS_FREEBSD
  stout_tests_LDADD += -ldl
endif

# Both libprocess-tests and benchmarks have a dependency on libgmock.la.
$(LIB_GMOCK):
	@cd .. && $(MAKE) $(AM_MAKEFLAGS) libgmock.la

# We use a check-local target for now to avoid the parallel test
# runner that ships with newer versions of autotools.
# See the following discussion for the workaround:
# http://lists.gnu.org/archive/html/automake/2013-01/msg00051.html
check-local: tests
	./stout-tests

tests: stout-tests

all-local: Makefile

EXTRA_DIST = $(stout_tests_SOURCES)
if !OS_LINUX
EXTRA_DIST += tests/proc_tests.cpp
endif
