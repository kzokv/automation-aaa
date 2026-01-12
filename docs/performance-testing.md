# Performance Testing Utilities

The `apps/performance` directory contains specialized utilities for facilitating load testing and performance analysis.

> [!NOTE]
> This app is not a standalone K6/JMeter runner. Instead, it serves as an **Authentication Bridge** for external performance tools.

## 🔑 Authentication Generation

Performance tests often require valid OIDC tokens (`idToken`, `accessToken`) to authorize API requests. Simulating the complex Cognito SRP (Secure Remote Password) protocol in tools like JMeter is difficult and brittle.

Our solution is a lightweight Node.js utility (`authorize.js`) that uses the official `amazon-cognito-identity-js` SDK to generate valid tokens.

### Usage

This script is typically invoked by external runners (e.g., Python scripts or CI jobs) to "bootstrap" a performance test session.

```javascript
// authorize.js
const { CognitoUser, AuthenticationDetails } = require("amazon-cognito-identity-js");

// Automates the SRP handshake
cognitoUser.authenticateUser(authDetails, {
    onSuccess: (result) => {
        // Output structured JSON for downstream tools
        console.log(JSON.stringify({ 
            idToken: result.idToken.jwtToken,
            tenantID: result.idToken.payload.usertenantid 
        }));
    }
});
```

### Integration Flow

1.  **CI/CD**: Sets `ADMIN_USERNAME` and `ADMIN_PASSWORD` env vars.
2.  **Runner**: Executes `node apps/performance/src/authorize.js`.
3.  **Output**: Captures the JSON output.
4.  **Load Test**: Injects the `idToken` into the `Authorization` header of 1000+ concurrent requests.
