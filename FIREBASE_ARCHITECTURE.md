# Kabadiwala Connect - Firebase Backend Architecture

**Live Firebase Project ID:** `kabadiwalaconnectdb`  
**Primary Platform Target:** Flutter Android & iOS Mobile Client  

---

## 1. System Topology

```text
                        KABADIWALA CONNECT MOBILE APP
                                     │
           ┌─────────────────────────┼─────────────────────────┐
           │                         │                         │
           ▼                         ▼                         ▼
     CUSTOMER APP              KABADIWALA APP             RECYCLER APP
           │                         │                         │
           └─────────────────────────┼─────────────────────────┘
                                     │
                                     ▼
                      FIREBASE BACKEND (kabadiwalaconnectdb)
                                     │
   ┌───────────────────┬─────────────┴─────────────┬───────────────────┐
   │                   │                           │                   │
   ▼                   ▼                           ▼                   ▼
Firebase Auth      Cloud Firestore             Firebase Storage       Cloud Messaging
   │                   │                           │                   │
 Phone OTP       users / scrapLots             Scrap Images         Order Alerts
 Role Session    pickupOrders / offers         Digital Receipts     Bidding Updates
```

---

## 2. Authentication & Role Claims
- **Primary Auth:** Phone Number SMS OTP (`FirebaseAuth.instance.verifyPhoneNumber`)
- **Web Auth:** Firebase Web `signInWithPhoneNumber` with the Firebase reCAPTCHA verifier; authorize `localhost` and each deployed web domain in Firebase Authentication settings.
- **User Document:** Created/stored at `users/{uid}` in Firestore
- **Role Permissions:**
  - `customer`: Creates scrap lots, requests pickup orders, views offers, confirms digital handovers.
  - `kabadiwala`: Views nearby pickup feed, submits counter-bids, verifies weights on-site, receives payouts.
  - `recycler`: Manages factory intake lots, verifies authorized material batches, updates price boards.
  - `admin`: Approves recycler facility accreditations, manages category rates, resolves marketplace disputes.

The current client loads the profile role after authentication. A role stored in
a client-writable user document is not a secure authorization boundary; move
privileged role grants to custom claims or a protected membership collection
before production, as described in `BACKEND_TASKS.md`.

---

## 3. Real-Time Data Pipeline & Status Machine
```text
Customer Creates Scrap Lot
         │
         ▼
Pickup Order Published (status: "matching")
         │
         ▼
Streamed to Kabadiwala Nearby Feed
         │
         ▼
Kabadiwalas Submit Bids / Accept
         │
         ▼
Customer Accepts Offer (status: "accepted")
         │
         ▼
Kabadiwala Collects & Verifies Weight (status: "weight_verified")
         │
         ▼
UPI / Cash Payout Issued
         │
         ▼
Digital Handover Receipt Generated (status: "handover_completed")
```
