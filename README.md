## About

This is fork of nginx web server with my own useful patches and patches found in the wild, which are not approved by main Nginx 

## Added features

### New features

* Kernel TLS support with Linux kernel 4.13+ and patched OpenSSL
* Reading with preadv2() syscall with RWF_NOWAIT to non-blocking send files from page-cache without threads with `aio threads`
* Discard of request body when client is waiting for "100 Continue" and upstream is ready with answer based on headers. Useful in case when fastcgi/uwsgi module processes POST/PUT request and can make result (redirect or deny) parsing only request headers.

### New variables

* `ssl_handshake_time` variable to log time spent in SSL Handshake
* `ssl_ecdhe_curve` variable to log curve used in SSL Handshake
* `tcpinfo_lost` variable to log TCP segments lost count
* `tcpinfo_retrans` variable to log TCP retransmition count
* `ssl_ktls_status` variable to log usage of Kernel TLS

### New options

* TLS Dynamic Record Size suggested by CloudFlare in https://blog.cloudflare.com/optimizing-tls-over-tcp-to-reduce-latency/
* `ssl_prefer_chacha (on|off)` option to prefer ChaCha-Poly cipher in OpenSSL 1.1.1
* `capability_netadmin` option of `listen` directive to preserve CAP_NET_ADMIN for workers
* `tcp_congestion=(congestion_proto)` option of `listen` directive to change congestion control protocol of listen socket leaving system wide value unchanged and   thats why used to set up connections in upstream module
* `ssl_ktls (on|off)` option to enable or disable Kernel TLS

### Optimizations

* MD5 cache key is calculated with builtin bswap32/bswap64
* SO_INCOMING_CPU option is set to make better CPU locality. Used with `worker_affinity`. Work only with kernels 4.4 - 4.6
* TCP options variables fill speed up. Uses only one syscall per request instead of one syscall per variable
* Speedup linux AIO eliminating syscall as made in libevent library
* SSL_sendfile is used when Kernel TLS configured to eliminated memory copy with SSL

### Fixes

* Fix "header too long" error in upstream module
* Fix possible worker stall on socket close with Kernel TLS

## Kernel TLS

This feature makes it possible to use SSL sockets as normal sockets with all available syscalls. The kernel is resposible to do all needed crypto modifications.
To use this nginx must be compiled againts OpenSSL library compiled with `enable-ktls` option. As of now only `master` (development) branch of OpenSSL repository contains code for Kernel TLS support made by Boris Pismenny from Mellanox. Commited code supports Kernel TLS only for TLS 1.2 connections with AES128-GCM ciphers. With kernel 5.1+ the list of supported ciphers and TLS protocols is extended including TLS1.3 protocol and AES128-CCM and AES256-GCM ciphers. My pull request to add the support of this extensions is still under review but it is available in my fork of OpenSSL on github. I have also backported all Kernel TLS code to OpenSSL 1.1.1 (branch OpenSSL_1_1_1-ktls) for testing with stable version of this SSL library.

To compile nginx against openssl with kernel TLS support add `--with-openssl=<OPENSSL_DIR> --with-openssl-opt="enable-ktls"` options to configure script and nginx will be statically linked to new library build.
