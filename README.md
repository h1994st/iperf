# iPerf3 with embedded TLS/DTLS support via WolfSSL
A fork of [iPerf3](https://github.com/esnet/iperf) that integrates Transport Layer Security 
and Datagram Transport Layer Security for all benchmark/payload traffic. To be 
used for benchmarking and research purposes only. Uses [WolfSSL](https://www.wolfssl.com/),
an SSL library specifically tuned for embedded environments.

## Usage
`iperf3` will automatically encrypt benchmark traffic with TLS (or DTLS when `-u` is specified) 
if any of the SSL options (see below) are specified. 

#### GENERAL
    --ssl-suites-file <CIPHER_SUITES_FILE>
    --ssl-tls-version [1.0, 1.1, 1.2, 1.3]
    --ssl-dtls-version [1.0, 1.1]
    
#### SERVER
    --ssl-server-key <KEY_FILE>.pem
    --ssl-server-cert <CERT_FILE>.pem

#### CLIENT
    --ssl-client-cert <CERT_FILE>.pem

## Note on DTLS Usage
WolfSSL's DTLS implementation imposes a maximum datagram size of approximately 
8KB. It's unclear (to me) whether this restriction is implemenation-specific or defined 
in the DTLS RFC. In any case, testing reveals that WolfSSL will refuse to send any DTLS traffic when 
`iperf3`'s blocksize is set any higher than 8092 bytes. Until this is resolved,
users should use `iperf3`'s `-l` option to specify a blocksize of less than or 
equal to 8092B whenever the `iperf3` client is run in UDP mode.

## Future Work / TODO
- Consolidate code in `iperf_udp.c` and `iperf_tcp.c` to reduce code duplication
- Add argument checking to prohibit zerocopy mode with SSL (incompatible.)
- Add argument checking to warn users if `-l XB` for X < 8092 is not specified in UDP mode.

    
Default WolfSSL Cipher Suites
