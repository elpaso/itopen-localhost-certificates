
# Test certificates for localhost, user: itopen

Create the certificates:

    # Generate the server private key
    openssl genrsa -des3 -out server.key 1024
    # Remove the passphrase
    openssl rsa -in server.key -out server.key
    chmod 400 server.key
    # Create and sign the root certificate for localhost CN=localhost
    openssl req -new -key server.key -days 3650 -out server.crt -x509 -subj '/C=CA/ST=Italy/L=LusernaSG/O=ItOpen.it/CN=localhost/emailAddress=info@itopen.it'
    # Copy into root because it's self signed
    cp server.crt root.crt
    # Create the client key
    openssl genrsa -des3 -out /tmp/postgresql.key 1024
    openssl rsa -in /tmp/postgresql.key -out /tmp/postgresql.key
    # Create the certificate signing request for user itopen (CN=itopen)
    openssl req -new -key /tmp/postgresql.key -out /tmp/postgresql.csr -subj '/C=CA/ST=Italy/L=LusernaSG/O=ItOpen.it/CN=itopen'
    # Create and sign the client certificates
    openssl x509 -req -in /tmp/postgresql.csr -CA root.crt -CAkey server.key -out /tmp/postgresql.crt -CAcreateserial


Note: all certificates must have permission 0400!

Client certificates: place in `~/.postgresql`

    -r--------   1 ale ale  1606 ott 21 13:00 postgresql.crt
    -r--------   1 ale ale   887 ott 21 13:00 postgresql.key
    -r--------   1 ale ale  8155 ott 21 13:00 root.crt

Server certificates:

    -r-------- 1 ale ale 1013 ott 20 09:55 root.crt
    -r-------- 1 ale ale   17 ott 20 09:58 root.srl
    -r-------- 1 ale ale 1013 ott 20 09:55 server.crt
    -r-------- 1 ale ale  891 ott 20 09:54 server.key


place the certificates wherever you like (example: `/pg/certs/`)
and in `postgresql.conf`, set the following:

    ssl = true
    ssl_ciphers = 'DEFAULT:!LOW:!EXP:!MD5:@STRENGTH'    # allowed SSL ciphers

    ssl_cert_file = '/pg/certs/server.crt'
    ssl_key_file = '/pg/certs/server.key'
    ssl_ca_file = '/pg/certs/root.crt'
    password_encryption = on

Run the server as a normal user with:

    postgres  -D /path_to/data_dir -c config_file=/path_to/postgresql.conf

You can access the DB on port 5432 with:

    psql "sslmode=verify-full host=localhost port=5432 user=itopen"
