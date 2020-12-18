# iPerf3 with embedded TLS/DTLS support via WolfSSL
- Author: Owen Webb (owebb@umich.edu)
- Date: Dec. 2020.

A fork of [iPerf3](https://github.com/esnet/iperf) that integrates Transport Layer Security 
and Datagram Transport Layer Security for all benchmark/payload traffic. To be 
used for benchmarking and research purposes only. Uses [WolfSSL](https://www.wolfssl.com/),
an SSL library specifically tuned for embedded environments.

Loosely based on [Mic92/iperf-3.7](https://github.com/Mic92/iperf-3.7/commit/3ff810a4ab2939454e5c812b4a7218a1cdda2136) 

## Build, Install, Run
Ensure WolfSSL is built with the following `./configure` flags: `--enable-dtls --enable-oldtls --enable-tls10`

    $ sudo apt-get install lib32z1 # iPerf dependency
    $ git clone git@github.com:owenlwebb/iperf.git && cd iperf/
    $ ./configure LIBS=-lwolfssl
    $ make
    $ sudo make install
    $ iperf3 ...

## Usage
`iperf3` will automatically encrypt benchmark traffic with TLS (or DTLS when `-u` is specified) 
when *any* of the following SSL flags are specified:

Note, all GENERAL SSL flags are *optional*, but all SERVER and CLIENT flags 
are *required*.

If specified, the argument to `--ssl-suites-file`
should be a colon-delimited file containing all of the cipher suites supported 
(in order of preference) by the client or server. If left unspecified, `iperf3`
uses the default cipher suite list for the current WolfSSL installation. 
See the "Default WolfSSL Cipher Suite" list at the bottom of this README for the usual
list.

`--ssl-tls-version` and `--ssl-dtls-version` are both straightforward. Both of 
these flags default to `1.2` if unspecified.

#### GENERAL
    --ssl-suites-file <CIPHER_SUITES_FILE>
    --ssl-tls-version [1.0, 1.1, 1.2, 1.3]
    --ssl-dtls-version [1.0, 1.2]
    
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
clients running in UDP mode should use the `-l` flag to specify a blocksize of less than or 
equal to 8092B


## Future Work / TODO
- Consolidate code in `iperf_udp.c` and `iperf_tcp.c` to reduce code duplication
- More graceful error handling (currently calls `exit` upon almost any error)
- Although I haven't tested it, I expect my current implementation may 
be incompatible with bidirectional and/or reverse modes.
- Support more certificate/key file types besides .pem
- More robust argument parsing
    - Prohibit zerocopy mode with TLS/DTLS (incompatible.)
    - Warn users if `-l XB` for X < 8092 is not specified in UDP mode.
    - Check for incompatible or missing TLS/DTLS args.

    
Default WolfSSL Cipher Suites:
- TLS13-AES128-GCM-SHA256
- TLS13-AES256-GCM-SHA384
- TLS13-CHACHA20-POLY1305-SHA256
- DHE-RSA-AES128-SHA
- DHE-RSA-AES256-SHA
- ECDHE-RSA-AES128-SHA
- ECDHE-RSA-AES256-SHA
- ECDHE-ECDSA-AES128-SHA
- ECDHE-ECDSA-AES256-SHA
- DHE-RSA-AES128-SHA256
- DHE-RSA-AES256-SHA256
- DHE-RSA-AES128-GCM-SHA256
- DHE-RSA-AES256-GCM-SHA384
- ECDHE-RSA-AES128-GCM-SHA256
- ECDHE-RSA-AES256-GCM-SHA384
- ECDHE-ECDSA-AES128-GCM-SHA256
- ECDHE-ECDSA-AES256-GCM-SHA384
- ECDHE-RSA-AES128-SHA256
- ECDHE-ECDSA-AES128-SHA256
- ECDHE-RSA-AES256-SHA384
- ECDHE-ECDSA-AES256-SHA384
- ECDHE-RSA-CHACHA20-POLY1305
- ECDHE-ECDSA-CHACHA20-POLY1305
- DHE-RSA-CHACHA20-POLY1305
- ECDHE-RSA-CHACHA20-POLY1305-OLD
- ECDHE-ECDSA-CHACHA20-POLY1305-OLD
- DHE-RSA-CHACHA20-POLY1305-OLD

Other cipher suites can be enabled through WolfSSL `./configure` flags.