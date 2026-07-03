## 1. Gateway-only communication

- [x] 1.1 Configure KrakenD as the single mobile-facing entrypoint
- [x] 1.2 Point the mobile app base URL at KrakenD, never at backend services

## 2. OAuth2

- [x] 2.1 Add OAuth2 bearer-token validation at the KrakenD edge
- [x] 2.2 Enable Spring Security resource-server validation on protected backend APIs
- [x] 2.3 Reject missing/invalid tokens at both edge and backend

## 3. Mutual TLS

- [x] 3.1 Enforce mTLS on app-to-gateway and gateway-to-backend channels
- [x] 3.2 Accept only certificates issued by the project trust chain

## 4. Fail-closed TLS

- [x] 4.1 Load project CA trust material in the mobile TLS client
- [x] 4.2 Remove permissive certificate callbacks so verification fails closed
