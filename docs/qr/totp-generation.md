# Client-Side TOTP 2FA Code Generation

When scanning a 2FA QR code with the [Sunasty QR Code Scanner](https://sunasty.com/qr-scan/), users can immediately preview the active 6-digit rolling code and its remaining validity countdown. 

To ensure absolute privacy, the cryptographic token calculation is executed **100% in the client's browser** using the W3C Web Crypto API (`crypto.subtle`). The secret seed never touches a network interface.

## Mathematical Algorithm (RFC 6238)

A Time-Based One-Time Password (TOTP) is calculated as:

$$\text{TOTP} = \text{HOTP}(K, T)$$

Where:
- $K$ is the shared secret key decoded from Base32 into raw bytes.
- $T$ is the number of time intervals since the Unix epoch:
  $$T = \left\lfloor \frac{\text{CurrentUnixTime} - T_0}{\text{TimeStep}} \right\rfloor$$
  (where $T_0 = 0$ and $\text{TimeStep} = 30$ seconds).
- $\text{HOTP}$ performs an `HMAC-SHA1` (or `SHA256`/`SHA512`) of the 8-byte big-endian representation of $T$ with key $K$.
- Dynamic truncation extracts a 4-byte integer from the 20-byte HMAC digest, modulo $10^6$ for a 6-digit code.

```
 Current Unix Timestamp (e.g. 1792840000)
             │
     [ / 30 seconds ] ──> Counter T (8-byte big-endian buffer)
             │
 Base32 Secret Key K ───> Raw Uint8Array Key
             │
             ▼
  Web Crypto HMAC-SHA1
             │
       20-byte Digest
             │
     [ Dynamic Truncation (Offset = last nibble & 0x0F) ]
             │
      31-bit Integer
             │
       [ Modulo 10^6 ]
             │
             ▼
     6-Digit Rolling Code (e.g. "482910")
```

## Complete Zero-Dependency Implementation

```typescript
/**
 * Decodes a standard Base32 encoded string into a Uint8Array.
 */
function base32ToUint8Array(base32: string): Uint8Array {
  const alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ234567';
  const clean = base32.toUpperCase().replace(/=+$/, '').replace(/\s+/g, '');
  let bits = 0;
  let value = 0;
  const output: number[] = [];

  for (let i = 0; i < clean.length; i++) {
    const idx = alphabet.indexOf(clean[i]);
    if (idx === -1) continue;
    value = (value << 5) | idx;
    bits += 5;
    if (bits >= 8) {
      output.push((value >>> (bits - 8)) & 255);
      bits -= 8;
    }
  }
  return new Uint8Array(output);
}

/**
 * Computes the real-time 6-digit TOTP token using the Web Crypto API.
 */
export async function generateTotpCode(
  secretBase32: string,
  timeStepSeconds: number = 30,
  digits: number = 6,
  algorithm: 'SHA-1' | 'SHA-256' | 'SHA-512' = 'SHA-1'
): Promise<{ code: string; remainingSeconds: number }> {
  const nowSeconds = Math.floor(Date.now() / 1000);
  const timeStep = Math.floor(nowSeconds / timeStepSeconds);
  const remainingSeconds = timeStepSeconds - (nowSeconds % timeStepSeconds);

  // 1. Prepare 8-byte big-endian counter buffer
  const counterBuffer = new ArrayBuffer(8);
  const counterView = new DataView(counterBuffer);
  // High 32 bits are 0 (unless timestamp exceeds year 2038 * 30)
  counterView.setUint32(0, Math.floor(timeStep / 0x100000000));
  counterView.setUint32(4, timeStep & 0xffffffff);

  // 2. Import raw key into Web Crypto API
  const rawKey = base32ToUint8Array(secretBase32);
  const cryptoKey = await crypto.subtle.importKey(
    'raw',
    rawKey,
    { name: 'HMAC', hash: { name: algorithm } },
    false,
    ['sign']
  );

  // 3. Compute HMAC digest
  const signature = await crypto.subtle.sign('HMAC', cryptoKey, counterBuffer);
  const digest = new Uint8Array(signature);

  // 4. Dynamic Truncation (RFC 4226 Section 5.4)
  const offset = digest[digest.length - 1] & 0x0f;
  const binary =
    ((digest[offset] & 0x7f) << 24) |
    ((digest[offset + 1] & 0xff) << 16) |
    ((digest[offset + 2] & 0xff) << 8) |
    (digest[offset + 3] & 0xff);

  // 5. Modulo to requested number of digits
  const code = (binary % Math.pow(10, digits)).toString().padStart(digits, '0');

  return { code, remainingSeconds };
}
```

## Security Guarantees

1. **Zero Exfiltration**: Computing the token locally avoids leaking user credentials or 2FA seeds to external telemetry servers.
2. **Hardware-Accelerated Web Crypto**: The calculation executes in constant time inside the browser's native cryptographic module, mitigating timing attack vectors.

Try scanning your 2FA backup codes locally:
- [Sunasty QR Code Scanner](https://sunasty.com/qr-scan/)
- [Sunasty Privacy Tools](https://sunasty.com/privacy-tools/)
- [Sunasty QR Scanner Chrome Extension](https://github.com/bsulyan/sunasty-qr-scanner)
