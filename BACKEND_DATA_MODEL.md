# Kabadiwala Connect backend data model

This is the proposed Firestore model for the app's Firebase backend. It is a
schema plan, not a dump of live customer data. The role workspaces currently
show local sample records; do not import those records as real users or
transactions.

Use Firebase Auth UIDs as document IDs where possible. Store timestamps as
Firestore `Timestamp` values, money as integer paise (or another documented
minor currency unit), and weights as validated decimal kilograms. Never trust
client-provided role, payment, verification, or state-transition fields without
server-side checks.

## Collections

### `users/{uid}`

One profile for each authenticated account.

| Field | Type | Purpose |
|---|---|---|
| `uid` | string | Firebase Auth UID; must match document ID. |
| `name` | string | Person or business display name. |
| `phone` | string | Verified E.164 phone number. |
| `email` | string | Optional contact email. |
| `role` | string | `customer`, `kabadiwala`, `recycler`, or `admin`. Assign privileged roles through a trusted backend operation. |
| `preferredLanguage` | string | `en`, `hi`, or `mr`. |
| `operatingArea` | string | City/district coverage label. |
| `address` | string | Private pickup/contact address; restrict reads. |
| `location` | GeoPoint | Optional coarse location; do not expose exact customer coordinates publicly. |
| `isActive` | boolean | Account access state. |
| `createdAt`, `updatedAt` | Timestamp | Server timestamps. |

Keep recycler accreditation in a separate private profile document. Avoid
storing redundant phone/name copies in marketplace documents unless a receipt
requires an immutable historical snapshot.

### `recyclerProfiles/{uid}`

Private facility and verification data for recycler accounts.

| Field | Type | Purpose |
|---|---|---|
| `uid` | string | Recycler account UID. |
| `facilityName` | string | Registered business/facility name. |
| `registrationNumber` | string | Regulatory or business registration reference. |
| `facilityAddress` | string | Private facility address. |
| `location` | GeoPoint | Facility location used for matching. |
| `acceptedMaterials` | string[] | Supported material categories. |
| `verificationStatus` | string | `pending`, `verified`, `rejected`, or `suspended`. |
| `documentPaths` | string[] | Private Firebase Storage paths; do not store public download tokens for sensitive documents. |
| `reviewedBy`, `reviewedAt` | string/Timestamp | Admin reviewer and decision time. |

### `scrapLots/{lotId}`

An offerable batch of material. The owner and server control its lifecycle.

| Field | Type | Purpose |
|---|---|---|
| `lotId` | string | Document ID. |
| `customerId` | string | Creating account UID. |
| `collectorId`, `recyclerId` | string/null | Assigned participants, when known. |
| `categoryType`, `subCategoryName` | string | Material classification. |
| `description`, `conditionNote` | string | Material and packing notes. |
| `imagePaths` | string[] | Storage object paths; validate owner, type, and size. |
| `estimatedWeightKg`, `actualVerifiedWeightKg` | number/null | Estimate and verified weight. |
| `estimatedRatePerKg`, `estimatedTotalValue`, `finalQuotedValue` | integer/null | Price snapshot and final amount. |
| `locationAddress`, `location` | string/GeoPoint | Exact location is private; expose only a coarse area for matching. |
| `status` | string | `draft`, `submitted`, `matching`, `offerReceived`, `accepted`, `pickupScheduled`, `collectorAssigned`, `pickedUp`, `weightVerified`, `paymentPending`, `paid`, `handoverCompleted`, `sentToRecycler`, `recycled`, `cancelled`. |
| `createdAt`, `updatedAt` | Timestamp | Server timestamps. |

### `pickupOrders/{orderId}`

Customer pickup request that can contain one or more lots.

| Field | Type | Purpose |
|---|---|---|
| `orderId`, `customerId` | string | Order ID and owner UID. |
| `scrapLotIds` | string[] | Included lot documents. |
| `scheduledDate`, `timeSlot` | Timestamp/string | Requested pickup window. |
| `assignedKabadiwalaId` | string/null | Assigned collector. |
| `totalEstimatedWeightKg`, `totalVerifiedWeightKg` | number/null | Weight totals. |
| `totalEstimatedAmount`, `finalVerifiedAmount` | integer/null | Amounts in minor currency units. |
| `orderStatus` | string | `matching`, `assigned`, `on_the_way`, `arrived`, `completed`, `cancelled`. |
| `paymentStatus`, `paymentMethod` | string | `pending`, `cash_paid`, `upi_paid`; method `cash`, `upi`, or `bank_transfer`. |
| `createdAt`, `updatedAt` | Timestamp | Server timestamps. |

Keep customer contact/address fields in a restricted subdocument or fetch them
only for the assigned collector. Do not publish phone numbers in public feeds.

### `offers/{offerId}`

Collector or recycler bid against a lot/order.

| Field | Type | Purpose |
|---|---|---|
| `offerId`, `orderIdOrLotId` | string | Offer ID and target. |
| `bidderUid`, `bidderRole` | string | Authenticated bidder and role. |
| `offeredAmount` | integer | Amount in minor currency units. |
| `etaTimeSlot`, `note` | string | Proposed collection terms. |
| `status` | string | `pending`, `accepted`, `countered`, `rejected`, `withdrawn`, `expired`. |
| `createdAt`, `updatedAt` | Timestamp | Server timestamps. |

Expose a public bidder name/rating only when needed. Keep bidder phone private.

### `prices/{priceId}`

Published material price by district and category.

| Field | Type | Purpose |
|---|---|---|
| `categoryType`, `subCategoryName`, `districtCode` | string | Price lookup dimensions. |
| `buyingRatePerKg`, `sellingRatePerKg` | integer | Rate in minor currency units per kg. |
| `currency`, `unit` | string | For example `INR` and `kg`. |
| `source`, `verificationStatus` | string | Provenance and review state. |
| `publishedAt`, `expiresAt`, `updatedAt` | Timestamp | Freshness and staleness handling. |
| `updatedBy` | string | Admin UID or trusted backend identity. |

Only published, current prices should be returned to the client. Preserve a
price snapshot on a lot/offer so later price updates cannot rewrite history.

### `handoverRecords/{handoverId}`

Immutable receipt for a collector-to-recycler transfer.

| Field | Type | Purpose |
|---|---|---|
| `handoverId`, `orderIdOrLotId` | string | Receipt ID and related order/lot. |
| `collectorUid`, `recyclerUid` | string | Verified participants. |
| `materialName`, `estimatedWeightKg`, `verifiedWeightKg` | string/number | Material and weights. |
| `finalAmountPaid`, `paymentMethod` | integer/string | Settled amount and method. |
| `paymentReference` | string/null | Optional provider reference; never store credentials. |
| `receiptVersion`, `receiptHash` | number/string | Versioned integrity metadata. |
| `customerConfirmed`, `collectorConfirmed`, `recyclerConfirmed` | boolean | Participant acknowledgements. |
| `createdAt` | Timestamp | Server timestamp. |

Generate receipts through a trusted transaction/function. Do not allow a client
to edit a finalized receipt or choose its QR verification payload.

### `disputes/{disputeId}`

Admin-reviewed issue linked to an order, offer, or receipt.

| Field | Type | Purpose |
|---|---|---|
| `relatedRecordId`, `raisedBy`, `againstUid` | string | Affected record and participants. |
| `reasonCode`, `description` | string | Structured reason and user explanation. |
| `evidencePaths` | string[] | Private Storage paths. |
| `status` | string | `open`, `under_review`, `resolved`, `dismissed`. |
| `resolution`, `resolvedBy`, `resolvedAt` | string/UID/Timestamp | Decision and reviewer. |
| `createdAt`, `updatedAt` | Timestamp | Server timestamps. |

### `adminAuditLogs/{eventId}`

Append-only record for privileged actions such as role grants, recycler
verification, rate publication, dispute decisions, and account suspension.

Store `actorUid`, `action`, `targetPath`, changed field names, a redacted before
and after snapshot where justified, `reason`, and server `createdAt`. Write this
collection only from trusted server code and make it immutable to clients.

### `notifications/{notificationId}`

Per-user delivery record for pickup, offer, verification, payment, and dispute
events. Include `recipientUid`, `type`, `relatedRecordId`, a non-sensitive
message payload, `readAt`, and server `createdAt`. FCM tokens belong in a
separate private device-token store and must be revocable.

## Access and transaction rules

- Deny by default; scope profile/contact data to the owner and authorized
  participants.
- Read roles from a trusted custom claim or protected membership source. Do not
  grant `admin` or `recycler` from a client-editable profile field.
- Use Cloud Functions or validated transactions for offer acceptance, role
  changes, rate publication, payouts, receipt creation, and state transitions.
- Keep all state transitions explicit and reject invalid or repeated changes.
- Query by district/status and participant IDs; add indexes only for queries
  used by the app.
- Use Firebase Emulator Suite fixtures with synthetic identities and records.
  Never put personal customer data, production tokens, or service-account keys
  in this handoff.
