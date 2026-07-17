# London Property Listings — Salesforce App

A ready-to-deploy Salesforce app for browsing and managing real estate listings
across London boroughs. Built as a standard SFDX source-format project so it can
be deployed to any Salesforce org and, from there, packaged for AppExchange.

## What's included

| Component | Purpose |
|---|---|
| `Property_Listing__c` custom object (20 fields) | Address, postcode, borough, price, beds/baths, type, status, tenure, EPC rating, agent contact info, etc. |
| `PropertyListingController.cls` (+ test class) | Apex backend powering search/filter, 100% method coverage in the test class |
| `propertyListingBrowser` LWC | Search/filter UI — borough, type, status, price range, min bedrooms |
| `propertyCard` LWC | Reusable property card used inside the browser |
| `London_Property_Listings` custom app + tab | Navigation entry point |
| `London_Property_Listings_User` permission set | Grants object/field/Apex/tab access |
| `data/sample_property_listings.csv` | 15 sample London listings across 14 boroughs |

## 1. Deploy to your org

You'll need [Salesforce CLI](https://developer.salesforce.com/tools/salesforcecli) (`sf`) installed locally and a Dev Hub–enabled org (required for AppExchange packaging later — a free [Developer Edition org](https://developer.salesforce.com/signup) works and can be Dev Hub–enabled in Setup).

```bash
# from inside the london-property-listings folder
sf org login web --alias LondonPropertyDev --set-default

# deploy all metadata
sf project deploy start --source-dir force-app

# assign yourself the permission set
sf org assign permset --name London_Property_Listings_User

# import sample data
sf data import tree --plan data/sample_property_listings.csv 2>/dev/null || \
sf data import bulk --sobject Property_Listing__c --file data/sample_property_listings.csv --wait 5
```

(If the bulk import command's flags differ for your CLI version, the CSV also imports cleanly through **Data Import Wizard** in Setup, or Workbench.)

## 2. Add the browser component to a page

The LWC (`propertyListingBrowser`) is exposed for App Pages, Home Pages, and Record Pages, so add it visually rather than via hand-edited metadata:

1. Setup → **Lightning App Builder** → New → App Page → name it "London Property Listings Home"
2. Drag `propertyListingBrowser` from the Custom components list onto the page
3. Activate it and add it to the `London Property Listings` app's navigation, or set it as the app's default landing page
4. Open the **London Property Listings** app from the App Launcher to try it end-to-end

## 3. What's already AppExchange-review-friendly

- `with sharing` on the Apex controller, no `without sharing` anywhere
- No hardcoded IDs, no `SeeAllData=true` in tests
- Dynamic SOQL uses bind variables (not string concatenation of user input) to avoid injection
- Test class has 5 test methods covering positive paths, filters, and bulk-safe assertions
- Permission set (not profile edits) used for access control, matching current ISV best practice

## 4. Path to actually publishing on AppExchange

Deploying this app to your org is step one. Publishing it publicly is a separate, Salesforce-managed process — here's what's still required, roughly in order:

1. **Become a Salesforce ISV Partner** at [partners.salesforce.com](https://partners.salesforce.com) — gives you a Partner Business Org, Environment Hub, and License Management App
2. **Move this source into a Packaging Org** and create a **2nd-Generation Managed Package (2GP)** from it (`sf package create`, `sf package version create`)
3. **Write a Solution Architecture Document** and prepare a demo video, screenshots, and listing copy
4. **Pass Salesforce's Security Review** ($999 fee for paid listings, waived for free apps) — expect several weeks of review + remediation cycles
5. **Create the AppExchange listing** in the Partner Portal (Publishing → Listings → New Listing), attach the package, set pricing
6. **Go live** once the listing and package are both approved

Realistic timeline for the full path (partner signup → live listing) is typically several weeks to a few months depending on how much the security review flags. I outlined this in more detail earlier in this conversation if you want to revisit it.

## Extending this app

Natural next additions if you want to keep building:
- A map view (Lightning Maps component or a custom LWC using `Latitude__c`/`Longitude__c`)
- A public Experience Cloud site so buyers/renters can browse without a Salesforce login
- A `Property_Enquiry__c` object + flow to capture buyer interest and route it to agents
- Duplicate/matching rules on `Postcode__c` + `Address__c`
