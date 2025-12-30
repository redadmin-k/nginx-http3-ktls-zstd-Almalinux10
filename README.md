# Nginx with QUIC (HTTP/3), Zstd compression, and KTLS on AlmaLinux 10

This repository provides an Nginx build for AlmaLinux 10 with:
- HTTP/3 (QUIC) via OpenSSL
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

## Build Details (OpenSSL / HTTP/3)

This build uses **OpenSSL 3.5.4** (standard OpenSSL, not quictls) for TLS/QUIC.

- OpenSSL as reported by nginx -V ("built with"): OpenSSL 3.5.4
- OpenSSL at runtime ("running with", if shown): OpenSSL 3.5.4

### How to capture the exact OpenSSL string
```bash
nginx -V 2>&1 | grep -E 'built with OpenSSL|running with OpenSSL'
```

---

## Features

### 1) HTTP/3 (QUIC)

Built with OpenSSL 3.5.4 to enable QUIC and HTTP/3.

Proof: HTTP/3 request in nginx access log:
```text
192.0.2.10 - - [15/Aug/2025:01:20:32 +0000] "GET / HTTP/3" 200 612 "-" "curl/8.9.0" "-"
```

Verification:
```bash
curl -I --http3 https://alma10.redadmin.org/
```

Expected output example:
```text
HTTP/3 200
server: nginx/1.28.1
alt-svc: h3=":443"; ma=86400
```

---

### 2) Zstd Compression

Built with zstd-nginx-module to support `Content-Encoding: zstd`.

Verification:
```bash
curl -sS -D- -o /dev/null -H 'Accept-Encoding: zstd' https://alma10.redadmin.org/
```

Expected output example:
```text
HTTP/1.1 200 OK
Content-Encoding: zstd
```

---

### 3) KTLS (Kernel TLS)

Linux Kernel TLS is enabled for sendfile-based TLS data transfer.

Proof: `strace` output on nginx worker process shows `sendfile()` syscalls:
```text
sendfile(15, 16, [2097152] => [4194304], 2097152) = 2097152
sendfile(15, 16, [4194304] => [6291456], 2097152) = 2097152
```

This indicates TLS records are being offloaded to the kernel via KTLS (when the kernel and cipher suite support it).

---

## Build Environment

- OS: AlmaLinux 10 (x86_64)
- Build Tool: mock (`alma+epel-10-x86_64` config)
- Nginx Version: 1.28.1
- OpenSSL Version: 3.5.4
- Modules Added:
  - https://github.com/tokers/zstd-nginx-module

---

## License

- nginx – BSD 2-Clause License
- OpenSSL – Apache License 2.0
- zstd-nginx-module – BSD 2-Clause License
