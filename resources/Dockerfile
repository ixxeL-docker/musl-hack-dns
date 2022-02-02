FROM gcc as builder

RUN git clone "git://git.musl-libc.org/musl" \
    && awk '1;!inserted && /nq\+\+;/{c=2}c && !--c{print "\t\tif (family == AF_UNSPEC) break;"; inserted=1}' /musl/src/network/lookup_name.c > /tmp/file.c \
    && mv /tmp/file.c /musl/src/network/lookup_name.c

WORKDIR /musl

RUN ./configure && make install

FROM alpine

LABEL maintainer="Frederic Spiers <fredspiers@gmail.com>" \
      component="Alpine kube-DNS fix"

ENV TZ="Europe/Paris" \
    IMG_VERSION="1.0.0"

COPY --from=builder /usr/local/musl/lib/libc.so /lib/ld-musl-x86_64.so.1
