TEST?=$$(go list ./... |grep -v 'vendor')
GOFMT_FILES?=$$(find . -name '*.go' |grep -v vendor)
WEBSITE_REPO=github.com/hashicorp/terraform-website
PKG_NAME=cds
RELEASE_TAG=$$(git describe --abbrev=0 --tags)

default: build

build: fmtcheck all

test: fmtcheck
	go test -i $(TEST) || exit 1
	echo $(TEST) | \
		xargs -t -n4 go test $(TESTARGS) -timeout=30s -parallel=4

testacc: fmtcheck
	TF_ACC=1 go test $(TEST) -v $(TESTARGS) -timeout 120m

vet:
	@echo "go vet ."
	@go vet $$(go list ./... | grep -v vendor/) ; if [ $$? -eq 1 ]; then \
		echo ""; \
		echo "Vet found suspicious constructs. Please check the reported constructs"; \
		echo "and fix them if necessary before submitting the code for review."; \
		exit 1; \
	fi

fmt:
	gofmt -w $(GOFMT_FILES)

fmtcheck:
	@sh -c "'$(CURDIR)/scripts/gofmtcheck.sh'"

test-compile:
	@if [ "$(TEST)" = "./..." ]; then \
		echo "ERROR: Set TEST to a specific package. For example,"; \
		echo "  make test-compile TEST=./$(PKG_NAME)"; \
		exit 1; \
	fi
	go test -c $(TEST) $(TESTARGS)

website:
ifeq (,$(wildcard $(GOPATH)/src/$(WEBSITE_REPO)))
	echo "$(WEBSITE_REPO) not found in your GOPATH (necessary for layouts and assets), get-ting..."
	git clone https://$(WEBSITE_REPO) $(GOPATH)/src/$(WEBSITE_REPO)
endif
	@$(MAKE) -C $(GOPATH)/src/$(WEBSITE_REPO) website-provider PROVIDER_PATH=$(shell pwd) PROVIDER_NAME=$(PKG_NAME)

website-test:
ifeq (,$(wildcard $(GOPATH)/src/$(WEBSITE_REPO)))
	echo "$(WEBSITE_REPO) not found in your GOPATH (necessary for layouts and assets), get-ting..."
	git clone https://$(WEBSITE_REPO) $(GOPATH)/src/$(WEBSITE_REPO)
endif
	@$(MAKE) -C $(GOPATH)/src/$(WEBSITE_REPO) website-provider-test PROVIDER_PATH=$(shell pwd) PROVIDER_NAME=$(PKG_NAME)

.PHONY: build test testacc vet fmt test-compile website website-test

all: mac windows linux

all-with-version: mac-with-version windows-with-version linux-with-version

dev: clean fmt mac copy

devlinux: clean fmt linux linuxcopy

devwin: clean fmt windows windowscopy

copy:
	tar -xvf bin/terraform-provider-cds_darwin-amd64.tgz && mv bin/terraform-provider-cds $(shell dirname `which terraform`)

clean:
	rm -rf bin/*

mac:
	GOOS=darwin GOARCH=amd64 go build -o bin/terraform-provider-cds
	tar czvf bin/terraform-provider-cds_darwin-amd64.tgz bin/terraform-provider-cds
	rm -rf bin/terraform-provider-cds

mac-with-version:
	GOOS=darwin GOARCH=amd64 go build -o bin/terraform-provider-cds_$(RELEASE_TAG)
	tar czvf bin/terraform-provider-cds_darwin-amd64_$(RELEASE_TAG).tgz bin/terraform-provider-cds_$(RELEASE_TAG)
	rm -rf bin/terraform-provider-cds_$(RELEASE_TAG)

windowscopy:
	tar -xvf bin/terraform-provider-cds_windows-amd64.tgz && mv bin/terraform-provider-cds $(shell dirname `which terraform`)

windows:
	GOOS=windows GOARCH=amd64 go build -o bin/terraform-provider-cds.exe
	tar czvf bin/terraform-provider-cds_windows-amd64.tgz bin/terraform-provider-cds.exe
	rm -rf bin/terraform-provider-cds.exe

windows-with-version:
	GOOS=windows GOARCH=amd64 go build -o bin/terraform-provider-cds_$(RELEASE_TAG).exe
	tar czvf bin/terraform-provider-cds_windows-amd64_$(RELEASE_TAG).tgz bin/terraform-provider-cds_$(RELEASE_TAG).exe
	rm -rf bin/terraform-provider-cds_$(RELEASE_TAG).exe

linuxcopy:
	tar -xvf bin/terraform-provider-cds_linux-amd64.tgz && mv bin/terraform-provider-cds $(shell dirname `which terraform`)

linux:
	GOOS=linux GOARCH=amd64 go build -o bin/terraform-provider-cds
	tar czvf bin/terraform-provider-cds_linux-amd64.tgz bin/terraform-provider-cds
	rm -rf bin/terraform-provider-cds

linux-with-version:
	GOOS=linux GOARCH=amd64 go build -o bin/terraform-provider-cds_$(RELEASE_TAG)
	tar czvf bin/terraform-provider-cds_linux-amd64_$(RELEASE_TAG).tgz bin/terraform-provider-cds_$(RELEASE_TAG)
	rm -rf bin/terraform-provider-cds_$(RELEASE_TAG)
