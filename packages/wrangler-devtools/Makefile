ROOT = $(realpath .)
# The upstream devtools commit upon which our patches are based
DEVTOOLS_HEAD = f931aec3eca7c860dc4d657f328daca19d19221d
DEVTOOLS_PATCHES = $(shell ls ${PWD}/patches/*.patch)

# A stable depot_tools commit
DEPOT_TOOLS_HEAD = 95a28c5a14cf0c7040794e10e9404b270784010c

depot:
	git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git depot
	git -C depot checkout $(DEPOT_TOOLS_HEAD)

devtools-frontend: depot
	git clone https://chromium.googlesource.com/devtools/devtools-frontend
	$(ROOT)/depot/gclient config https://chromium.googlesource.com/devtools/devtools-frontend --unmanaged
	git -C devtools-frontend checkout $(DEVTOOLS_HEAD)
	git -C devtools-frontend config user.email "workers-devprod@cloudflare.com"
	git -C devtools-frontend config user.name "Workers DevProd"
	git -C devtools-frontend am $(DEVTOOLS_PATCHES)

devtools-frontend/out/Default/gen/front_end: devtools-frontend
	cd devtools-frontend && $(ROOT)/depot/gclient sync
	cd devtools-frontend && $(ROOT)/depot/gn gen out/Default
	cd devtools-frontend && $(ROOT)/depot/autoninja -C out/Default

node_modules:
	npm ci

publish: node_modules devtools-frontend/out/Default/gen/front_end
	npx wrangler pages deploy --project-name cloudflare-devtools devtools-frontend/out/Default/gen/front_end

cleanup:
	rm -rf devtools-frontend .gclient* .cipd node_modules depot

test:
	git -C devtools-frontend am $(DEVTOOLS_PATCHES)
