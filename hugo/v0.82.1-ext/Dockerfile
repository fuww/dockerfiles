
FROM google/cloud-sdk:latest

ENV HUGO_VERSION 0.82.1
ENV NODE_VERSION 14.15.4

RUN ARCH= && dpkgArch="$(dpkg --print-architecture)" \
    && case "${dpkgArch##*-}" in \
    amd64) ARCH='x64';; \
    ppc64el) ARCH='ppc64le';; \
    s390x) ARCH='s390x';; \
    arm64) ARCH='arm64';; \
    armhf) ARCH='armv7l';; \
    i386) ARCH='x86';; \
    *) echo "unsupported architecture"; exit 1 ;; \
    esac \
    # gpg keys listed at https://github.com/nodejs/node#release-keys
    && set -ex \
    && for key in \
    94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
    FD3A5288F042B6850C66B31F09FE44734EB7990E \
    71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
    DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
    C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
    B9AE9905FFD7803F25714661B63B535A4C206CA9 \
    77984A986EBC2AA786BC0F66B01FBB92821C587A \
    8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
    4ED778F539E3634C779C87C6D7062848A1AB005C \
    A48C2BEE680E841632CD4E44F07496B3EB3C1762 \
    B9E2F5981AA6E0CD28160D9FF13993A75599653C \
    ; do \
    gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
    gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
    gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
    done \
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH.tar.xz" \
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
    && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
    && grep " node-v$NODE_VERSION-linux-$ARCH.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
    && tar -xJf "node-v$NODE_VERSION-linux-$ARCH.tar.xz" -C /usr/local --strip-components=1 --no-same-owner \
    && rm "node-v$NODE_VERSION-linux-$ARCH.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs \
    # smoke tests
    && node --version \
    && npm --version

# gcc for cgo
RUN apt-get update && apt-get install -y --no-install-recommends \
    wget \
    g++ \
    gcc \
    libc6-dev \
    make \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

ENV PATH /usr/local/go/bin:$PATH

ENV GOLANG_VERSION 1.16.3

RUN set -eux; \
    \
    dpkgArch="$(dpkg --print-architecture)"; \
    url=; \
    case "${dpkgArch##*-}" in \
    'amd64') \
    url='https://dl.google.com/go/go1.16.3.linux-amd64.tar.gz'; \
    sha256='951a3c7c6ce4e56ad883f97d9db74d3d6d80d5fec77455c6ada6c1f7ac4776d2'; \
    ;; \
    'armel') \
    export GOARCH='arm' GOARM='5' GOOS='linux'; \
    ;; \
    'armhf') \
    url='https://dl.google.com/go/go1.16.3.linux-armv6l.tar.gz'; \
    sha256='0dae30385e3564a557dac7f12a63eedc73543e6da0f6017990e214ce8cc8797c'; \
    ;; \
    'arm64') \
    url='https://dl.google.com/go/go1.16.3.linux-arm64.tar.gz'; \
    sha256='566b1d6f17d2bc4ad5f81486f0df44f3088c3ed47a3bec4099d8ed9939e90d5d'; \
    ;; \
    'i386') \
    url='https://dl.google.com/go/go1.16.3.linux-386.tar.gz'; \
    sha256='48b2d1481db756c88c18b1f064dbfc3e265ce4a775a23177ca17e25d13a24c5d'; \
    ;; \
    'mips64el') \
    export GOARCH='mips64le' GOOS='linux'; \
    ;; \
    'ppc64el') \
    url='https://dl.google.com/go/go1.16.3.linux-ppc64le.tar.gz'; \
    sha256='5eb046bbbbc7fe2591846a4303884cb5a01abb903e3e61e33459affe7874e811'; \
    ;; \
    's390x') \
    url='https://dl.google.com/go/go1.16.3.linux-s390x.tar.gz'; \
    sha256='3e8bd7bde533a73fd6fa75b5288678ef397e76c198cfb26b8ae086035383b1cf'; \
    ;; \
    *) echo >&2 "error: unsupported architecture '$dpkgArch' (likely packaging update needed)"; exit 1 ;; \
    esac; \
    build=; \
    if [ -z "$url" ]; then \
    # https://github.com/golang/go/issues/38536#issuecomment-616897960
    build=1; \
    url='https://dl.google.com/go/go1.16.3.src.tar.gz'; \
    sha256='b298d29de9236ca47a023e382313bcc2d2eed31dfa706b60a04103ce83a71a25'; \
    echo >&2; \
    echo >&2 "warning: current architecture ($dpkgArch) does not have a corresponding Go binary release; will be building from source"; \
    echo >&2; \
    fi; \
    \
    wget -O go.tgz.asc "$url.asc" --progress=dot:giga; \
    wget -O go.tgz "$url" --progress=dot:giga; \
    echo "$sha256 *go.tgz" | sha256sum --strict --check -; \
    \
    # https://github.com/golang/go/issues/14739#issuecomment-324767697
    export GNUPGHOME="$(mktemp -d)"; \
    # https://www.google.com/linuxrepositories/
    gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys 'EB4C 1BFD 4F04 2F6D DDCC EC91 7721 F63B D38B 4796'; \
    gpg --batch --verify go.tgz.asc go.tgz; \
    gpgconf --kill all; \
    rm -rf "$GNUPGHOME" go.tgz.asc; \
    \
    tar -C /usr/local -xzf go.tgz; \
    rm go.tgz; \
    \
    if [ -n "$build" ]; then \
    savedAptMark="$(apt-mark showmanual)"; \
    apt-get update; \
    apt-get install -y --no-install-recommends golang-go; \
    \
    ( \
    cd /usr/local/go/src; \
    # set GOROOT_BOOTSTRAP + GOHOST* such that we can build Go successfully
    export GOROOT_BOOTSTRAP="$(go env GOROOT)" GOHOSTOS="$GOOS" GOHOSTARCH="$GOARCH"; \
    ./make.bash; \
    ); \
    \
    apt-mark auto '.*' > /dev/null; \
    apt-mark manual $savedAptMark > /dev/null; \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    rm -rf /var/lib/apt/lists/*; \
    \
    # pre-compile the standard library, just like the official binary release tarballs do
    go install std; \
    # go install: -race is only supported on linux/amd64, linux/ppc64le, linux/arm64, freebsd/amd64, netbsd/amd64, darwin/amd64 and windows/amd64
    #		go install -race std; \
    \
    # remove a few intermediate / bootstrapping files the official binary release tarballs do not contain
    rm -rf \
    /usr/local/go/pkg/*/cmd \
    /usr/local/go/pkg/bootstrap \
    /usr/local/go/pkg/obj \
    /usr/local/go/pkg/tool/*/api \
    /usr/local/go/pkg/tool/*/go_bootstrap \
    /usr/local/go/src/cmd/dist/dist \
    ; \
    fi; \
    \
    go version

ENV GOPATH /go
ENV PATH $GOPATH/bin:$PATH
RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"
# WORKDIR $GOPATH

RUN npm i lerna -g --loglevel notice
RUN npm i firebase-tools -g

RUN (cd /usr/local/bin; curl -qL https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_Linux-64bit.tar.gz | tar xz hugo)

ENV LANG=en_US.UTF-8