PKG_NAME=vinyldns
NAME=terraform-provider-vinyldns
WEBSITE_REPO=github.com/hashicorp/terraform-website
SOURCE=./...
VERSION=0.8.0

all: updatedeps test install

updatedeps:
	go get -u github.com/golang/dep/cmd/dep
	go get -u golang.org/x/tools/cmd/cover
	go get -u github.com/mitchellh/gox
	dep ensure

# NOTE: acceptance tests assume a VinylDNS instance is running on localhost:9000 using the
# technique here: https://github.com/vinyldns/vinyldns/blob/master/bin/docker-up-vinyldns.sh
test:
	go vet
	go test -cover
	VINYLDNS_ACCESS_KEY=okAccessKey VINYLDNS_SECRET_KEY=okSecretKey VINYLDNS_HOST=http://localhost:9000 TF_LOG=DEBUG TF_ACC=1 go test ${SOURCE} -v

cover:
	go test $(TEST) -coverprofile=coverage.out
	go tool cover -html=coverage.out
	rm coverage.out

install:
	dep ensure

build: updatedeps
	export CGO_ENABLED=0; gox -ldflags "-X main.version=${VERSION}" -os "linux darwin windows" -arch "386 amd64" -output "build/{{.OS}}_{{.Arch}}/terraform-provider-vinyldns"

version:
	echo ${VERSION}

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

.PHONY: updatedeps test cover install build version website website-test

release: build test
	go get github.com/aktau/github-release
	github-release release \
		--user vinyldns \
		--repo $(NAME) \
		--tag $(VERSION) \
		--name "$(VERSION)" \
		--description "$(NAME) version $(VERSION)"
