## Create Certificate chain and sign certificates using Openssl
./prepare  
cd ca  
openssl genrsa -aes256 -out private/ca.webrtc.key.pem 4096  
openssl req -config openssl_root.cnf -new -x509 -sha512 -extensions v3_ca -key private/ca.webrtc.key.pem -out certs/ca.webrtc.crt.pem -days 3650 -set_serial 0  
  
cd ../intermediate  
openssl req -config openssl_intermediate.cnf -new -newkey rsa:4096 -keyout private/int.webrtc.key.pem -out csr/int.webrtc.csr  
  
cd ../ca  
openssl ca -config openssl_root.cnf -extensions v3_intermediate_ca -days 3650 -notext -md sha512 -in ../intermediate/csr/int.webrtc.csr -out ../intermediate/certs/int.webrtc.crt.pem  
  
cd ../  
cat intermediate/certs/int.webrtc.crt.pem ca/certs/ca.webrtc.crt.pem > intermediate/certs/chain.webrtc.crt.pem  
  
cd server  
openssl req -out csr/server.csr.pem -newkey rsa:2048 -nodes -keyout private/server.key.pem -config openssl_csr_san.cnf  
  
cd ../intermediate  
openssl ca -config openssl_intermediate.cnf -extensions server_cert -days 3750 -notext -md sha512 -in ../server/csr/server.csr.pem -out ../server/certs/server.crt.pem  
cd ../  
  
## Verify the certificate.
Intermediate CA:  
openssl verify -verbose -CAfile ca/certs/ca.webrtc.crt.pem intermediate/certs/int.webrtc.crt.pem  
  
Server CA  
openssl verify -verbose -CAfile intermediate/certs/chain.webrtc.crt.pem server/certs/server.crt.pem  
or  
openssl verify -CAfile ca/certs/ca.webrtc.crt.pem -untrusted intermediate/certs/int.webrtc.crt.pem server/certs/server.crt.pem  
or
openssl verify -verbose -CAfile <(cat intermediate/certs/int.webrtc.crt.pem ca/certs/ca.webrtc.crt.pem) server/certs/server.crt.pem  
  
  
