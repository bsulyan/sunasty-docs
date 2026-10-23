# QR Code Payload Parsing

The `sunasty-qr-payload` library parses and validates structured QR code contents completely on-device without any server communication.

## Supported Payload Schemas

### Wi-Fi Network (`WIFI:`)
```text
WIFI:T:WPA;S:MyNetwork;P:MyPassword;H:false;;
```
Parses authentication type (`WPA`, `WEP`, `nopass`), SSID, password, and hidden network flag.

### TOTP 2FA (`otpauth://totp/`)
```text
otpauth://totp/Sunasty:user@example.com?secret=JBSWY3DPEHPK3PXP&issuer=Sunasty
```
Parses secret key, issuer, algorithm (`SHA1`, `SHA256`, `SHA512`), digits (`6` or `8`), and period (`30s`). Includes client-side RFC 6238 HMAC code generation using the Web Crypto API (`SubtleCrypto`).

### vCard Contacts (`BEGIN:VCARD`)
Parses full name (`FN`), organization (`ORG`), phone numbers (`TEL`), email addresses (`EMAIL`), and website URLs (`URL`).

### URL and Deep Links
Validates HTTP/HTTPS links and custom application protocol schemes.

## Implementations

- **JavaScript / TypeScript**: [`sunasty-qr-payload` on npm](https://www.npmjs.com/package/sunasty-qr-payload)
- **Source Code**: [GitHub bsulyan/sunasty-qr-payload](https://github.com/bsulyan/sunasty-qr-payload)

## Interactive Tools

- [Sunasty QR Code Scanner](https://sunasty.com/qr-scan/)
- [Sunasty QR Code Generator](https://sunasty.com/qr-code/)
- [Sunasty Privacy Tools](https://sunasty.com/privacy-tools/)
- [Ulysan Studio](https://ulysan.fr/)
