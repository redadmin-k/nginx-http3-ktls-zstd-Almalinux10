# Nginx with QUIC (HTTP/3), Zstd compression, and KTLS on AlmaLinux 10

This repository provides an Nginx build for AlmaLinux 10 with:
- HTTP/3 (QUIC) via quictls/openssl
- Zstd compression via zstd-nginx-module
- KTLS (Kernel TLS offload) for sendfile-based TLS I/O

---

## Repository Layout

- RPMS/  – Built binary RPM packages
- SRPM/  – Source RPM package
- SPEC/  – Spec file used for building
- logs/  – Build logs from mock
- README.md – Documentation and verification notes

---

## Build Details (OpenSSL / QUIC TLS)

This build uses quictls/openssl (not the system OpenSSL) to enable QUIC/HTTP/3.

- QUIC TLS (quictls/openssl): <TAG_OR_COMMIT>
- OpenSSL as reported by nginx -V ("built with"): <OPENSSL_VERSION_STRING>
- OpenSSL at runtime ("running with", if shown): <OPENSSL_RUNTIME_STRING_OR_SAME>

### How to capture the exact OpenSSL string
nginx -V 2>&1 | grep -E 'built with OpenSSL|running with OpenSSL'

---

## Features

### 1) HTTP/3 (QUIC)

Built with quictls/openssl to enable QUIC and HTTP/3.

Proof: HTTP/3 request in nginx access log:
192.0.2.10 - - [15/Aug/2025:01:20:32 +0000] "GET / HTTP/3" 200 612 "-" "curl/8.9.0" "-"

Verification:
curl -I --http3 https://alma10.redadmin.org/

Expected output example:
HTTP/3 200
server: nginx/1.28.0
alt-svc: h3=":443"; ma=86400

---

### 2) Zstd Compression

Built with zstd-nginx-module to support Content-Encoding: zstd.

Verification:
curl -sS -D- -o /dev/null -H 'Accept-Encoding: zstd' https://alma10.redadmin.org/

Expected output example:
HTTP/1.1 200 OK
Content-Encoding: zstd

---

### 3) KTLS (Kernel TLS)

Linux Kernel TLS is enabled for sendfile-based TLS data transfer.

Proof: strace output on nginx worker process shows sendfile() syscalls:
sendfile(15, 16, [2097152] => [4194304], 2097152) = 2097152
sendfile(15, 16, [4194304] => [6291456], 2097152) = 2097152

This indicates TLS records are being offloaded to the kernel via KTLS (when the kernel and cipher suite support it).

---

## Build Environment

- OS: AlmaLinux 10 (x86_64)
- Build Tool: mock (alma+epel-10-x86_64 config)
- Nginx Version: 1.28.0
- QUIC TLS (quictls/openssl): <TAG_OR_COMMIT>
- OpenSSL (nginx -V "built with"): <OPENSSL_VERSION_STRING>
- Modules Added:
  - https://github.com/quictls/openssl
  - https://github.com/tokers/zstd-nginx-module

---

## License

- nginx – BSD 2-Clause License
- quictls/openssl – Apache License 2.0
- zstd-nginx-module – BSD 2-Clause License

