.DEFAULT_GOAL:=all

ROOT_CA_IMAGE:=../root-ca/mkosi.output/root-ca.raw
PASS_ROOT_KEY:=$(shell cat ../passwords/pass-root-key)

CA_IMAGE:=mkosi.output/ca.raw
CA_DOMAIN:=example.org
PASS_CA_KEY:=$(shell cat ../passwords/pass-ca-key)
PASS_CA_JWK:=$(shell cat ../passwords/pass-ca-jwk)

OUTPUTS:=$(CA_IMAGE) mkosi.output/ssh_user_ca_key.pub mkosi.output/ssh_host_ca_key.pub mkosi.output/intermediate_ca.crt mkosi.output/root_ca.crt mkosi.output/known_hosts

.PHONY: clean
clean:
	sudo mkosi clean
	rm -f mkosi.output/*id_ecdsa* $(OUTPUTS)

.PHONY: all
all: export CA_IPv4?=10.0.100.3
all: export CA_IPv6?=fc64::3
all: export CA_NAME?=ca
all: export CA_DOMAIN?=$(CA_DOMAIN)
all: export PASS_CA_KEY?=$(PASS_CA_KEY)
all: export PASS_CA_JWK?=$(PASS_CA_JWK)
all: export PASS_ROOT_KEY?=$(PASS_ROOT_KEY)
all: export ROOT_CA_IMAGE?=$(ROOT_CA_IMAGE)
all: $(OUTPUTS)

$(OUTPUTS) &: $(ROOT_CA_IMAGE)
	mkdir -p mkosi.output
	sudo --preserve-env=CA_IPv4,CA_IPv6,CA_NAME,CA_DOMAIN,PASS_CA_KEY,PASS_CA_JWK,PASS_ROOT_KEY,ROOT_CA_IMAGE mkosi -i

$(ROOT_CA_IMAGE):
	make -C ../root-ca

SSH_USER_OUTPUTS:=mkosi.output/%_id_ecdsa mkosi.output/%_id_ecdsa.pub mkosi.output/%_id_ecdsa-cert.pub
$(SSH_USER_OUTPUTS) &: export PASS_CA_KEY?=$(PASS_CA_KEY)
$(SSH_USER_OUTPUTS) &: export PASS_CA_JWK?=$(PASS_CA_JWK)
$(SSH_USER_OUTPUTS) &: export NOT_AFTER?=24h
$(SSH_USER_OUTPUTS) &: $(CA_IMAGE)
	@echo "You'll need to enter this manually 3x: ${PASS_CA_KEY}"
	sudo systemd-nspawn \
	  --set-credential=jwkpass:"$$PASS_CA_JWK" \
	  --setenv=NOT_AFTER="$$NOT_AFTER" \
	  --setenv=PRINCIPAL="$$(grep -oP "mkosi.output/\K.+(?=_id_ecdsa(|\.pub|-cert\.pub))" <<<'$@')" \
	  --setenv=STEPPATH=/var/lib/step-ca \
	  --private-users=identity \
	  --bind="$$PWD/mkosi.output":/mnt \
	  --image=mkosi.output/ca.raw \
	  --as-pid2 sh -c \
	  'step-cli ssh certificate "$$PRINCIPAL" /mnt/"$${PRINCIPAL}"_id_ecdsa \
	     --insecure --no-password --offline --force \
	     --not-after "$${NOT_AFTER}" \
	     --provisioner jwk \
	     --provisioner-password-file "$${CREDENTIALS_DIRECTORY}"/jwkpass'
	@sudo chown $$(stat -c "%u:%g" mkosi.output) mkosi.output/*
