# QR Code Payload Schemas

The `sunasty-qr-payload` library parses structured text content found inside QR codes into strongly-typed objects. It operates 100% locally without external dependencies or network requests.

Published on npm: [`sunasty-qr-payload`](https://www.npmjs.com/package/sunasty-qr-payload)

## Supported Schemas

### 1. Wi-Fi Network (`WIFI:`)

Enables automatic device Wi-Fi connection.

#### Format Specification
```text
WIFI:T:<AuthType>;S:<SSID>;P:<Password>;H:<Hidden>;;
```

- `T`: Authentication type (`WPA`, `WEP`, `nopass`, or empty).
- `S`: Network SSID name. Special characters `\`, `;`, `,`, and `:` must be escaped with a backslash.
- `P`: Pre-shared network key / password.
- `H`: Optional boolean (`true` or `false`) indicating a hidden SSID broadcast.

#### Parsed Output Structure
```typescript
interface WifiPayload {
  type: 'wifi';
  ssid: string;
  auth: 'WPA' | 'WEP' | 'nopass';
  password?: string;
  hidden: boolean;
}
```

---

### 2. Time-Based One-Time Password (`otpauth://totp/`)

Standard format used by authenticator apps (Google Authenticator, Authy, Bitwarden) for two-factor authentication (2FA).

#### Format Specification
```text
otpauth://totp/[Issuer:]AccountName?secret=BASE32KEY&issuer=Issuer&algorithm=SHA1&digits=6&period=30
```

- `secret`: Base32-encoded secret cryptographic seed (RFC 3548).
- `issuer`: Name of the service provider or application.
- `algorithm`: Cryptographic hash function (`SHA1`, `SHA256`, `SHA512`). Default is `SHA1`.
- `digits`: Output token length (usually `6` or `8`). Default is `6`.
- `period`: Time step in seconds (usually `30` or `60`). Default is `30`.

---

### 3. Electronic Contact Cards (`BEGIN:VCARD`)

Standard contact cards following the vCard 3.0 / 4.0 specification (RFC 6350).

#### Format Specification
```text
BEGIN:VCARD
VERSION:3.0
N:Sulyan;Bastien;;;
FN:Bastien Sulyan
ORG:Ulysan
TITLE:Software Engineer
TEL;TYPE=CELL:+33600000000
EMAIL;TYPE=WORK:contact@ulysan.fr
URL:https://sunasty.com
END:VCARD
```

---

### 4. Communication Protocols

- **Mailto (`mailto:`)**: Standard email URI including recipient, `?subject=...`, and `?body=...`.
- **SMS (`smsto:`)**: Mobile SMS dispatch including phone number and predefined message body.
- **Telephone (`tel:`)**: Direct dialer shortcut.
- **Geolocation (`geo:`)**: Coordinate pair `geo:48.5734,7.7521` (latitude, longitude, optional altitude).

---

## TypeScript Parsing Example

```typescript
import { parseQrPayload } from 'sunasty-qr-payload';

// 1. Parse Wi-Fi
const wifi = parseQrPayload('WIFI:T:WPA;S:SunastyOffice;P:SuperSecretPass;H:false;;');
if (wifi.type === 'wifi') {
  console.log(`SSID: ${wifi.ssid}, Security: ${wifi.auth}`);
}

// 2. Parse 2FA Authenticator Seed
const totp = parseQrPayload('otpauth://totp/Sunasty:admin@sunasty.com?secret=JBSWY3DPEHPK3PXP&issuer=Sunasty');
if (totp.type === 'totp') {
  console.log(`Issuer: ${totp.issuer}, Account: ${totp.label}`);
}
```

Try parsing and scanning QR codes live in your browser:
- [Sunasty QR Code Scanner](https://sunasty.com/qr-scan/)
- [Sunasty QR Code Generator](https://sunasty.com/qr-code/)
- [Sunasty Chrome Extension](https://github.com/bsulyan/sunasty-qr-scanner)
