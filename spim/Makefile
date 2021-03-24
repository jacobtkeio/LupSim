#
# SPIM S20 MIPS Simulator.
# Makefile for SPIM.
#
# SPIM is covered by a BSD license.
#
# Copyright (c) 1990-2020, James R. Larus.
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without modification,
# are permitted provided that the following conditions are met:
#
# Redistributions of source code must retain the above copyright notice,
# this list of conditions and the following disclaimer.
#
# Redistributions in binary form must reproduce the above copyright notice,
# this list of conditions and the following disclaimer in the documentation and/or
# other materials provided with the distribution.
#
# Neither the name of the James R. Larus nor the names of its contributors may be
# used to endorse or promote products derived from this software without specific
# prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
# LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE
# GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
# HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
# LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY
# OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#

# Avoid builtin rules and variables
MAKEFLAGS += -rR

# =================== BUILD PARAMETERS ==================== #
# Path for directory that contains the source of the CPU code:
CPU_DIR := ../CPU
VPATH = src:$(CPU_DIR)

# Path of directory that contains SPIM tests:
TEST_DIR := ../Tests

# Path of directory that contains documentation:
DOC_DIR := ../Documentation

# Full path for the parent directory of all installed files:
PREFIX := /usr

# Full path for the directory that will hold the executable files:
BIN_DIR := $(PREFIX)/bin

# Full path for the directory that will hold the exception handler:
EXCEPTION_DIR := $(PREFIX)/share/spim

# Full path for the directory that will hold the man files:
MAN_DIR := $(PREFIX)/share/man/man1

# SPIM needs flex's -I flag since the scanner is used interactively.
# You can set the -8 flag so that funny characters do not hang the scanner.
LEX := flex
LFLAGS := -I -8 -o lex.yy.c

YACC := bison
YFLAGS := -d --defines=parser_yacc.h --output=parser_yacc.c -p yy


# Size of the segments when spim starts up (data segment must be >= 64K).
# (These sizes are fine for most users since SPIM dynamically expands
# the memory as necessary.)
MEM_SIZES := -DTEXT_SIZE=65536 -DDATA_SIZE=131072 -DK_TEXT_SIZE=65536

# Name of target executable
SPIM := spim

# =================== END BUILD PARAMETERS ==================== #

all: $(SPIM)

CXX 	:= g++
DEFINES := $(MEM_SIZES) -DDEFAULT_EXCEPTION_HANDLER="\"$(EXCEPTION_DIR)/exceptions.s\""
CPPFLAGS := -I. -I$(CPU_DIR) $(DEFINES) -O -g -Wall -pedantic -Wextra -Wunused -Wno-write-strings -x c++ -MMD
LDFLAGS := -lm
CSH = bash

# Objects
OBJS := \
    spim.o \
    parser_yacc.o \
    lex.yy.o \
    $(CPU_DIR)/spim-utils.o \
    $(CPU_DIR)/run.o \
    $(CPU_DIR)/mem.o \
    $(CPU_DIR)/inst.o \
    $(CPU_DIR)/data.o \
    $(CPU_DIR)/sym-tbl.o \
    $(CPU_DIR)/syscall.o \
    $(CPU_DIR)/display-utils.o \
    $(CPU_DIR)/string-stream.o

# Dependency mangement
DEPS := $(patsubst %.o,%.d,$(OBJS))
-include $(DEPS)

# The SPIM application requires the parser to have been generated
# This dependency cannot be automatically deduced with `-MMD`
spim.o: parser_yacc.h


# Don't print the commands unless explicitly requested with `make V=1`
ifneq ($(V),1)
Q = @
endif

# Main rule for building target executable
$(SPIM): $(OBJS)
	@echo "LD	 $@"
	$(Q)$(CXX) $(OBJS) $(LDFLAGS) -o $@

# Generic rule for compiling C files
%.o: %.c
	@echo "CC	 $@"
	$(Q)$(CXX) $(CPPFLAGS) -c -o $@ $<


# Generic rule for compiling C++ files
%.o: %.cpp
	@echo "CXX	 $@"
	$(Q)$(CXX) $(CPPFLAGS) -c -o $@ $<

# Specific rules to generate parser and lexer
parser_yacc.h: parser_yacc.c

parser_yacc.c: $(CPU_DIR)/parser.y
	@echo "YACC	 $@"
	$(Q)$(YACC) $(YFLAGS) $(CPU_DIR)/parser.y

lex.yy.c: $(CPU_DIR)/scanner.l
	@echo "LEX	 $@"
	$(Q)$(LEX) $(LFLAGS) $(CPU_DIR)/scanner.l

.PHONY: clean install install-man splint
clean:
	@echo "CLEAN	 ."
	$(Q)rm -f $(SPIM) $(OBJS) $(DEPS) parser_yacc.* lex.yy.* tags

install: spim
	$(Q)install -d $(DESTDIR)$(BIN_DIR)
	$(Q)install spim $(DESTDIR)$(BIN_DIR)/spim
	$(Q)install -d $(DESTDIR)$(EXCEPTION_DIR)
	$(Q)install -m 0444 $(CPU_DIR)/exceptions.s $(DESTDIR)$(EXCEPTION_DIR)/exceptions.s

install-man:
	$(Q)install -d $(DESTDIR)$(MAN_DIR)
	$(Q)install -m 0444 $(DOC_DIR)/spim.man $(DESTDIR)$(MAN_DIR)/spim.1

# Tests
test:	spim
	@echo
	@echo "Testing tt.bare.s:"
	$(CSH) -c "./spim -delayed_branches -delayed_loads -noexception -file $(TEST_DIR)/tt.bare.s >& test.out"
	@tail -2 test.out
	@echo

	@echo
	@echo "Testing tt.core.s:"
	$(Q)$(CSH) -c "./spim -ef $(CPU_DIR)/exceptions.s -file $(TEST_DIR)/tt.core.s < $(TEST_DIR)/tt.in >& test.out"
	@tail -2 test.out
	@echo

	@echo
	@echo "Testing tt.le.s:"
	$(Q)$(CSH) -c "./spim -ef $(CPU_DIR)/exceptions.s -file $(TEST_DIR)/tt.le.s  >& test.out"
	@tail -2 test.out
	@echo
	@echo

# This test currently only works for little-endian machines.  The file
# tt.alu.bare.s needs to be converted in places for big-endian machines.

test_bare: spim
	@echo
	@echo "Testing tt.alu.bare.s:"
	$(Q)$(CSH) -c "./spim -bare -noexception -file $(TEST_DIR)/tt.alu.bare.s >& test.out"
	@tail -2 test.out
	@echo

	@echo
	@echo "Testing tt.fpt.bare.s:"
	$(Q)$(CSH) -c "./spim -bare -noexception -file $(TEST_DIR)/tt.fpu.bare.s >& test.out"
	@tail -2 test.out
	@echo
	@echo

# Generate an index of the source code for editor integration
tags: *.cpp *.h
	ctags --append=yes $?
