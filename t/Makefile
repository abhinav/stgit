# Run tests
#
# Copyright (c) 2005 Junio C Hamano
#

PYTHON ?= python3
# STG_TEST_OPTS = --verbose-log
# STG_PROVE_OPTS = --jobs=9
STG_PROVE_OPTS ?=
SHELL_PATH ?= $(SHELL)
TEST_SHELL_PATH ?= $(SHELL_PATH)
PERL_PATH ?= /usr/bin/perl
TAR ?= $(TAR)
RM ?= rm -f
PROVE ?= prove
DEFAULT_TEST_TARGET ?= test
TEST_LINT ?= test-lint

ifdef TEST_OUTPUT_DIRECTORY
TEST_RESULTS_DIRECTORY = $(TEST_OUTPUT_DIRECTORY)/test-results
else
TEST_RESULTS_DIRECTORY = test-results
endif

# Shell quote;
SHELL_PATH_SQ = $(subst ','\'',$(SHELL_PATH))
TEST_SHELL_PATH_SQ = $(subst ','\'',$(TEST_SHELL_PATH))
PERL_PATH_SQ = $(subst ','\'',$(PERL_PATH))
TEST_RESULTS_DIRECTORY_SQ = $(subst ','\'',$(TEST_RESULTS_DIRECTORY))

T = $(sort $(wildcard t[0-9][0-9][0-9][0-9]-*.sh))
THELPERS = $(sort $(filter-out $(T),$(wildcard *.sh)))

all: $(DEFAULT_TEST_TARGET)

test: pre-clean $(TEST_LINT)
	$(MAKE) aggregate-results-and-cleanup

failed:
	@failed=$$(cd '$(TEST_RESULTS_DIRECTORY_SQ)' && \
		grep -l '^failed [1-9]' *.counts | \
		sed -n 's/\.counts$$/.sh/p') && \
	test -z "$$failed" || $(MAKE) $$failed

prove: pre-clean $(TEST_LINT)
	@echo "*** prove ***"; $(PROVE) --exec '$(TEST_SHELL_PATH_SQ)' $(STG_PROVE_OPTS) $(T) :: $(STG_TEST_OPTS)
	$(MAKE) clean-except-prove-cache

$(T):
	@echo "*** $@ ***"; '$(TEST_SHELL_PATH_SQ)' $@ $(STG_TEST_OPTS)

pre-clean:
	$(RM) -r '$(TEST_RESULTS_DIRECTORY_SQ)'

clean-except-prove-cache:
	$(RM) -r 'trash directory'.* '$(TEST_RESULTS_DIRECTORY_SQ)'

clean: clean-except-prove-cache
	$(RM) .prove

test-lint: test-lint-duplicates test-lint-executable test-lint-shell-syntax \
	test-lint-filenames

test-lint-duplicates:
	@dups=`echo $(T) | tr ' ' '\n' | sed 's/-.*//' | sort | uniq -d` && \
		test -z "$$dups" || { \
		echo >&2 "duplicate test numbers:" $$dups; exit 1; }

test-lint-executable:
	@bad=`for i in $(T); do test -x "$$i" || echo $$i; done` && \
		test -z "$$bad" || { \
		echo >&2 "non-executable tests:" $$bad; exit 1; }

test-lint-shell-syntax:
	@'$(PERL_PATH_SQ)' check-non-portable-shell.pl $(T) $(THELPERS)

test-lint-filenames:
	@# We do *not* pass a glob to ls-files but use grep instead, to catch
	@# non-ASCII characters (which are quoted within double-quotes)
	@bad="$$(git -c core.quotepath=true ls-files 2>/dev/null | \
			grep '["*:<>?\\|]')"; \
		test -z "$$bad" || { \
		echo >&2 "non-portable file name(s): $$bad"; exit 1; }

aggregate-results-and-cleanup: $(T)
	$(MAKE) aggregate-results
	$(MAKE) clean

aggregate-results:
	for f in '$(TEST_RESULTS_DIRECTORY_SQ)'/t*-*.counts; do \
		echo "$$f"; \
	done | '$(SHELL_PATH_SQ)' ./aggregate-results.sh

.PHONY: pre-clean $(T) aggregate-results clean
