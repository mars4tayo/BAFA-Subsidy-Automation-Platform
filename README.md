

https://github.com/user-attachments/assets/ea2f7be9-a0cd-43e9-8614-880c9600d120









# BAFA Subsidy Automation Platform

## Overview

An end-to-end automation platform that streamlines the German BAFA (Bundesamt für Wirtschaft und Ausfuhrkontrolle) subsidy application process for EEW Modul 4 (Premium) and Modul 6 programs. The system reduces manual effort from weeks to hours by orchestrating data collection, validation, calculation, and document generation.

---

## Problem Statement

Processing BAFA Energy Savings Concept (ESK) applications involves:

- 50+ page PDF forms requiring extensive manual data entry
- Complex calculations for energy consumption, CO₂ savings, and amortization
- Reference machine matching from a database of approved equipment
- Multiple data sources: client information, utility bills, fuel prices, machine specs
- High error risk leading to application rejection or delays

---

## Technical Architecture

### Core Components

| Component      | Purpose 
|----------------|-------------------------------------------------------
| Airtable       | Central database, workflow orchestration, and UI layer
| Custom Scripts | Business logic, validation, and data transformation
| AI Agents      | Intelligent data extraction and matching
| External APIs  | Real-time fuel prices and geocoding

### Automation Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                    AUTOMATION SEQUENCE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐  │
│  │  Create  │───▶│   Find   │───▶│ Confirm  │───▶│   Sync   │  │
│  │  Record  │    │ Machine  │    │Reference │    │   Data   │  │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘  │
│        │                                              │         │
│        ▼                                              ▼         │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐  │
│  │  Fetch   │───▶│Calculate │───▶│ Generate │───▶│  Update  │  │
│  │  Prices  │    │Consumption│    │ESK PDF   │    │ Record   │  │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Features

### 1. Automated Data Validation

- Tax Number Standardization: Converts various German tax number formats to the required 13-digit Bundessteuernummer format for BAFA compliance
- Geocoding: Resolves client addresses to coordinates for fuel price lookup
- Fuel Type Detection: Automatically identifies required fuel types from application context

### 2. Intelligent Reference Machine Matching

- Matches client machine specifications against a curated database of BAFA-approved reference equipment
- Validates that new equipment is within ≤10% capacity deviation from the reference machine (BAFA requirement)
- Scores and ranks potential matches based on technical similarity

### 3. Energy Consumption Calculations

- Annual energy consumption for both electric and reference machines
- Energy conversion: Converts fuel consumption (L/h) to MWh/year using standard energy density values
- CO₂ calculations: Applies BAFA-standard emission factors
- Savings validation: Ensures CO₂ reduction thresholds are met

### 4. Cost & Financial Analysis

- Fuel prices: Real-time pricing from local stations via API
- Electricity costs: Parsed from client utility bills
- Investment eligibility: Filters only BAFA-eligible cost components (equipment, installation, software licenses)
- Förderquote: Automatically applies the correct grant rate (up to 45% for eligible small companies)
- Amortization: Calculates payback period with and without grant funding

### 5. ESK PDF Generation

- Automatically populates all required sections of the BAFA Energy Savings Concept form
- Calculates and inserts all financial and technical data
- Attaches required supporting documents (data sheets, offer letters, utility bills)

---

## Sample Calculations

### CO₂ Savings Calculation

```javascript
// Example: Electric vs. Reference Machine
const referenceConsumption = 10.436; // MWh/year (Petrol)
const newConsumption = 0.136;        // MWh/year (Electricity)
const co2FactorPetrol = 0.264;      // t CO₂/MWh
const co2FactorElectricity = 0.435; // t CO₂/MWh

const co2Reference = referenceConsumption  co2FactorPetrol;   // 2.755 t CO₂/a
const co2New = newConsumption  co2FactorElectricity;          // 0.059 t CO₂/a
const co2Savings = co2Reference - co2New;                     // 2.696 t CO₂/a
const savingsPercent = (co2Savings / co2Reference)  100;      // 97.85%
```

### System Utility Validation

```javascript
// Ensure new machine is within 10% of reference capacity
const referenceCapacity = 1650; // m²/h
const newCapacity = 1519.2;     // m²/h
const deviation = Math.abs(newCapacity - referenceCapacity) / referenceCapacity  100;

// BAFA requirement: ≤10% deviation
if (deviation <= 10) {
    // Valid: Machine comparison accepted
} else {
    // Invalid: Needs alternative reference machine
}
```

---

## Data Sources

| Data Type                 | Source                         | Method 
|---------------------------|--------------------------------|--------
| Fuel Prices               | Fuel stations API              | Real-time API call
| Client Location           | Map API                        | Geocoding API
| Client Electricity Cost   | Utility Bill                   | Manual upload + parsing
| Machine Data              | Reference Machines DB          | Airtable query
| Energy Density Values     | Standardized lookups           | Static database

---

## Technologies Used

- Platform: Airtable (database + orchestration + UI)
- Language: JavaScript (Airtable Scripting)
- AI/ML: Custom similarity scoring algorithms
- APIs: API source (fuel prices), API source (geocoding)
- Output: Automated PDF generation
- Version Control: Git / GitHub

---

## Project Structure

```
├── scripts/
│   ├── bundessteuernummer.js      # Tax number validation
│   ├── findMachine.js             # Reference machine matching
│   ├── confirmReference.js        # Auto-confirm machine selection
│   ├── sync.js                    # Data synchronization
│   ├── fuelFinder.js              # Fuel price API integration
│   └── generateESK.js             # PDF generation engine
├── schemas/
│   ├── m4ToEsk.json               # Main table schema
│   └── referenceMachines.json     # Reference machines schema
├── docs/
│   ├── api_reference.md           # API documentation
│   └── workflow_diagram.png       # Automation flow
└── README.md
```

---

## Key Metrics

| Metric                 | Before                 | After
|------------------------|------------------------|-----------------------
| Application Prep Time  | Weeks                  | Hours
| Data Entry Errors      | High                   | Eliminated
| Manual Checks Required | Extensive              | Minimal
| Scalability            | 1-2 applications/month | 100+ applications/month
| Compliance Rate        | Variable               | Baked into workflow

---

## Future Enhancements

- [ ] Direct BAFA API integration for submission
- [ ] OCR for automated utility bill parsing
- [ ] Machine learning for improved reference matching
- [ ] Multi-language support (English/DE)
- [ ] Dashboard with real-time application status tracking

---

## Contributing

This is a proprietary automation platform designed for internal use. For inquiries about similar solutions, please contact the repository maintainer.

---

## License

All Rights Reserved. This code and documentation are proprietary and confidential.

---

