# Merchant Onboarding Platform: Technical & Functional Documentation (Complete)

## 1. Introduction

The **Merchant Onboarding Platform** is a robust, data-driven system designed to streamline the process of acquiring and registering new merchants. This document outlines the technical architecture, functional requirements, and implementation guidelines for building the platform. The system is divided into two primary components: a front-end **Merchant Submission Web Interface** for data entry, and a **Back-Office Dashboard** for administrative review and management.

The platform leverages a modern technology stack to ensure scalability, security, and a seamless user experience. It incorporates complex form logic, including cascading dropdowns, dynamic auto-filling, and strict validation, to minimize manual data entry errors and enhance operational efficiency.

## 2. Core Architecture & Technology Stack

The platform is architected using a modern, decoupled approach, separating the client-side interface from the server-side processing and data storage.

### 2.1. Technology Stack

- **Frontend (Client-Side):** React or Next.js. These frameworks provide the necessary tools for building dynamic, responsive, and state-driven user interfaces, which are essential for handling complex form logic and conditional rendering.
- **Backend (Server-Side):** Node.js or Python. These environments offer robust capabilities for building RESTful APIs, handling file uploads, processing business logic, and integrating with external data sources.
- **Database:** Supabase PostgreSQL (Relational Database). A relational database is required to maintain structured data, manage relationships between merchants, owners, and business entities, and ensure data integrity.

### 2.2. External Data Sources (Raw URLs)

To facilitate accurate data entry and reduce user friction, the platform integrates with external JSON data sources for cascading dropdowns and auto-fill logic. These sources must be fetched dynamically and stored locally as database tables to ensure resilience.

| Data Source | Raw URL | Purpose |
| :--- | :--- | :--- |
| **Address Data** | `https://raw.githubusercontent.com/mmtool/address/main/mm_address.json` | Region, Township, District, City (EN/MM), Postal Code |
| **MCC Data** | `https://raw.githubusercontent.com/mmtool/address/main/mm_mcc.json` | MCC Group, MCC Name, MCC Code |
| **NRC Data** | `https://raw.githubusercontent.com/mmtool/address/main/nrc.json` | NRC Prefix, Tsp Code, Type |

#### Table 2.2.1: Address Data Structure (mm_address.json)

| Field Name | Description | Used For |
| :--- | :--- | :--- |
| **Region** | State or Region name (e.g., Yangon, Mandalay) | Primary dropdown selection |
| **Township** | Township name within a Region | Secondary dropdown, filters by Region |
| **District** | District name | Auto-filled based on Township |
| **City EN** | City name in English | Auto-filled based on Township |
| **City MM** | City name in Myanmar Unicode | Auto-filled based on Township |
| **Postal Code** | Postal code for the area | Auto-filled based on Township |

*Logic:* Region selection filters the Township dropdown. Selecting a Township auto-fills the **District**, **City EN**, **City MM**, and **Postal Code** fields.

#### Table 2.2.2: MCC Data Structure (mm_mcc.json)

| Field Name | Description | Used For | Input Type |
| :--- | :--- | :--- | :--- |
| **MCC Group** | High-level business category (e.g., Retail, Dining) | Primary selection | **Standard Select Dropdown** |
| **MCC Name** | Specific business type name | Secondary selection, filters by Group | **Searchable Dropdown** |
| **MCC Code** | The 4-digit numeric code for the business type | Auto-filled based on MCC Name | **Auto-populated Text/Display Field** |

*Logic:* 
1. User selects an **MCC Group** from a standard dropdown.
2. The **MCC Name** searchable dropdown is populated only with names belonging to that group.
3. Upon selecting an **MCC Name**, the corresponding **MCC Code** is automatically populated.

#### Table 2.2.3: NRC Data Structure (nrc.json)

| Field Name | Description | Used For |
| :--- | :--- | :--- |
| **Prefix (No.)** | Numeric prefix from 1 to 12 representing state/region | Primary dropdown selection |
| **Tsp Code (Township)** | Township code associated with the Prefix | Secondary dropdown, filters by Prefix |
| **Type** | Citizen type (e.g., N - Native, P - Naturalized, E - Associate) | Tertiary dropdown, filters by Prefix |

*Logic:* Prefix selection filters both the Tsp Code and Type dropdowns.

## 3. Merchant Submission Web Interface (Front-End)

The front-end interface is designed to guide the applicant through the onboarding process logically. It employs strict validation, conditional rendering for hidden fields, and dynamic auto-filling.

### 3.1. Section 0: Applicant Information

This initial section captures the details of the individual submitting the application.

| Field Name | Type | Constraints & Logic |
| :--- | :--- | :--- |
| **Applicant Type** | Select Dropdown | Mandatory. Options: "Customer", "Employee". Must be the first field. |
| **Applicant Name** | Text Input | Mandatory. Name of the person completing the form. |
| **Applicant Email** | Email Input | Mandatory. Used for confirmation and status notifications. Must be a valid email format. |

### 3.2. Section 1: Merchant Owner Information

This section gathers personal and identification details of the merchant owner.

| Field Name | Type | Constraints & Logic |
| :--- | :--- | :--- |
| **Merchant Phone No** | Text Input | Mandatory. Primary contact number. |
| **First Name(EN)** | Select Dropdown | Mandatory. Options: "U", "Daw". |
| **Last Name(EN)** | Text Input | Mandatory. Full surname. |
| **First Name(MM)** | (Auto-filled) | Mandatory. If "U" -> "ဦး", if "Daw" -> "ဒေါ်". |
| **Last Name(MM)** | Text Input | Mandatory. Requires Myanmar Unicode. |
| **Date of Birth** | Date Picker | Mandatory. Format: DD/MM/YYYY. |
| **Father Name** | Text Input | Mandatory. Identity verification. |
| **Marital Status** | Hidden Field | Default value: "Single". |
| **Gender** | Hidden Field | Auto-filled based on First Name ("Male" for "U", "Female" for "Daw"). |
| **Contact Number** | Hidden Field | Auto-filled from Merchant Phone No. |

#### 3.2.1. Section 1.2: Owner Identification (NRC)

*Format: Prefix/Tsp Code(Type)/NRC Number (e.g., 12/ma ga da(C)160954)*

| Field Name | Type | Constraints & Logic |
| :--- | :--- | :--- |
| **Prefix** | Select Dropdown | Mandatory. Sourced from NRC table ('No' field, 1–12). |
| **Tsp Code** | Searchable Dropdown | Mandatory. Filtered dynamically by Prefix from 'township' column. |
| **Type** | Select Dropdown | Mandatory. Filtered dynamically by Prefix from 'type' column (e.g., N, P, E). |
| **NRC Number** | Text Input | Mandatory. Restricted to exactly 6 numeric digits. |

#### 3.2.2. Section 1.3: Owner Address

| Field Name | Type | Constraints & Logic |
| :--- | :--- | :--- |
| **Country** | Hidden Field | Default value: "Myanmar". |
| **Region** | Select Dropdown | Mandatory. Sourced from Address table. |
| **Township** | Searchable Dropdown | Mandatory. Filtered dynamically by selected Region. Auto-fills District, City EN, City MM, and Postal Code (all stored in DB). |
| **House No(EN)** | Text Input | Mandatory. Manual input. |
| **Street (EN)** | Text Input | Mandatory. Manual input. |
| **House No(MM)** | Text Input | Mandatory. Myanmar Manual input. |
| **Street (MM)** | Text Input | Mandatory. Requires Myanmar Unicode. |
| **Full Address** | Hidden Field | Auto-generated: `{House No(EN)}, {Street(EN)}, {City EN}, {Postal Code}, Myanmar`. |

### 3.3. Section 2: Merchant Business Information

This section collects details about the business entity and its operational characteristics.

| Field Name | Type | Constraints & Logic |
| :--- | :--- | :--- |
| **Merchant Label EN** | Text Input | Mandatory. Display name. Auto-fills hidden fields: Company Name_EN, Company Short Name_EN, Business Name_EN. |
| **Merchant Label (MM)** | Text Input | Mandatory. Requires Myanmar Unicode. Auto-fills hidden fields: Company Name_MM, Company Short Name(MM), Business Name(MM). |
| **MCC Group** | **Select Dropdown** | Mandatory. Sourced from MCC table (unique groups). When selected, filters the MCC Name dropdown. |
| **MCC Name** | **Searchable Dropdown** | Mandatory. Dynamically populated based on selected MCC Group. User can type to search. |
| **MCC Code** | **Auto-populated Text Field** | Mandatory. Automatically filled when an MCC Name is selected. |
| **Acquirer** | Hidden Field | Default value: "Shwebank". |
| **Offer** | Hidden Field | Default value: "Merchant". |
| **Email** | Hidden Field | Default value: "merchant@mo.com.mm". |
| **DICA / GRN / RCDC** | Text Input | Optional. Business registration IDs. |

#### 3.3.1. Section 2.3: Merchant Location & Operational Data

| Field Name | Type | Constraints & Logic |
| :--- | :--- | :--- |
| **Same As Owner Address** | Toggle Button | If Toggle ON, all address fields are populated from Owner Address. User allowed to edit after. |
| **Region** | Select Dropdown | Mandatory. Sourced from Address table. |
| **Township** | Searchable Dropdown | Mandatory. Filtered dynamically by selected Region. Auto-fills District, City EN, City MM, and Postal Code. |
| **House No(EN)** | Text Input | Mandatory. Manual input. |
| **Street(EN)** | Text Input | Mandatory. Manual input. |
| **House No(MM)** | Text Input | Mandatory. Requires Myanmar Unicode. |
| **Street (MM)** | Text Input | Mandatory. Requires Myanmar Unicode. |
| **Latitude & Longitude** | Text Inputs | Mandatory. Default to user's GPS location via browser Geolocation API, or allow manual input. |
| **Open 24/7** | Hidden Field | Default value: Checked (True). |

### 3.4. Section 3: Document Management & Media

This section handles optional file uploads for verification purposes. The system must support PDF, JPG, and PNG formats.

| Document Type | Description | Format |
| :--- | :--- | :--- |
| **Business Doc** | Official business registration or license. | PDF / Image |
| **Agreement / Contract** | Signed terms and conditions. | PDF / Image |
| **BOD (Board of Directors)** | List of directors. | PDF / Image |
| **Shop Photo** | Image of the storefront. | Image |
| **NRC Front** | Front side of the Owner's ID. | Image |
| **NRC Back** | Back side of the Owner's ID. | Image |

## 4. Back-Office Dashboard

The administrative dashboard provides back-office users with the tools necessary to review, verify, and manage merchant applications.

### 4.1. Authentication & Security

Access to the dashboard is strictly controlled.

- **Credentials:** Default Username: `admin`, Default Password: `admin@123` (Note: These should be changed in a production environment).
- **Mechanism:** Implementation of session-based or token-based (JWT) authentication to secure API endpoints and dashboard routes.

### 4.2. Dashboard Features

The dashboard includes the following core functionalities:

1. **Data Grid / List View:** A comprehensive table displaying all submitted applications. Key columns should include Merchant Label, Owner Name, Submission Date, and Current Status (e.g., Pending, Verified, Rejected).
2. **Detailed Preview Modal/Page:** An interface to view the complete dataset for a specific submission, including all auto-populated hidden fields that are not visible to the applicant.
3. **Document Viewer:** Integrated viewing capabilities or direct download links for all uploaded media associated with a submission.
4. **Export / Download Functionality:**
   - Export submission data as CSV or Excel files.
   - Download a compiled ZIP archive containing all associated media and documents for a specific merchant.
5. **Email Notification Settings:** A configuration panel allowing administrators to define the recipient email address for new submission alerts.
6. **Verification Workflow:** Controls to mark a submission as "Approved" or "Rejected". This action triggers automated email notifications to the applicant.

## 5. Implementation Guidelines

To ensure a robust and reliable platform, the following guidelines must be adhered to during development:

- **Email Notification Logic:**
  - *On Submission:* Trigger an immediate email to the configured administrator address containing a summary of the new application.
  - *On Verification:* Trigger an email to the Applicant Email address detailing the result (Verified or Rejected) when the status is updated in the back-office.
- **Unicode Support:** Ensure comprehensive validation and storage mechanisms for Myanmar Unicode inputs across all relevant fields.
- **External Data Resilience:** The application must seed its database tables from the provided GitHub raw URLs during initial setup. All subsequent operations must read from these local tables. Implement error handling if the seed operation fails.
- **User Experience (UX):** The UI must be responsive, mobile-friendly, and accessible. Clear, descriptive validation messages must be provided for all mandatory fields.
- **Security of Hidden Fields:** Hidden fields must be managed securely on the server-side. The backend must validate all incoming data and not solely rely on client-side logic, preventing manipulation via browser developer tools.
- **MCC Searchable Dropdown:** Implement client-side filtering for the MCC Name dropdown to allow typing/searching, especially useful when a group contains many MCC names.
- **✅ Database Storage of All Fields (Including Hidden Fields):** The database **MUST** store every field from the submission form, including all hidden fields (e.g., `Marital Status`, `Gender`, `Contact Number`, `Country`, `Full Address`, `Acquirer`, `Offer`, `Email`, `Open 24/7`, and any auto‑populated values). Hidden fields are critical for data integrity, auditing, back‑office visibility, and reporting. Do **not** rely on re‑computation; store the exact value as submitted.

## 6. Application Pages Overview

### Web App Onboarding Form Page

This is the public-facing, multi-step form where applicants enter their information. It features:
- A clean, step-by-step layout (Sections 0-3)
- Dynamic dropdowns and auto-filling logic as defined in Section 3
- **MCC Group** as a standard dropdown
- **MCC Name** as a searchable dropdown that updates based on the selected group
- **MCC Code** auto-populating when an MCC Name is selected
- File upload components for documents
- Client-side validation with clear error messages
- A final "Submit Application" button that sends data to the backend API

### Admin Dashboard Page

This is a password-protected interface for administrators. It features:
- A login screen (using `admin` / `admin@123`)
- A main "Applications" table with filtering and sorting
- A "View Details" button for each application, opening a modal or new page showing all submitted data
- An embedded or linked document viewer
- "Approve" and "Reject" action buttons that update the application status
- An "Export" button to download application data and files
- A "Settings" section to configure the notification recipient email address

---

## Appendix A: Complete Database Schema Reference

**Note:** The following tables include **all fields**, both visible and hidden. Hidden fields are marked with `(Hidden)` in the description. Every field submitted from the form must be persisted.

### Table: merchants
| Column | Type | Description |
| :--- | :--- | :--- |
| id | UUID (PK) | Primary key |
| applicant_type | VARCHAR(20) | Customer or Employee |
| applicant_name | VARCHAR(100) | Name of person submitting |
| applicant_email | VARCHAR(100) | Email for notifications |
| status | VARCHAR(20) | Pending, Verified, Rejected |
| created_at | TIMESTAMP | Submission date |
| updated_at | TIMESTAMP | Last update date |

### Table: merchant_owners
| Column | Type | Description |
| :--- | :--- | :--- |
| id | UUID (PK) | Primary key |
| merchant_id | UUID (FK) | References merchants.id |
| phone_no | VARCHAR(20) | Contact number |
| first_name_en | VARCHAR(10) | U or Daw |
| last_name_en | VARCHAR(50) | Surname |
| first_name_mm | VARCHAR(10) | ဦး or ဒေါ် |
| last_name_mm | VARCHAR(50) | Myanmar surname |
| dob | DATE | Date of birth |
| father_name | VARCHAR(100) | Father's name |
| marital_status | VARCHAR(20) | (Hidden) Default: Single |
| gender | VARCHAR(10) | (Hidden) Auto-filled Male/Female |
| nrc_prefix | VARCHAR(5) | 1-12 |
| nrc_tsp_code | VARCHAR(20) | Township code |
| nrc_type | VARCHAR(5) | N, P, E |
| nrc_number | VARCHAR(6) | 6-digit number |
| address_country | VARCHAR(50) | (Hidden) Default: Myanmar |
| address_region | VARCHAR(100) | Region |
| address_township | VARCHAR(100) | Township |
| address_district | VARCHAR(100) | Auto-filled from Township |
| address_city_en | VARCHAR(100) | Auto-filled from Township |
| address_city_mm | VARCHAR(100) | Auto-filled from Township |
| address_postal_code | VARCHAR(20) | Auto-filled from Township |
| address_house_no_en | VARCHAR(50) | House number (EN) |
| address_street_en | VARCHAR(100) | Street (EN) |
| address_house_no_mm | VARCHAR(50) | House number (MM) |
| address_street_mm | VARCHAR(100) | Street (MM) |
| full_address | TEXT | (Hidden) Generated full address |

### Table: merchant_businesses (UPDATED with Company/Short/Business Names)

| Column | Type | Description |
| :--- | :--- | :--- |
| id | UUID (PK) | Primary key |
| merchant_id | UUID (FK) | References merchants.id |
| label_en | VARCHAR(100) | Merchant Label EN (visible input) |
| label_mm | VARCHAR(100) | Merchant Label MM (visible input) |
| **company_name_en** | VARCHAR(100) | (Hidden) Auto-filled from label_en |
| **company_short_name_en** | VARCHAR(100) | (Hidden) Auto-filled from label_en |
| **business_name_en** | VARCHAR(100) | (Hidden) Auto-filled from label_en |
| **company_name_mm** | VARCHAR(100) | (Hidden) Auto-filled from label_mm |
| **company_short_name_mm** | VARCHAR(100) | (Hidden) Auto-filled from label_mm |
| **business_name_mm** | VARCHAR(100) | (Hidden) Auto-filled from label_mm |
| mcc_group | VARCHAR(100) | MCC group |
| mcc_name | VARCHAR(200) | MCC name |
| mcc_code | VARCHAR(4) | 4-digit code |
| acquirer | VARCHAR(50) | (Hidden) Default: Shwebank |
| offer | VARCHAR(50) | (Hidden) Default: Merchant |
| email | VARCHAR(100) | (Hidden) Default: merchant@mo.com.mm |
| dica_grn_rcdc | VARCHAR(100) | Registration IDs |
| business_region | VARCHAR(100) | Business region |
| business_township | VARCHAR(100) | Business township |
| business_district | VARCHAR(100) | Auto-filled from Township |
| business_city_en | VARCHAR(100) | Auto-filled from Township |
| business_city_mm | VARCHAR(100) | Auto-filled from Township |
| business_postal_code | VARCHAR(20) | Auto-filled from Township |
| business_house_no_en | VARCHAR(50) | House number (EN) |
| business_street_en | VARCHAR(100) | Street (EN) |
| business_house_no_mm | VARCHAR(50) | House number (MM) |
| business_street_mm | VARCHAR(100) | Street (MM) |
| latitude | DECIMAL(10,8) | GPS latitude |
| longitude | DECIMAL(11,8) | GPS longitude |
| open_247 | BOOLEAN | (Hidden) Default: TRUE |

### Table: merchant_documents
| Column | Type | Description |
| :--- | :--- | :--- |
| id | UUID (PK) | Primary key |
| merchant_id | UUID (FK) | References merchants.id |
| document_type | VARCHAR(50) | Business Doc, Agreement, BOD, Shop Photo, NRC Front, NRC Back |
| file_path | VARCHAR(500) | Path to stored file |
| file_name | VARCHAR(200) | Original file name |
| mime_type | VARCHAR(50) | PDF, JPG, PNG |
| uploaded_at | TIMESTAMP | Upload timestamp |

### Table: address_cache (for seeded data from mm_address.json)
| Column | Type |
| :--- | :--- |
| region | VARCHAR(100) |
| township | VARCHAR(100) |
| district | VARCHAR(100) |
| city_en | VARCHAR(100) |
| city_mm | VARCHAR(100) |
| postal_code | VARCHAR(20) |

### Table: mcc_cache (for seeded data from mm_mcc.json)
| Column | Type |
| :--- | :--- |
| mcc_group | VARCHAR(100) |
| mcc_name | VARCHAR(200) |
| mcc_code | VARCHAR(4) |

### Table: nrc_cache (for seeded data from nrc.json)
| Column | Type |
| :--- | :--- |
| prefix | VARCHAR(5) |
| tsp_code | VARCHAR(20) |
| type | VARCHAR(5) |

---

*End of Documentation*
