# Common Bug Patterns for Security Risk Assessment Tool

## 1. Electron Vulnerabilities

- **Context Isolation Disabled:**
  ```javascript
  webPreferences: {
    contextIsolation: false // VULNERABLE
  }
  ```
- **Node Integration Enabled:**
  ```javascript
  webPreferences: {
    nodeIntegration: true // VULNERABLE
  }
  ```
- **Unsafe IPC (Inter-Process Communication):**
  Missing validation on `ipcMain.on` channels. Allowing arbitrary execution via IPC messages.
- **Enable Remote Module:**
  ```javascript
  webPreferences: {
    enableRemoteModule: true // VULNERABLE in older Electron
  }
  ```

## 2. Cross-Site Scripting (XSS)

- **Unsafe DOM Manipulation:**
  Directly injecting user input into the DOM using jQuery without escaping.
  ```javascript
  $('#element').html(userInput); // VULNERABLE
  $('#element').append('<div>' + userInput + '</div>'); // VULNERABLE
  ```
- **Fix:** Use `.text()` instead of `.html()`, or sanitize HTML before insertion.

## 3. XML External Entity (XXE) Processing

- **xml2js Unsafe Parsing:**
  If the application uses `xml2js` or similar libraries to parse user-uploaded XML files, ensure it rejects external entities. Node.js `xml2js` doesn't support XXE by default (it uses `sax-js`), but check for custom parsers or other packages (like `libxmljs` with `noent: true`).

## 4. Logic Flaws & Error Handling

- **Swallowed Exceptions:**
  ```javascript
  try {
    // ...
  } catch (err) {
    // Empty catch block - silently failing
  }
  ```
- **Unhandled Promise Rejections:**
  Missing `.catch()` in Promise chains or missing `try/catch` in `async/await` blocks.
- **Data Validation Bypass:**
  In `/lib/src/model/schema/`, ensure `ajv` schemas properly enforce `additionalProperties: false` where appropriate to prevent mass assignment, and that all critical string fields have `maxLength` and regex validations.

## 5. Security & Cryptography

- **Hardcoded Secrets:**
  Look for hardcoded tokens, passwords, or encryption keys in the codebase.
- **Insecure Randomness:**
  Use of `Math.random()` for security-sensitive operations (e.g. ID generation) instead of `crypto.randomBytes` or `crypto.randomUUID()`.