PROJECT = kazoo_documents
ROOT = ../..

EBINS = $(shell find $(ROOT)/core/whistle-* -maxdepth 2 -name ebin -print) \
	$(shell find $(ROOT)/applications/crossbar -maxdepth 2 -name ebin -print) \
	$(shell find $(ROOT)/core/whistle_couch-* -maxdepth 2 -name ebin -print) \
	$(shell find $(ROOT)/deps/cowboy-* -maxdepth 2 -name ebin -print) \
	$(shell find $(ROOT)/deps/ejson-* -maxdepth 2 -name ebin -print) \
	$(shell find $(ROOT)/deps/jesse-* -maxdepth 2 -name ebin -print) \
	$(shell find $(ROOT)/deps/lager-* -maxdepth 2 -name ebin -print) \
	$(shell find $(ROOT)/core/kazoo_documents-* -maxdepth 2 -name ebin -print)
PA = $(foreach EBIN,$(EBINS),-pa $(EBIN))

ERLC_OPTS = -Werror +debug_info +warn_export_all -pa ebin $(PA)

.PHONY: all compile clean

all: compile

MODULES = $(shell ls src/*.erl | sed 's/src\///;s/\.erl/,/' | sed '$$s/.$$//')
TEST_MODULES = $(shell ls test/*.erl | sed 's/test\///;s/\.erl/,/' | sed '$$s/.$$//')

compile: ebin/$(PROJECT).app json
	@cat src/$(PROJECT).app.src \
		| sed 's/{modules, \[\]}/{modules, \[$(MODULES)\]}/' \
		> ebin/$(PROJECT).app
	-@$(MAKE) ebin/$(PROJECT).app

ebin/$(PROJECT).app: src/*.erl
	@mkdir -p ebin/
	erlc -v $(ERLC_OPTS) -o ebin/ -pa ebin/ $(PA) $?

json: JSON = $(shell find priv/ -name *.json -print)
json:
	@for jobj in $(JSON); do \
		echo checking $$jobj; \
		python2 -c 'import sys,json; json.encoder.FLOAT_REPR=str; data=json.load(open("'$$jobj'")); print json.dumps(data, sort_keys=True, indent=4, separators=(",", ": "))' >$$jobj~ && mv $$jobj~ $$jobj; \
	done

compile-test: test/$(PROJECT).app
	@cat src/$(PROJECT).app.src \
		| sed 's/{modules, \[\]}/{modules, \[$(MODULES),$(TEST_MODULES)\]}/' \
		> test/$(PROJECT).app
	-@$(MAKE) test/$(PROJECT).app

test/$(PROJECT).app: src/*.erl test/*.erl
	@mkdir -p test/
	erlc -v $(ERLC_OPTS) -DTEST -o test/ -pa test/ $(PA) $?

clean:
	rm -f ebin/*
	rm -f erl_crash.dump

clean-test:
	rm -f test/*.beam test/$(PROJECT).app

test: clean-test compile-test eunit

eunit: compile-test
	erl -noshell $(PA) -pa test -eval "case eunit:test([$(TEST_MODULES),$(MODULES)], [verbose]) of 'ok' -> init:stop(); _ -> init:stop(1) end."
