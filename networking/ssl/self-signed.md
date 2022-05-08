# Self-signed SSL certificates cheat sheet

## Generate new Certificate Authority (CA)
1. Create new folder for CA files
```bash
mkdir ca && cd $_
```
2. Generate CA's RSA key
```bash
openssl genrsa -aes256 -out key.pem
```
3. Generate CA's Public Certificate
```bash
# Change 10950 (30 years) to set desired expiration date 
openssl req -new -x509 -sha256 -days 10950 -key ckey.pem -out cert.pem
```

## Generate new Certificate
1. Create new folder for Certificate files
```bash
cd ..
mkdir cert && cd $_
```
2. Create Certificate's RSA key
```bash
openssl genrsa -out cert-key.pem 4096
```
3. Generate a Certificate Signing Request (CSR)
```bash
# Change yourcn to the YOUR NAME field of CA
openssl req -new -sha256 -subj "/CN=yourcn" -key cert-key.pem -out cert.csr
```
4. Create `extfile.cnf` with all alternative domains/IPs
```bash
echo "subjectAltName=DNS:domain.name.local,IP:127.0.0.1" >> extfile.cnf
```
```bash
# Optional (this option is enabled by default)
echo extendedKeyUsage = serverAuth >> extfile.cnf
```
5. Create the certificate
```bash
# Change 365 (1 year) to set desired expiration date
openssl x509 -req -sha256 -days 365 -in cert.csr -CA ../ca/cert.pem -CAkey ../ca/key.pem -out cert.pem -extfile extfile.cnf -CAcreateserial
```
6. Create `chain.pem` and `fullchain.pem`
```bash
# chain.pem
cp ../ca/cert.pem ./chain.pem
```
```bash
# fullchain.pem
cat cert.pem >> fullchain.pem
cat ../ca/cert.pem >> fullchain.pem
```
7. Verify the newly created Certificate
```bash
openssl verify -CAfile ../ca/ca.pem -verbose cert.pem
```

## Install/remove CAs on your computer/phone

### Arch Linux or Artix Linux
**Install CA**
```bash
# If you are using sudo, change doas -> sudo
cd ..
doas trust anchor --store ca/cert.pem
doas update-ca-trust
```
**Remove CA**
```bash
# If you are using sudo, change doas -> sudo
# Find your certificate in trust-source
doas ls /etc/ca-certificates/trust-source
doas rm /etc/ca-certificates/trust-source/<FILENAME>
doas update-ca-trust
```

