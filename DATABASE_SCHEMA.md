# Kabadiwala Connect - Firestore Database Schema

---

## 1. `users` Collection
**Path:** `users/{uid}`

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `uid` | String | Unique Firebase user ID |
| `name` | String | Full user name or business title |
| `phone` | String | Mobile number (+91 format) |
| `role` | String | Role: `customer`, `kabadiwala`, `recycler`, `admin` |
| `preferredLanguage` | String | Language code: `en`, `hi`, `mr` |
| `operatingArea` | String | City or area location |
| `address` | String | Full doorstep pickup address |
| `latitude` | Number | GPS latitude coordinate |
| `longitude` | Number | GPS longitude coordinate |
| `isAuthorizedRecycler` | Boolean | True for EPA/PCB verified recyclers |
| `createdAt` | Timestamp | Account creation time |

---

## 2. `scrapLots` Collection
**Path:** `scrapLots/{lotId}`

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `lotId` | String | Unique scrap lot ID |
| `customerId` | String | Creator UID |
| `categoryType` | String | `paper`, `plastic`, `metal`, `eWaste`, `glass`, `cardboard` |
| `subCategoryName` | String | Item name (e.g. Copper Wire, PET Bottles) |
| `description` | String | Condition & packing notes |
| `imageUrls` | Array | Storage photo URLs |
| `estimatedWeightKg` | Number | Approximate weight entered by seller |
| `actualVerifiedWeightKg` | Number | Verified weight measured by collector |
| `estimatedRatePerKg` | Number | Category market rate at creation |
| `estimatedTotalValue` | Number | Estimated payout valuation |
| `status` | String | `submitted`, `matching`, `accepted`, `picked_up`, `recycled` |

---

## 3. `pickupOrders` Collection
**Path:** `pickupOrders/{orderId}`

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `orderId` | String | Unique pickup order ID |
| `customerId` | String | Customer UID |
| `scrapLotIds` | Array | Array of lot IDs included in pickup |
| `assignedKabadiwalaId` | String | Assigned collector UID |
| `totalEstimatedWeightKg` | Number | Combined weight estimate |
| `totalEstimatedAmount` | Number | Combined payout estimate |
| `scheduledDate` | Timestamp | Doorstep pickup date |
| `timeSlot` | String | Scheduled time window |
| `orderStatus` | String | `matching`, `assigned`, `arrived`, `completed` |
| `paymentStatus` | String | `pending`, `cash_paid`, `upi_paid` |

---

## 4. `handoverRecords` Collection
**Path:** `handoverRecords/{handoverId}`

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `handoverId` | String | Unique handover receipt ID |
| `orderIdOrLotId` | String | Associated order/lot ID |
| `collectorUid` | String | Collector UID |
| `recyclerUid` | String | Recycler UID |
| `verifiedWeightKg` | Number | Actual weighed value |
| `finalAmountPaid` | Number | Final payout |
| `qrPayloadString` | String | Public QR verification string |
| `timestamp` | Timestamp | Completion time |
