# sunasty-qr-payload (JavaScript & TypeScript)

Official documentation for the `sunasty-qr-payload` npm package. A zero-dependency, tree-shakeable parser and utility library for QR code payloads (Wi-Fi, 2FA TOTP, vCard, mailto, SMS, tel, and URL security inspection), Web Crypto RFC 6238 TOTP code generation, canvas image preprocessing, and scan history serialization.

Originally developed for the [Sunasty QR Code Scanner & Generator](https://sunasty.com/qr-scan/), the [Sunasty QR Generator](https://sunasty.com/qr-code/), and the [Sunasty QR Scanner Chrome Extension](https://github.com/bsulyan/sunasty-qr-scanner).

---

## Package Overview

- **npm Registry**: [`sunasty-qr-payload` on npmjs.com](https://www.npmjs.com/package/sunasty-qr-payload)
- **CDN (ESM)**: `https://cdn.jsdelivr.net/gh/bsulyan/sunasty-qr-payload@1.0.0/index.js`
- **Source Code**: [GitHub bsulyan/sunasty-qr-payload](https://github.com/bsulyan/sunasty-qr-payload)
- **License**: MIT
- **Dependencies**: 0 external runtime dependencies (< 6 kB minified + gzipped)
- **TypeScript**: First-class type definitions included (`index.d.ts`)
- **Compatibility**: Browser, Node.js (`>= 18.0.0`), Bun, Deno

---

## Installation

```bash
# npm
npm install sunasty-qr-payload

# pnpm
pnpm add sunasty-qr-payload

# yarn
yarn add sunasty-qr-payload

# bun
bun add sunasty-qr-payload
```

### Direct Browser Import (ESM CDN)

```html
<script type="module">
  import { parseQrPayload, generateTotpCode } from 'https://cdn.jsdelivr.net/gh/bsulyan/sunasty-qr-payload@1.0.0/index.js';

  const payload = parseQrPayload('WIFI:S:MyNetwork;T:WPA;P:SecretKey123;;');
  console.log(payload);
</script>
```

---

## Complete API Reference

### 1. Payload Parsing (`parseQrPayload`)

#### `parseQrPayload(data: string): ParsedQrPayload`
Detects and parses raw decoded QR text into a typed discrimination union:

- `kind: 'wifi'`
- `kind: 'vcard'`
- `kind: 'email'`
- `kind: 'sms'`
- `kind: 'phone'`
- `kind: 'totp'`
- `kind: 'url'`
- `kind: 'text'`

```typescript
import { parseQrPayload } from 'sunasty-qr-payload';

const result = parseQrPayload('WIFI:S:OfficeWifi;T:WPA;P:SuperSecretPass;;');

if (result.kind === 'wifi') {
  console.log(`SSID: ${result.ssid}`);
  console.log(`Encryption: ${result.encryption}`);
  console.log(`Password: ${result.password}`);
}
```

#### Supported Payload Formats

##### Wi-Fi Network (`WifiPayload`)
```typescript
interface WifiPayload {
  kind: 'wifi';
  ssid: string;
  encryption: string; // 'WPA', 'WEP', 'nopass'
  password: string;
  hidden: boolean;
}
```

##### 2FA TOTP (`TotpPayload`)
Parses `otpauth://totp/Issuer:account?secret=BASE32&period=30&digits=6`:
```typescript
interface TotpPayload {
  kind: 'totp';
  label: string;
  account: string;
  issuer: string;
  secret: string;
  digits: number;
  period: number;
}
```

##### vCard 2.1 / 3.0 / 4.0 (`VcardPayload`)
Parses contact cards starting with `BEGIN:VCARD`:
```typescript
interface VcardPayload {
  kind: 'vcard';
  fullName: string;
  phone: string;
  email: string;
  org: string;
  rawVcard: string;
}
```

##### Email & MATMSG (`EmailPayload`)
Supports standard `mailto:user@domain.com?subject=...&body=...` as well as Docomo `MATMSG:TO:...;SUB:...;BODY:...;;`:
```typescript
interface EmailPayload {
  kind: 'email';
  to: string;
  subject: string;
  body: string;
  mailtoHref: string;
}
```

##### SMS (`SmsPayload`)
Supports `smsto:+123456789:Hello` and `SMS:+123456789:Hello`.

##### Phone Call (`PhonePayload`)
Supports `tel:+123456789`.

---

### 2. URL Security Analysis

#### `analyzeUrlSecurity(rawUrl: string): UrlSecurityAnalysis`
Performs safety and protocol validation on scanned URLs to alert users about potential phishing, insecure plain HTTP, or raw IP address destinations before navigation.

```typescript
import { analyzeUrlSecurity } from 'sunasty-qr-payload';

const sec = analyzeUrlSecurity('http://192.168.1.1/login');
console.log(sec);
// => {
//   valid: true,
//   href: "http://192.168.1.1/login",
//   protocol: "http:",
//   domain: "192.168.1.1",
//   pathAndQuery: "/login",
//   isInsecureHttp: true,
//   isRawIp: true,
//   hasSecurityWarning: true
// }
```

---

### 3. RFC 6238 TOTP Generator (Web Crypto API)

#### `generateTotpCode(secretBase32: string, timestampMs?: number, period?: number, digits?: number): Promise<TotpResult>`
Generates a cryptographically verified RFC 6238 Time-based One-Time Password using the native Web Crypto API (`crypto.subtle`). Runs entirely on-device with zero server transmission.

Returns:
- `code`: 6-digit (or configured length) formatted authentication token (e.g., `"482910"`).
- `remainingSeconds`: Seconds remaining before expiration (e.g., `18`).
- `period`: Configured time step interval (default `30`).
- `counter`: Unix epoch step index.

```typescript
import { generateTotpCode } from 'sunasty-qr-payload';

const totp = await generateTotpCode('JBSWY3DPEHPK3PXP');
console.log(`Current Code: ${totp.code} (Expires in ${totp.remainingSeconds}s)`);
```

#### `decodeBase32(base32: string): Uint8Array`
Decodes an RFC 4648 Base32 secret string into a raw byte buffer for HMAC-SHA1 signature verification.

---

### 4. Image Preprocessing for Difficult QR Codes

When decoding QR codes from blurry mobile camera feeds, low-contrast document scans, or dark-mode screens, standard decoders (like `jsQR`) frequently fail. `sunasty-qr-payload` exports canvas pixel preprocessing algorithms:

#### `applyHighContrastThreshold(data: Uint8ClampedArray, threshold?: number, invert?: boolean): Uint8ClampedArray`
Converts RGBA pixel buffer to a high-contrast binary map with optional color inversion (essential for light-on-dark inverted QR codes).

```typescript
const ctx = canvas.getContext('2d');
const imgData = ctx.getImageData(0, 0, canvas.width, canvas.height);

// Invert and binarize for inverted QR codes
applyHighContrastThreshold(imgData.data, 128, true);
ctx.putImageData(imgData, 0, 0);
```

#### `normalizeAndBinarizeLowContrast(data: Uint8ClampedArray): Uint8ClampedArray`
Computes luminance min/max bounds across the image buffer and dynamically stretches the dynamic range to reconstruct washed-out finder patterns.

---

### 5. Scan History Serialization

#### `serializeScanHistoryToCsv(results: ScanRecord[]): string`
Exports an array of scan history entries into an RFC 4180 compliant CSV string with escaped delimiters.

```typescript
import { serializeScanHistoryToCsv } from 'sunasty-qr-payload';

const csv = serializeScanHistoryToCsv([
  { timestamp: '2026-10-24T12:00:00Z', type: 'WIFI', data: 'WIFI:S:Guest;;' },
  { timestamp: '2026-10-24T12:05:00Z', type: 'URL', data: 'https://sunasty.com/qr-scan/' }
]);
```

#### `serializeScanHistoryToJson(results: Array<Record<string, unknown>>): string`
Serializes records into formatted JSON with indentation.

---

## Practical Example: End-to-End Scan & 2FA Flow

```typescript
import { parseQrPayload, generateTotpCode } from 'sunasty-qr-payload';

// Raw string extracted from a camera feed or image file
const rawQrString = 'otpauth://totp/Sunasty:admin@sunasty.com?secret=HXDMVJECJJWSRB3HWIZR4IFUGFTMXBOZ&issuer=Sunasty';

const payload = parseQrPayload(rawQrString);

if (payload.kind === 'totp') {
  console.log(`Service: ${payload.issuer} (${payload.account})`);
  
  // Real-time generator loop
  const updateCode = async () => {
    const { code, remainingSeconds } = await generateTotpCode(payload.secret);
    console.log(`Token: ${code} [Refresh in ${remainingSeconds}s]`);
  };

  await updateCode();
}
```

---

## Unit Testing with Node.js (`node:test`)

```javascript
import test from 'node:test';
import assert from 'node:assert/strict';
import { parseQrPayload, generateTotpCode, analyzeUrlSecurity } from 'sunasty-qr-payload';

test('parses wifi payload', () => {
  const result = parseQrPayload('WIFI:S:MyNet;T:WPA;P:Pass123;;');
  assert.equal(result.kind, 'wifi');
  assert.equal(result.ssid, 'MyNet');
  assert.equal(result.password, 'Pass123');
});

test('detects insecure http url warning', () => {
  const sec = analyzeUrlSecurity('http://192.168.1.50/admin');
  assert.equal(sec.isInsecureHttp, true);
  assert.equal(sec.isRawIp, true);
  assert.equal(sec.hasSecurityWarning, true);
});

test('generates deterministic RFC 6238 TOTP', async () => {
  // RFC 6238 test secret "12345678901234567890" in base32 is GEZDGNBVGY3TQOJQGEZDGNBVGY3TQOJQ
  const secret = 'GEZDGNBVGY3TQOJQGEZDGNBVGY3TQOJQ';
  const timestamp = 59000; // T=1
  const result = await generateTotpCode(secret, timestamp);
  assert.equal(result.code.length, 6);
});
```

---

## Online Playground & Utilities

Test QR code generation, scanning, and payload extraction in your browser:
- [Sunasty QR Code Scanner & Reader](https://sunasty.com/qr-scan/)
- [Sunasty QR Code Generator (PNG / SVG)](https://sunasty.com/qr-code/)
- [Sunasty Developer Utilities](https://sunasty.com/dev/)
- [Sunasty QR Scanner Chrome Extension](https://github.com/bsulyan/sunasty-qr-scanner)
