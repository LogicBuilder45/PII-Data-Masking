**PII Data Masking with SSIS & KingswaySoft Data Anonymizer**

**1. What is PII and why mask it?**

Personally Identifiable Information (PII) is any data that can be used to identify a specific person. Typical examples in a SQL Server database:

- Names (FirstName, LastName, FullName)
- Addresses (Street, City, State, PostalCode)
- Contact details (Email, Phone)
- Government IDs, customer IDs, account numbers
- Date of birth, IP address in some contexts

If this data leaks into non-production environments (dev / QA / UAT) or into analytics sandboxes, it can cause:

- Privacy violations and compliance issues (GDPR, HIPAA, PCI, etc.) 
- Legal and financial penalties
- Reputation damage

Data masking / anonymization replaces sensitive values with realistic but fake data, while keeping:

- The same schema
- The same data types and patterns
- The same row counts and relationships

So developers and analysts can work with “real-looking” data without exposing real people. KingswaySoft’s Data Anonymizer component is built exactly for this use case. 


**2. What this guide covers**

This page shows how to use KingswaySoft SSIS Productivity Pack – Data Anonymizer to mask PII when moving data with SSIS.

You will learn how to:
1. Identify PII columns in a SQL Server database.
2. Build an SSIS Data Flow that:
    - Reads from a source table
    - Masks PII columns using Data Anonymizer
    - Writes masked data to a target table (or another environment)

3. Configure anonymization rules for common PII fields (name, email, phone, address).
4. Apply best practices for repeatable and consistent anonymization.



**3. Prerequisites**

You’ll need:

- SQL Server + sample database (e.g., AdventureWorks or WideWorldImporters). 
- SSIS (installed with SQL Server Data Tools / Visual Studio).
- KingswaySoft SSIS Productivity Pack with the Data Anonymizer component installed. 

Once Productivity Pack is installed, Data Anonymizer will appear in the SSIS Toolbox under Data Generation and Anonymization.

**4. High-level design**

The basic pattern:

Source (Prod / Copy) → Data Anonymizer (mask PII) → Destination (Dev / Test / Analytics)

In SSIS Data Flow, that looks like:

- OLE DB Source (or any other source)

- Data Anonymizer (KingswaySoft)

- OLE DB Destination (or flat file / other system)

You can run this as a one-time migration or scheduled job that regularly refreshes masked data.

**5. Step-by-step implementation**
Step 1 – Identify PII columns

Example using an imaginary dbo.Customer table:

SELECT TOP 10
    CustomerID,
    FirstName,
    LastName,
    EmailAddress,
    PhoneNumber,
    AddressLine1,
    AddressLine2,
    City,
    PostalCode,
    CreatedDate
FROM dbo.Customer;


Mark which columns need masking:

**Direct PII:** FirstName, LastName, EmailAddress, PhoneNumber, AddressLine1/2

**Quasi**-**PII** (depends on your rules): City, PostalCode, CreatedDate

You’ll use this list inside Data Anonymizer.

**Step 2 – Create an SSIS package & Data Flow**

In Visual Studio / SSDT, create a new SSIS project.

Add a Data Flow Task to the Control Flow (rename it to Mask PII – Customer).

Open the Data Flow and drag an OLE DB Source.

Configure connection to your SQL Server.

Use the query above or simply select the dbo.Customer table.

**Step 3 – Add the Data Anonymizer component**

From the SSIS Toolbox, drag Data Anonymizer onto the Data Flow.

Connect OLE DB Source → Data Anonymizer.

Double-click Data Anonymizer to open the editor.

The Columns grid shows all incoming columns. For each PII column you want to mask, you’ll choose an Anonymization Type and parameters. 


**Step 4 – Configure anonymization rules (examples)**

The Data Anonymizer provides many anonymization types (Name, Email, Numeric ranges, Address, GUID, etc.). 
KingswaySoft

Below is one realistic configuration for the Customer table:

**4.1 Names**

Columns: FirstName, LastName

Anonymization Type:

FirstName → First Name

LastName → Last Name (or generic Name depending on version)

Effect: real names are replaced with random realistic names (John → Ethan, Priya → Sofia), preserving gender patterns where possible.

Tip: If you want to preserve uniqueness, enable options like source data affinity or seeded randomness so the same original value always maps to the same fake value. 


**4.2 Email addresses**

Column: EmailAddress

Anonymization Type: Email

Optional configuration:

Domain: masked.company.test (or something non-real)

Preserve username length: Yes

Example:

**Original Email	          |            Masked Email**

john.smith@contoso.com     |     	jq8p2@masked.company.test

priya.shah@example.org      |    	h4lrm@masked.company.test



**4.3 Phone numbers**

Column: PhoneNumber

Anonymization Type: Phone Number (or Random Digits)

Region / format: choose a format similar to your production data (e.g., (###) ###-####).

Example:

**Original Phone	| Masked Phone**

(312) 555-9821	| (773) 555-3142

+1-630-555-4477	| +1-224-555-8903



**4.4 Street address**

Column: AddressLine1

Anonymization Type: Address Line 1

Option: Include Address Line 2 = False (you can handle AddressLine2 separately). 
KingswaySoft

Column: AddressLine2

Either set to <Null> or basic pattern like Random Text.

Example:

**Original AddressLine1	| Masked AddressLine1**

123 W Main Street |	842 Oakview Drive

9B Liberty Plaza Apt 3A	| 57 Hillcrest Lane



**4.5 IDs and keys**

For IDs that are used as foreign keys, you have options:

Leave them as-is (no masking) if they are non-sensitive technical keys.

Or use Random Integer / GUID with a stable seed if you want them anonymized but consistent.

For completely internal IDs (e.g., loyalty card numbers):

Column: LoyaltyCardNumber

Anonymization Type: Random Digits or GUID


**4.6 Non-PII fields**

For non-sensitive fields:

Set Anonymization Type = <Ignore> so Data Anonymizer passes them through unchanged. 
KingswaySoft

**Step 5 – Add a destination**

Drag OLE DB Destination to the Data Flow.

Connect Data Anonymizer → OLE DB Destination.

Create a new table like dbo.Customer_Masked or point to a dev database.

Example DDL:

SELECT TOP 0 *
INTO dbo.Customer_Masked
FROM dbo.Customer;


Then in SSIS, map columns 1:1 from Data Anonymizer to dbo.Customer_Masked.

Run the Data Flow. You should now see:

SELECT TOP 10 *
FROM dbo.Customer_Masked;


…showing realistic but safely masked values.

**6. Two common patterns**
**Pattern A – Copy & Mask (Prod → Non-Prod)**

Source = Production database (read-only).

Data Anonymizer masks PII columns.

Destination = Dev/QA database.

This keeps prod untouched and ensures lower environments never see real PII. 


**Pattern B – Mask in Place (Single Environment)**

Source = same database.

Data Anonymizer masks data.

Destination = staging / temp table.

Use SQL UPDATE / MERGE to overwrite original values.

This is useful when you want to “scrub” an existing sandbox that already has live data.

**7. Making masking consistent & repeatable**

KingswaySoft’s documentation highlights several advanced options for consistent anonymization, such as: 

Random seed control – so you get predictable outputs run-over-run if needed.

Source data affinity – the same input value → same anonymized output (e.g., all rows for “John Smith” map to “Liam Brown” everywhere).

Connected entity generation – maintain relationships across multiple tables.

Why this matters:

Reports and joins still work.

Testers can track a single “fake person” across systems.

Re-running the job doesn’t completely reshuffle identities unless you want it to.

**8. Example: Before vs After masking**

Before (dbo.Customer):

**CustomerID |	FirstName	LastName |	EmailAddress |	PhoneNumber |	AddressLine1**

1001	| John	| Smith	| john.smith@contoso.com |	(312) 555-9821	| 123 W Main Street

1002	| Priya	| Shah	| priya.shah@example.org |	(630) 555-4477	| 9B Liberty Plaza Apt


**After (dbo.Customer_Masked):**

**CustomerID	|FirstName	LastName|	EmailAddress|	PhoneNumber|	AddressLine1**

1001	|Ethan	|Lopez|	v7z6p@masked.company.test	|(773) 555-3142	|842 Oakview Drive

1002|	Sofia|	King	|r9wq2@masked.company.test	|+1-224-555-8903	|57 Hillcrest Lane

Same structure, totally different identities.
