# DEG Purchase Orders — Beta

Web-based PO request and tracking tool for Distributed Energy Infrastructure LLC.  
Single-file HTML app · MSAL auth · Microsoft Graph API · SharePoint backend.

Same stack and Azure registration as **Project Truth V3**.

---

## Deployment

```bash
git clone https://github.com/distributedei/DEG-PurchaseOrders.git
# Drop index.html into repo root, push
# GitHub Pages → serves at distributedei.github.io/DEG-PurchaseOrders/
```

Push via PAT (avoids iPad en-dash corruption):
```bash
git remote set-url origin https://[PAT]@github.com/distributedei/DEG-PurchaseOrders.git
git add index.html && git commit -m "beta" && git push
```

---

## Azure App Registration

Reuses the Project Truth V3 registration — no changes needed.

| Setting | Value |
|---|---|
| Client ID | `613928f8-0497-47ea-bb0f-c1725a1f156a` |
| Tenant ID | `ec357a32-5b85-438d-a765-ae33c8eb13b7` |
| Site ID   | `8103997e-0688-4832-a5b4-bf18de7f96f5` |
| Scopes    | `Sites.ReadWrite.All` · `Files.ReadWrite.All` |

**Add Redirect URI** in Azure → App Registration → Authentication:  
`https://distributedei.github.io/DEG-PurchaseOrders/`

---

## SharePoint Lists Required

### 1. `PO-List` ← **Create this new list**

| Internal Name | Display Name | Type | Notes |
|---|---|---|---|
| `Title` | PO Number | Single line | e.g. PO-2026-001 · assigned by Corinne |
| `PODate` | Date | Date | |
| `POStatus` | Status | Choice | Draft · Submitted · Approved · Rejected |
| `ProjectName` | Project | Single line | |
| `VendorName` | Vendor | Single line | |
| `VendorDetails` | Vendor Details | Single line | Auto-filled from Vendor List |
| `SellerQuote` | Seller Quote # | Single line | |
| `POType` | PO Type | Choice | Equipment · Service |
| `ShipTo` | Ship To | Single line | |
| `Contact1` | Contact 1 | Single line | |
| `Contact2` | Contact 2 | Single line | |
| `CreatedBy` | Created By | Single line | |
| `BallInCourt` | Ball in Court | Single line | Who is actioning |
| `Comments` | Comments | Multi-line | |
| `LineItemsJSON` | Line Items JSON | Multi-line | JSON array of line item objects |
| `Subtotal` | Subtotal | Number | Currency format |
| `Tax` | Tax | Number | |
| `Shipping` | Shipping | Number | |
| `OtherCharges` | Other | Number | |
| `Total` | Total | Number | |
| `VendorQuoteURL` | Vendor Quote URL | Hyperlink | Link to uploaded quote |
| `DocuSignStatus` | DocuSign Status | Single line | e.g. Pending · Signed · Complete |

### 2. `ProjectData` ← Already exists in Project Truth

Needs column: `Title` (project name). App reads this for the project dropdown.

### 3. `Vendor-List` ← Already exists (Vendor Prequal system)

Needs columns: `Title` (vendor name), `Address`, `City`, `State`.

### 4. `Employees` ← Create or reuse existing

| Internal Name | Type |
|---|---|
| `Title` | Single line (Full Name) |
| `Email` | Single line |
| `Title_x002f_Role` | Single line |

---

## Workflow

```
Equipment/Service Identified
        │
   ┌────┴────┐
Equipment   Service
   │           │
Low Risk?      │
  │Yes         └──→ Create & Submit PO Request
  │                        │
  └──→ Approved Submittal? │
           │Yes             │
           └──────────────→ PO logged in SharePoint "PO-List"
                                    │
                             Corinne Assigned
                                    │
                          Agreed Terms / MSA / PSA?
                                    │Yes
                          PDF of PO + Terms → DocuSign
                                    │
                          Vendor Signs → DEI Signs
                                    │
                          EXE Documented · PO-List Updated
                                    │
                          PO Creator Notified ✓
```

---

## Features (Beta)

- [x] MSAL sign-in (Microsoft 365)
- [x] PO List dashboard — KPIs, searchable/filterable table
- [x] New PO form — project, vendor, personnel, line items, file upload
- [x] Detail slide-in panel — full PO view with workflow progress bar
- [x] Edit existing POs
- [x] Delete POs
- [x] Status management (Draft → Submitted → Approved)
- [x] Vendor auto-fill from Vendor List
- [x] Line item calculator with subtotal / tax / shipping / other / grand total
- [ ] DocuSign integration (next phase)
- [ ] Email notification to Corinne on submit (Power Automate trigger)
- [ ] MSA terms stored as column in Vendor List
- [ ] File upload to SharePoint document library
- [ ] PO number auto-generation by Corinne role

---

## Next Phase (Post-Beta)

### Power Automate Flow — on PO-List item create
1. Trigger: SharePoint item created in `PO-List`
2. Action: Send email to Corinne with PO details + link
3. Action: Set `BallInCourt` = "Corinne"
4. When Corinne assigns PO#: update `Title` + notify submitter

### DocuSign Integration
- When status → Approved: generate PDF from PO data
- Upload PDF + MSA terms to DocuSign envelope
- Send to vendor → then DEI signatory (Prateek)
- On complete: update `DocuSignStatus`, attach executed PDF

### MSA Terms on Vendor List
- Add `MSAStatus` column to Vendor-List (Active MSA / PSA / None)
- Add `MSADocURL` column linking to executed agreement
- Auto-populate `Comments` field in PO with standard vs custom terms flag

---

## File Structure

```
DEG-PurchaseOrders/
├── index.html          ← entire app (single file)
└── README.md
```
