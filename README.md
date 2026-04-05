
# Merchant Onboarding Platform: Technical & Functional Documentation

## 1. Introduction

The **Merchant Onboarding Platform** is a robust, data-driven system designed to streamline the process of acquiring and registering new merchants. This document outlines the technical architecture, functional requirements, and implementation guidelines for building the platform. The system is divided into two primary components: a front-end **Merchant Submission Web Interface** for data entry, and a **Back-Office Dashboard** for administrative review and management.

The platform leverages a modern technology stack to ensure scalability, security, and a seamless user experience. It incorporates complex form logic, including cascading dropdowns, dynamic auto-filling, and strict validation, to minimize manual data entry errors and enhance operational efficiency.

## 2. Core Architecture & Technology Stack

The platform is architected using a modern, decoupled approach, separating the client-side interface from the server-side processing and data storage.

### 2.1. Technology Stack

- **Frontend (Client-Side):** React or Next.js. These frameworks provide the necessary tools for building dynamic, responsive, and state-driven user interfaces, which are essential for handling complex form logic and conditional rendering.
- **Backend (Server-Side):** Node.js or Python. These environments offer robust capabilities for building RESTful APIs, handling file uploads, processing business logic, and integrating with external data sources.
- **Database:**  Use Supabase PostgreSQL (Relational Database). A relational database is required to maintain structured data, manage relationships between merchants, owners, and business entities, and ensure data integrity.

### 2.2. External Data Sources

To facilitate accurate data entry and reduce user friction, the platform integrates with external JSON data sources for cascading dropdowns and auto-fill logic. These sources must be fetched dynamically:
( Make them into Table no need to use url )
- **Address Data:** Used for Region, Township, District, and City selections.
    - _Source:_ `https://raw.githubusercontent.com/mmtool/address/main/mm_address.json`
    - _Logic:_ Region selection filters Township. Township selection auto-fills District, City EN, City MM, and Postal Code.
- **MCC (Merchant Category Code) Data:** Used for classifying the merchant's business type.
    - _Source:_ `https://raw.githubusercontent.com/mmtool/address/main/mm_mcc.json`
    - _Logic:_ Group selection filters MCC Name. MCC Name selection auto-fills MCC Code.
- **NRC (National Registration Card) Data:** Used for identity verification of the merchant owner.
    - _Source:_ `https://raw.githubusercontent.com/mmtool/address/main/nrc.json`
    - _Logic:_ Prefix selection filters Tsp Code and Type. The 'No' field corresponds to the Prefix, the 'township' array to Tsp Code options, and the 'type' array to Type options.

## 3. Merchant Submission Web Interface (Front-End)

The front-end interface is designed to guide the applicant through the onboarding process logically. It employs strict validation, conditional rendering for hidden fields, and dynamic auto-filling.

### 3.1. Section 0: Applicant Information

This initial section captures the details of the individual submitting the application.

| Field Name          | Type            | Constraints & Logic                                                                      |
| :------------------ | :-------------- | :--------------------------------------------------------------------------------------- |
| **Applicant Type**  | Select Dropdown | Mandatory. Options: "Customer", "Employee". Must be the first field.                     |
| **Applicant Name**  | Text Input      | Mandatory. Name of the person completing the form.                                       |
| **Applicant Email** | Email Input     | Mandatory. Used for confirmation and status notifications. Must be a valid email format. |

### 3.2. Section 1: Merchant Owner Information

This section gathers personal and identification details of the merchant owner.

| Field Name            | Type            | Constraints & Logic                                                   |
| :-------------------- | :-------------- | :-------------------------------------------------------------------- |
| **Merchant Phone No** | Text Input      | Mandatory. Primary contact number.                                    |
| **First Name(EN)**    | Select Dropdown | Mandatory. Options: "U", "Daw".                                       |
| **Last Name(EN)**     | Text Input      | Mandatory. Full surname.                                              |
| **First Name(MM)**    | (Auto-filled)   | Mandatory. If "U" -> "ဦး", if "Daw" -> "ဒေါ်".                        |
| **Last Name(MM)**     | Text Input      | Mandatory. Requires Myanmar Unicode.                                  |
| **Date of Birth**     | Date Picker     | Mandatory. Format: DD/MM/YYYY.                                        |
| **Father Name**       | Text Input      | Mandatory. Identity verification.                                     |
| **Marital Status**    | Hidden Field    | Default value: "Single".                                              |
| **Gender**            | Hidden Field    | Auto-filled based on First Name ("Male" for "U", "Female" for "Daw"). |
| **Contact Number**    | Hidden Field    | Auto-filled from Merchant Phone No.                                   |

#### 3.2.1. Section 1.2: Owner Identification (NRC)
Prefix/Tsp Code(Type)/NRC Number
eg: 12/ma ga da(C)160954

| Field Name     | Type                | Constraints & Logic                                                          |
| :------------- | :------------------ | :--------------------------------------------------------------------------- |
| **Prefix**     | Select Dropdown     | Mandatory. Sourced from NRC JSON ('No' field, 1–12).                         |
| **Tsp Code**   | Searchable Dropdown | Mandatory. Filtered dynamically by Prefix from 'township' array.             |
| **Type**       | Select Dropdown     | Mandatory. Filtered dynamically by Prefix from 'type' array (e.g., N, P, E). |
| **NRC Number** | Text Input          | Mandatory. Restricted to exactly 6 numeric digits.                           |

#### 3.2.2. Section 1.3: Owner Address

| Field Name       | Type                | Constraints & Logic                                                                                                         |
| :--------------- | :------------------ | :-------------------------------------------------------------------------------------------------------------------------- |
| **Country**      | Hidden Field        | Default value: "Myanmar".                                                                                                   |
| **Region**       | Select Dropdown     | Mandatory. Sourced from Address JSON.                                                                                       |
| **Township**     | Searchable Dropdown | Mandatory. Filtered dynamically by selected Region. Auto-fills District, City EN, City MM, and Postal Code (hidden fields). |
| **House No(EN)** | Text Input          | Mandatory. Manual input.                                                                                                    |
| **Street (EN)**  | Text Input          | Mandatory. Manual input.                                                                                                    |
| **House No(MM)** | Text Input          | Mandatory. Myanmar Manual input.                                                                                            |
| **Street (MM)**  | Text Input          | Mandatory. Requires Myanmar Unicode.                                                                                        |
| **Full Address** | Hidden Field        | Auto-generated: `{House No}, {Street}, {City EN}, {Postal Code}, Myanmar`.                                                  |

### 3.3. Section 2: Merchant Business Information

This section collects details about the business entity and its operational characteristics.

| Field Name              | Type                | Constraints & Logic                                                                                                        |
| :---------------------- | :------------------ | :------------------------------------------------------------------------------------------------------------------------- |
| **Merchant Label EN**   | Text Input          | Mandatory. Display name. Auto-fills hidden fields: Company Name_EN, Company Short Name_EN, Business Name_EN.               |
| **Merchant Label (MM)** | Text Input          | Mandatory. Requires Myanmar Unicode. Auto-fills hidden fields: Company Name_MM, Company Short Name(MM), Business Name(MM). |
| **MCC Group**           | Select Dropdown     | Mandatory. Sourced from MCC JSON. Cascading logic: Group -> MCC Name -> MCC Code.                                          |
| **MCC Code**            | Searchable Dropdown | Mandatory. Sourced from MCC JSON. Cascading logic: Group -> MCC Name -> MCC Code.                                          |
| **Acquirer**            | Hidden Field        | Default value: "Shwebank".                                                                                                 |
| **Offer**               | Hidden Field        | Default value: "Merchant".                                                                                                 |
| **Email**               | Hidden Field        | Default value: "[merchant@mo.com.mm](mailto:merchant@mo.com.mm)".                                                          |
| **DICA / GRN / RCDC**   | Text Input          | Optional. Business registration IDs.                                                                                       |

#### 3.3.1. Section 2.3: Merchant Location & Operational Data

| Field Name                | Type                 | Constraints & Logic                                                                                                          |
| :------------------------ | :------------------- | :--------------------------------------------------------------------------------------------------------------------------- |
| **Same As Owner Address** | Toggle Button        | If Toggle On, All Address filled same as Merchant Owner ,Allow to edit,                                                      |
| **Region **               | Select Dropdowns     | Mandatory. Sourced from Address JSON                                                                                         |
| **Township**              | Searchable Dropdowns | .Mandatory. Filtered dynamically by selected Region. Auto-fills District, City EN, City MM, and Postal Code (hidden fields). |
| **House No(EN)**          | Text Input           | Mandatory. Manual input.                                                                                                     |
| **Street(EN)**            | Text Input           | Mandatory. Manual input.                                                                                                     |
| **House No(MM)**          | Text Input           | Mandatory. Requires Myanmar Unicode.                                                                                         |
| **Street (MM)**           | Text Input           | Mandatory. Requires Myanmar Unicode.                                                                                         |
| **Latitude & Longitude**  | Text Inputs          | Mandatory. Default to user's GPS location via browser Geolocation API. or Input Manual                                       |
| **Open 24/7**             | Hidden Field         | Default value: Checked (True).                                                                                               |
### 3.4. Section 3: Document Management & Media

This section handles Options file uploads for verification purposes. The system must support PDF, JPG, and PNG formats.

| Document Type                | Description                                | Format      |     |
| :--------------------------- | :----------------------------------------- | :---------- | --- |
| **Business Doc**             | Official business registration or license. | PDF / Image |     |
| **Agreement / Contract**     | Signed terms and conditions.               | PDF / Image |     |
| **BOD (Board of Directors)** | List of directors.                         | PDF / Image |     |
| **Shop Photo**               | Image of the storefront.                   | Image       |     |
| **NRC Front**                | Front side of the Owner's ID.              | Image       |     |
| **NRC Back**                 | Back side of the Owner's ID.               | Image       |     |

## 4. Back-Office Dashboard

The administrative dashboard provides back-office users with the tools necessary to review, verify, and manage merchant applications.

### 4.1. Authentication & Security

Access to the dashboard is strictly controlled.

- **Credentials:** Default Username: `admin`, Default Password: `admin@123` (Note: These should be changed in a production environment).
- **Mechanism:** Implementation of session-based or token-based (JWT) authentication to secure API endpoints and dashboard routes.

### 4.2. Dashboard Features

The dashboard includes the following core functionalities:

1. **Data Grid / List View:** A comprehensive table displaying all submitted applications. Key columns should include Merchant Label, Owner Name, Submission Date, and Current Status (e.g., Pending, Verified, Rejected).
2. **Detailed Preview Modal/Page:** An interface to view the complete dataset for a specific submission, including all auto-populated hidden fields that are not visible to the applicant.
3. **Document Viewer:** Integrated viewing capabilities or direct download links for all uploaded media (Business Docs, NRC photos, Shop Photos) associated with a submission.
4. **Export / Download Functionality:**
    - Export submission data as CSV or Excel files.
    - Download a compiled ZIP archive containing all associated media and documents for a specific merchant.
5. **Email Notification Settings:** A configuration panel allowing administrators to define the recipient email address for new submission alerts.
6. **Verification Workflow:** Controls to mark a submission as "Approved" or "Rejected". This action triggers automated email notifications to the applicant.

## 5. Implementation Guidelines

To ensure a robust and reliable platform, the following guidelines must be adhered to during development:

- **Email Notification Logic:**
    - _On Submission:_ Trigger an immediate email to the configured administrator address containing a summary of the new application.
    - _On Verification:_ Trigger an email to the Applicant Email address detailing the result (Verified or Rejected) when the status is updated in the back-office.
- **Unicode Support:** Ensure comprehensive validation and storage mechanisms for Myanmar Unicode inputs across all relevant fields.
- **External Data Resilience:** Implement robust error handling for fetching external JSON data. If the primary GitHub raw URLs are unavailable, the application should gracefully degrade (e.g., switch to manual input) or display a clear alert to the user.
- Create Data table from provides GitHub Raw URLs store instead of using github.
- **User Experience (UX):** The UI must be responsive, mobile-friendly, and accessible. Clear, descriptive validation messages must be provided for all mandatory fields.
- **Security of Hidden Fields:** Hidden fields must be managed securely on the server-side. The backend must validate all incoming data and not solely rely on client-side logic, preventing manipulation via browser developer tools.

Clear Provide 
Web App Onboarding Form Page and Admin Page
