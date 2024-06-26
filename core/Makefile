# -*- Mode: makefile-gmake -*-

.PHONY: clean all debug release pkgconfig
.PHONY: print_debug_lib print_release_lib
.PHONY: print_debug_path print_release_path

#
# Required packages
#

PKGS = libnfcdef libglibutil glib-2.0 gobject-2.0 gio-2.0 gio-unix-2.0

#
# Default target
#

all: debug release

#
# Library name
#

NAME = nfc-core
LIB_NAME = lib$(NAME)
LIB = $(LIB_NAME).a

#
# Sources
#

SRC = \
  nfc_adapter.c \
  nfc_crc.c \
  nfc_config.c \
  nfc_core.c \
  nfc_host.c \
  nfc_host_app.c \
  nfc_host_service.c \
  nfc_initiator.c \
  nfc_llc.c \
  nfc_llc_io.c \
  nfc_llc_io_initiator.c \
  nfc_llc_io_target.c \
  nfc_llc_param.c \
  nfc_manager.c \
  nfc_ndef.c \
  nfc_peer.c \
  nfc_peer_connection.c \
  nfc_peer_initiator.c \
  nfc_peer_service.c \
  nfc_peer_services.c \
  nfc_peer_socket.c \
  nfc_peer_target.c \
  nfc_plugins.c \
  nfc_plugin.c \
  nfc_snep_server.c \
  nfc_tag.c \
  nfc_tag_t2.c \
  nfc_tag_t4.c \
  nfc_tag_t4a.c \
  nfc_tag_t4b.c \
  nfc_target.c \
  nfc_util.c

#
# Directories
#

SRC_DIR = src
INCLUDE_DIR = include
BUILD_DIR = build
DEBUG_BUILD_DIR = $(BUILD_DIR)/debug
RELEASE_BUILD_DIR = $(BUILD_DIR)/release
COVERAGE_BUILD_DIR = $(BUILD_DIR)/coverage

#
# Pull nfcd version from nfc_version.h file
#

VERSION_FILE = $(INCLUDE_DIR)/nfc_version.h
get_version = $(shell grep -E '^ *\#define +NFC_VERSION_$1 +[0-9]+$$' $(VERSION_FILE) | sed 's/  */ /g' | cut -d ' ' -f 3)

VERSION_MAJOR = $(call get_version,MAJOR)
VERSION_MINOR = $(call get_version,MINOR)
VERSION_RELEASE = $(call get_version,RELEASE)

# Version for pkg-config
PCVERSION = $(VERSION_MAJOR).$(VERSION_MINOR).$(VERSION_RELEASE)

#
# Tools and flags
#

CC = $(CROSS_COMPILE)gcc
LD = $(CC)
WARNINGS = -Wall -Wstrict-aliasing -Wunused-result
INCLUDES = -I$(INCLUDE_DIR)
BASE_FLAGS = -fPIC
FULL_CFLAGS = $(BASE_FLAGS) $(CFLAGS) $(DEFINES) $(WARNINGS) $(INCLUDES) \
  -DGLIB_VERSION_MAX_ALLOWED=GLIB_VERSION_2_32 \
  -DGLIB_VERSION_MIN_REQUIRED=GLIB_VERSION_MAX_ALLOWED \
  -MMD -MP $(shell pkg-config --cflags $(PKGS))
DEBUG_FLAGS = -g
RELEASE_FLAGS =
COVERAGE_FLAGS = -g

KEEP_SYMBOLS ?= 0
ifneq ($(KEEP_SYMBOLS),0)
RELEASE_FLAGS += -g
endif

DEBUG_CFLAGS = $(FULL_CFLAGS) $(DEBUG_FLAGS) -DDEBUG
RELEASE_CFLAGS = $(FULL_CFLAGS) $(RELEASE_FLAGS) -O2
COVERAGE_CFLAGS = $(FULL_CFLAGS) $(COVERAGE_FLAGS) --coverage

#
# Files
#

PKGCONFIG_NAME = nfcd-plugin.pc
PKGCONFIG_TEMPLATE = $(PKGCONFIG_NAME).in
PKGCONFIG = $(BUILD_DIR)/$(PKGCONFIG_NAME)
DEBUG_OBJS = $(SRC:%.c=$(DEBUG_BUILD_DIR)/%.o)
RELEASE_OBJS = $(SRC:%.c=$(RELEASE_BUILD_DIR)/%.o)
COVERAGE_OBJS = $(SRC:%.c=$(COVERAGE_BUILD_DIR)/%.o)


#
# Dependencies
#

DEPS = $(DEBUG_OBJS:%.o=%.d) $(RELEASE_OBJS:%.o=%.d)
ifneq ($(MAKECMDGOALS),clean)
ifneq ($(strip $(DEPS)),)
-include $(DEPS)
endif
endif

$(PKGCONFIG): | $(BUILD_DIR)
$(DEBUG_OBJS) $(DEBUG_LIB): | $(DEBUG_BUILD_DIR)
$(RELEASE_OBJS) $(RELEASE_LIB): | $(RELEASE_BUILD_DIR)
$(COVERAGE_OBJS) $(COVERAGE_LIB): | $(COVERAGE_BUILD_DIR)

#
# Rules
#

DEBUG_LIB = $(DEBUG_BUILD_DIR)/$(LIB)
RELEASE_LIB = $(RELEASE_BUILD_DIR)/$(LIB)
COVERAGE_LIB = $(COVERAGE_BUILD_DIR)/$(LIB)

debug: $(DEBUG_LIB) $(PKGCONFIG)

release: $(RELEASE_LIB) $(PKGCONFIG)

debug_lib: $(DEBUG_LIB)

release_lib: $(RELEASE_LIB)

coverage_lib: $(COVERAGE_LIB)

pkgconfig: $(PKGCONFIG)

clean:
	rm -f *~ $(SRC_DIR)/*~ $(INCLUDE_DIR)/*~ $(INCLUDE_DIR)/internal/*~
	rm -fr $(BUILD_DIR)

$(BUILD_DIR):
	mkdir -p $@

$(DEBUG_BUILD_DIR):
	mkdir -p $@

$(RELEASE_BUILD_DIR):
	mkdir -p $@

$(COVERAGE_BUILD_DIR):
	mkdir -p $@

$(DEBUG_BUILD_DIR)/%.o : $(SRC_DIR)/%.c
	$(CC) -c $(DEBUG_CFLAGS) -MT"$@" -MF"$(@:%.o=%.d)" $< -o $@

$(RELEASE_BUILD_DIR)/%.o : $(SRC_DIR)/%.c
	$(CC) -c $(RELEASE_CFLAGS) -MT"$@" -MF"$(@:%.o=%.d)" $< -o $@

$(COVERAGE_BUILD_DIR)/%.o : $(SRC_DIR)/%.c
	$(CC) -c $(COVERAGE_CFLAGS) -MT"$@" -MF"$(@:%.o=%.d)" $< -o $@

$(DEBUG_LIB): $(DEBUG_OBJS)
	$(AR) rc $@ $?
	ranlib $@

$(RELEASE_LIB): $(RELEASE_OBJS)
	$(AR) rc $@ $?
	ranlib $@

$(COVERAGE_LIB): $(COVERAGE_OBJS)
	$(AR) rc $@ $?
	ranlib $@

$(PKGCONFIG): $(PKGCONFIG_TEMPLATE) $(VERSION_FILE)
	sed -e 's|@version@|$(PCVERSION)|g' $(PKGCONFIG_TEMPLATE) > $@

#
# Install
#

LIBDIR ?= $(shell pkg-config --variable=libdir glib-2.0)
ABS_LIBDIR := $(shell echo /$(LIBDIR) | sed -r 's|/+|/|g')

INSTALL = install
INSTALL_DIRS = $(INSTALL) -d
INSTALL_FILES = $(INSTALL) -m 644

INSTALL_INCLUDE_DIR = $(DESTDIR)/usr/include/nfcd
INSTALL_PKGCONFIG_DIR = $(DESTDIR)$(ABS_LIBDIR)/pkgconfig

install: $(INSTALL_INCLUDE_DIR) $(INSTALL_PKGCONFIG_DIR)
	$(INSTALL_FILES) $(INCLUDE_DIR)/*.h $(INSTALL_INCLUDE_DIR)
	$(INSTALL_FILES) $(PKGCONFIG) $(INSTALL_PKGCONFIG_DIR)

$(INSTALL_INCLUDE_DIR):
	$(INSTALL_DIRS) $@

$(INSTALL_PKGCONFIG_DIR):
	$(INSTALL_DIRS) $@
