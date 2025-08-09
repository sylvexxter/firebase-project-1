# Firebase Project 1 — Secure Text Log

A minimal, framework-free web app that lets authenticated users submit short text entries and view their own submissions in real time. It also includes a client-side feature to detect strings that look like JWTs and verify their ES256 signatures using a provided public key.

## What this app does
- Sign in with Google (Firebase Authentication)
- Add a text entry (Firestore `submissions` collection)
- See your entries update live (Firestore real-time listeners)
- If a submitted text looks like a JWT, the app tries to verify it with ES256 using a public key and visually highlights verified rows

## Tech stack
- Hosting: Firebase Hosting (static site served from `public/`)
- Auth: Firebase Authentication (Google provider)
- Database: Cloud Firestore (client SDK)
- UI: Bootstrap via CDN
- JWT verification: `jose` ESM build via CDN (signature verification only)
- Framework: None — plain HTML/CSS/JS

## Firestore security
Rules restrict access so users only read documents where `userId == request.auth.uid`. Creates are limited to specific fields and types; updates/deletes are disallowed. See `firestore.rules` for details.

## JWT verification (ES256 with jose)
- Library: [`jose`](https://github.com/panva/jose) ESM import from CDN
- Public key: a PEM-encoded ES256 public key defined in the client
- Flow:
  1. When rendering rows, if a text matches a simple JWT pattern (three base64url segments), the app attempts a signature verification.
  2. The app imports the public key with `importSPKI` and runs `compactVerify`.
  3. If the signature is valid and `alg` is ES256, the row is visually marked.

Important: This verification is signature-only. It doesn’t validate claims like `exp`, `nbf`, `aud`, or `iss`. Do not rely on this for authorization. Server-side validation is recommended for production use.

Example (from `public/index.html`):

```js
import { compactVerify, importSPKI } from "https://cdn.jsdelivr.net/npm/jose@5.2.3/+esm";

const PUBLIC_ES256_KEY_PEM = `-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----`;
const jwtPublicKeyPromise = importSPKI(PUBLIC_ES256_KEY_PEM, "ES256");

async function isJwtVerifiedEs256(token) {
  try {
    const publicKey = await jwtPublicKeyPromise;
    const { protectedHeader } = await compactVerify(token, publicKey);
    if (protectedHeader?.alg !== "ES256") return false;
    return true;
  } catch {
    return false;
  }
}
```

## Local development
- Serve locally with Firebase emulators:

```bash
firebase emulators:start --only hosting
```

## Deploy
```bash
firebase deploy --only hosting,firestore
```

## Repository
- GitHub: `https://github.com/sylvexxter/firebase-project-1`

## Notes
- Firebase Web `apiKey` in client code is not a secret; protect server credentials and any service accounts.
- For server-only logic (e.g., admin tasks, secure token validation), use Cloud Functions and store secrets in Firebase/Google Cloud configuration.


