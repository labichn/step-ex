OUTPUT:=mkosi.output/root-ca.raw

.PHONY: all
all: export PASS_ROOT_KEY?=$(shell cat ../passwords/pass-root-key)
all: $(OUTPUT)

.PHONY: clean
clean:
	sudo mkosi clean
	rm -f $(OUTPUT)

$(OUTPUT): 
	mkdir -p mkosi.output
	sudo --preserve-env=PASS_ROOT_KEY mkosi -i
