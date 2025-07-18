# Portal – Contact Multiple Email Addresses

## Executive Summary

### Project Overview
Enhancement to Portal's contact management system to support multiple email addresses per contact, replacing the current 3-field limitation with a flexible, status-based email management system.

### Business Justification
**Critical Problem**: When contacts change email addresses, historical email communications become "orphaned" and are lost from the unified communication view, breaking a core value proposition of the Portal system for legal case management.

**Business Impact**: Loss of complete client communication history affects legal matter management, compliance, and client service quality.

### Key Stakeholders
- **Primary Users**: Lawyers (coordinators, reviewers, team members) - 80% of usage
- **Secondary Users**: Business operations staff (invoicing, client support) - 20% of usage
- **Technical Stakeholders**: Development team, system administrators
- **Business Stakeholders**: Legal practice partners, compliance officers

### Success Criteria Summary
- 100% preservation of historical email data during migration
- 80%+ reduction in "missing email history" support tickets
- Seamless user experience with new multi-email management capabilities
- No degradation in system performance or email association processing

### Project Scope
**Included:**
- Migration from 3 email fields to separate email address table
- 5-status email management system (Primary, Invoice, Active, Dormant, Ignore)
- Updated email association batch processing
- New communications box UI with email management capabilities
- Complete codebase update across all identified integration points

**Excluded:**
- Changes to Microsoft 365 email download process
- Modifications to email deduplication logic
- Integration with external email systems beyond current M365 setup

## Current State

Currently each contact has up to 3 emails. All 3 emails are separate fields in the contact table.

The existing email fields are:
- Primary Email
- Secondary Email  
- Invoice Email

When the contact changes emails, one of these emails is replaced. The old replaced email is no longer recorded in portal.

## Technical Context: Email Association Process

Portal's core functionality provides a **unified communications view** by downloading all firm emails and associating them with contacts via email address matching. This enables lawyers and business operations staff to see complete communication history for each contact.

### Current Email Matching Process

- **Exact email address matching** against only the single `Email` field (Primary field only - Secondary and Invoice fields are currently ignored in matching)
- **Batch processing**: Emails downloaded and matched in batches every 1 hour via Hangfire background jobs within the main Portal application
- **Deduplication logic**: 
  - For emails to multiple staff members: First TO recipient wins, others discarded
  - For emails involving multiple contacts: Each contact gets their own copy of the email
- **User base**: Primarily lawyers (coordinators/reviewers/team members), with business operations staff using for invoicing and client support

### Integration with ALP System

- Portal is part of the ALP legal practice management system (.NET Core + Vue.js + PostgreSQL)
- Microsoft 365 email integration
- Emails stored in dedicated `emails` table with many-to-many contact associations via `ContactEmails` junction table
- Email communication displayed in contact's email tab for unified case management

## User Analysis

### User Personas

#### Primary Persona: Senior Lawyer
**Background**: Sarah, Senior Lawyer with 5+ years experience
**Role**: Legal coordinator managing 40-60 active matters
**Technology Comfort**: Moderate - comfortable with business software
**Email Management Needs**:
- Needs complete communication history for each client across matter lifecycle
- Often deals with clients who change jobs/email addresses
- Requires quick access to previous communications for matter preparation
- Values accuracy and completeness over advanced features
**Pain Points with Current System**:
- Loses track of client communications when email addresses change
- Spends time searching for "missing" emails
- Client communication history appears incomplete for matter documentation

#### Secondary Persona: Business Operations Manager  
**Background**: Mark, Business Operations with focus on billing and client relations
**Role**: Manages invoicing, payment follow-up, and client account maintenance
**Technology Comfort**: High - power user of business systems
**Email Management Needs**:
- Reliable invoice delivery to correct email addresses
- Ability to maintain separate billing contacts for clients
- Accurate client communication records for billing disputes
- Efficient bulk communication management
**Pain Points with Current System**:
- Invoice delivery failures when client email addresses change
- Difficulty maintaining separate billing vs. general communication addresses
- Limited audit trail for client communications

#### Supporting Persona: Legal Principal
**Background**: David, Principal with 15+ years experience
**Role**: Matter reviewer and client relationship oversight
**Technology Comfort**: Low to moderate - relies on efficient, simple interfaces
**Email Management Needs**:
- Quick overview of client communication status so her can speak knowledgeably to the client about where there matter is up to
- Confidence in complete communication records for regulatory compliance
- Simple interface that doesn't require training
**Pain Points with Current System**:
- Lack of confidence in communication record completeness
- Concerns about regulatory compliance for communication retention

### User Journey Analysis

#### Current State Journey: "Client Changes Jobs"
1. **Trigger**: Client starts new job, gets new email address
2. **User Action**: User updates client's primary email in Portal
3. **System Behavior**: Old email address is overwritten and lost
4. **Problem**: Historical emails associated with old address become "orphaned"
5. **User Experience**: User notices "missing" emails, creates support ticket
6. **Business Impact**: Incomplete case documentation, compliance concerns

#### Future State Journey: "Client Changes Jobs" 
1. **Trigger**: Client starts new job, gets new email address
2. **User Action**: User adds new email address with Primary status
3. **System Behavior**: New email becomes Primary, old email automatically becomes Dormant
4. **Outcome**: All historical emails remain visible, new emails go to new address
5. **User Experience**: Seamless transition with complete communication history preserved
6. **Business Impact**: Enhanced case documentation, improved compliance

## Security & Compliance Requirements

### Data Security Requirements

#### Multi-Tenant Data Isolation
- **Email Address Uniqueness**: Email addresses must be unique across entire system (all tenants)
- **Tenant Isolation**: Email associations must respect tenant boundaries
- **Access Control**: Users can only modify email addresses for contacts within their tenant
- **Audit Trail**: All email address changes must be logged with user, timestamp, and change details

#### Data Protection
- **Encryption**: Email addresses stored using same encryption standards as existing contact data
- **PII Handling**: Email addresses treated as personally identifiable information (PII)
- **Access Logging**: Email address access and modifications logged for security auditing
- **Data Retention**: Email addresses subject to same retention policies as contact data

### Legal Industry Compliance

#### Communication Record Retention
- **Legal Requirement**: Complete client communication history must be maintained for regulatory compliance
- **Email Association Integrity**: System must ensure all historical emails remain associated with correct contacts
- **Audit Requirements**: Changes to email associations must be auditable for compliance reporting
- **Data Integrity**: Email communication links must be immutable once established

#### Regulatory Considerations
- **Client Confidentiality**: Email address changes must not compromise client confidentiality
- **Matter Management**: Email communication history critical for legal matter documentation
- **Professional Standards**: System must support legal professional communication record requirements
- **Trust Account Correlation**: Email communications may be linked to trust account transactions requiring preservation

### Security Constraints

#### Email Association Process Security
- **Batch Processing**: Email association process must maintain security controls during bulk operations
- **Error Handling**: Security-conscious error handling that doesn't expose sensitive information
- **Performance Monitoring**: Security monitoring of email association process for anomalies
- **Failsafe Mechanisms**: Security failsafes if email association process encounters errors

#### Integration Security
- **Microsoft 365**: Maintain existing security standards for M365 email integration
- **API Security**: Email management APIs must enforce same authentication/authorization as existing Portal APIs
- **Database Security**: New email table must implement same security controls as existing contact table

## Business Problem

The business problem is that when a contact changes email addresses, we lose sync with their old email addresses, and therefore old emails no longer show in the email communication tab.

Users want to see all emails that have gone to and from a contact, this includes emails to the contact's current email address, as well as former email addresses.

In addition, contacts often have more than one email address at the same time, but for different purposes. For example, some contacts want invoices sent to a different email address.

**Critical Impact**: When email addresses are overwritten, historical emails become "orphaned" and break the unified communication view essential for legal case management.

## Solution: Allowing for Multiple Email Addresses

We need to break the email fields into a separate table and create one-to-many relationships between contacts and emails.

We also need to attach a status to each email address, so users and portal know what email address to use for different purposes.

## Email Address Status

The email status is an attribute of an email record in the email address table.

We need to be able to designate the following address status:

- **Primary**: A single primary email address for most communications
- **Invoice**: A single email address to receive invoices – only necessary if invoices are not to go to the Primary address
- **Active**: Other active email addresses used by the contact
- **Dormant**: Other dormant email addresses previously used by the contact but no longer used
- **Ignore**: Email addresses that should be completely excluded from email matching and communication processes

### Email Routing Rules

- Portal defaults to sending emails to the **Primary** address
- If the contact has an address with status **Invoice**, then invoices are sent to this address, and all other emails go to the Primary address
- If no invoice address exists, then invoices are sent to the Primary address by default
- Other **Active** addresses can still be used to communicate with the contact, and the system will capture emails to and from this address. However, the system will not use these addresses for general communications or for Invoice purposes
- The **Dormant** addresses are ones that the contact has used historically, but which they have deprecated and no longer use. The system will still associate emails to and from these addresses with the contact and display in the email communications tab. This will allow users to see all historical email communications with the contact, even at these old addresses. Dormant addresses remain active in the email matching process as contacts sometimes revert to using old email addresses
- **Ignore** addresses are completely excluded from all email matching processes and communication workflows. Use this status for email addresses that should never be processed by the system. If an existing email is changed to Ignore, it should not remove any existing associations, only not use this address in the future matching process.

### Email Address Constraints

- **Invoice Email Exception**: Invoice status email addresses **CAN be duplicated** across contacts to accommodate business scenarios (e.g., shared accountants, billing departments)
- **Unique Constraint for Non-Invoice Emails**: Primary, Active, Dormant, and Ignore status emails must remain unique across the system
- **Status Change Restriction**: Cannot change Invoice email to another status if that email address already exists elsewhere in the system
- **Resolution Path**: Users must delete Invoice email addresses rather than change their status if duplication conflict exists
- **Contact Without Email**: A contact can be established without an email address using the 'I do not have email address' checkbox

### Email Association Process Integration

The new multiple email address structure must integrate seamlessly with Portal's existing email association process:

- **Batch Email Matching**: The hourly batch process will search across ALL email addresses for a contact (Primary, Invoice, Active, Dormant) when associating downloaded emails
- **Ignore Status Exclusion**: Email addresses with "Ignore" status are completely excluded from the email matching process (on a go-forward basis, i.e. from when this status applies. There is no need to un-associate emails that have already been associated with the Contact)
- **Historical Matching**: Dormant email addresses continue to participate in email matching, as contacts sometimes revert to using previous email addresses
- **Multiple Contact Matching**: Invoice email addresses may match multiple contacts (e.g., shared accountant), resulting in the same email being associated with multiple contacts simultaneously
- **Enhanced Email Association**: Email matching may return multiple contacts for a single email address due to Invoice email duplication - all matched contacts receive email associations

### Performance Considerations

- **Expected Volume**: Most contacts have only one email address; expansion primarily accommodates invoice emails and dormant addresses from job changes
- **Resource Usage**: Email processing remains resource-intensive but not expected to be materially worse with separate email table
- **Potential Efficiency Gains**: Separate email table structure may improve email matching process efficiency compared to current 3-field approach from more detailed Contact table
- **Monitoring**: Continue existing resource monitoring during email association batch processing
- **Updated Matching Logic**: Email association process requires modification to query separate email table instead of contact table email fields

## Displaying Multiple Emails in the Contact Detail View

### Communications Box UI Design

Email addresses display within a **communications box** in the Contact detail view (alongside phone numbers) to accommodate multiple addresses:

**Display Order:**
1. `xxx@xxx.com (Primary)` [First]
2. `xxx@xxx.com (Invoice)` [Second in order]
3. `xxx@xxx.com (Active)` [Third in order]
4. `xxx@xxx.com (Active)` [Third in order]
5. `xxx@xxx.com (Dormant)` [After primary, invoice and active addresses…]
6. `xxx@xxx.com (Dormant)` [After primary, invoice and active addresses…]
7. `xxx@xxx.com (Ignore)` [Last in order]

### Performance & Display Constraints

- **Maximum Display**: Show up to **6 email addresses** in the communications box
- **Scrolling Interface**: Additional email addresses beyond 6 are accessible via scrolling. Adjust as necessary for optimal look and feel of UI and adopt best practices
- **Efficient Loading**: Contact detail view handles email table join without degrading page load performance
- **Database Query Optimization**: Single query to retrieve all contact email addresses with status

### Email Status Management

#### User Interface Controls

- **Add Email**: Users click a "+" button to add new email addresses
- **Delete Email**: Users click an inline delete button next to each email address (except Primary emails). Emails are soft-deleted in email table and data is retained
- **Status Changes**: Inline dropdown selector for each email address
- **Primary Email Protection**: Primary email addresses display without delete button and cannot be removed directly

#### Primary and Invoice Email Management

- **Locked Dropdowns**: Primary and Invoice statuses are **locked** in the dropdown and cannot be changed directly
- **Indirect Management**: To set a new Primary or Invoice email:
  1. Add a new email address and set status to Primary/Invoice, OR
  2. Change an existing email address status to Primary/Invoice
- **Automatic Demotion**: When a new Primary/Invoice is set, the existing Primary/Invoice automatically becomes Active
- **Single Constraint Enforcement**: System maintains exactly one Primary and one Invoice address through this automatic demotion process. No complex workflows are required to change these status

#### Status Rules

- You can have **unlimited Active, Dormant, and Ignore** addresses
- **Ignore** status should be used sparingly and only when an email address must be completely excluded from all system processes (on a go-forward basis from status change to Ignore)

#### Validation and Error Handling

- **Email Format**: Must be well-formed email address - **invalid emails are blocked from saving**
- **Uniqueness**: Email address must not already exist in the system
- **Seamless Primary/Invoice Changes**: No error messages needed for Primary/Invoice conflicts - system automatically handles demotion of existing Primary/Invoice to Active status

#### Email Deletion Rules

- **Primary Email Protection**: Primary email addresses **cannot be directly deleted**. To remove a Primary email: 1) Add a new email address and set status to Primary (existing Primary automatically becomes Active), 2) Delete the now-Active email address.
- **Non-Primary Deletion**: All other email statuses (Invoice, Active, Dormant, Ignore) can be directly deleted
- **No Email Constraint**: Contacts can exist without any email addresses (handled by checkbox during contact setup)

### Email Address Deletion Policy

When an email address is deleted from a contact, the system must handle existing email associations appropriately. This policy balances data integrity, legal compliance, and business requirements.

#### Core Principle: Preserve Communication History

**Default Behavior**: **Existing email associations are PRESERVED** when an email address is deleted.

**Business Justification**:
- Maintains complete communication history essential for legal case management
- Supports regulatory compliance for communication record retention
- Preserves audit trail for matter documentation
- Aligns with core Portal value proposition of unified communication view

#### Deletion vs Status Change Guidance

**Use Email Address Deletion When**:
- Do not want email as option in other areas of Portal
- Do not want email displayed in communications box - tidy up display
- Email address was incorrectly entered/associated with wrong contact
- Email address contains sensitive information that must be removed
- System cleanup of test/invalid data
[Conflict: As above that email associations are retained even after email delete. Delete will therefore not achieve some of these objectives, e.g. clean up, remove sensitive information, etc]

**Use Status Change to "Ignore" When**:
- Email address is no longer active but was legitimately used by contact
- Want to stop future email matching but preserve historical associations
- Client requests removal from communications but history must be maintained
- Temporary suppression of email address without losing associations

**Use Status Change to "Dormant" When**:
- Contact no longer uses email address but may revert to it
- Email address is historical but should remain searchable
- Want to maintain complete communication timeline

#### Email Deletion Business Rules

1. **Primary Email Exception**: Primary email addresses cannot be deleted - no delete button is displayed for Primary emails
2. **Association Preservation**: Existing email associations remain intact and continue to display in contact's email communication tab
3. **Search Functionality**: Deleted email addresses are removed from contact search but historical emails remain findable through contact
4. **Audit Trail**: Email address deletion is logged with timestamp, user, and reason
5. **Display Behavior**: Deleted email addresses no longer appear in contact's communications box
6. **Email Matching**: Deleted email addresses are permanently excluded from future email association batch processing

#### Deletion Confirmation Process

**Standard Deletion Confirmation**:
```
"Delete Email Address: [email@example.com]

This will remove the email address from the contact but preserve existing email communication history.

• This email address will no longer appear in the contact's email list  
• Future emails to/from this address will not be associated with this contact
• This action cannot be undone

Consider changing status to 'Ignore' instead if you want to stop future matching while keeping the email address visible.

[Cancel] [Change to Ignore] [Delete Email Address]"
```

**Primary Email Deletion Prevention**:
Primary email addresses have no delete button and cannot be selected for deletion. Users attempting to remove a Primary email are guided through the proper process:

```
"Primary Email Cannot Be Deleted

To remove this Primary email address:
1. Add a new email address and set its status to Primary
2. The current Primary will automatically become Active status  
3. You can then delete the Active email address if needed

[Add New Email Address] [Cancel]"
```

#### Special Considerations

**Legal Industry Requirements**:
- Email deletion must not compromise legal matter documentation
- Email deletion will occur as soft-delete in the email table
- Audit trail must support regulatory compliance requirements  
- Communication history preservation takes precedence over data cleanup

**Security Considerations**:
- Email address deletion removes PII from active contact data
- Historical email associations may still contain the email address in email metadata
- Consider data retention policies and right-to-be-forgotten requirements

**Technical Implementation**:
- Soft delete email address record (mark as deleted rather than hard delete)
- Maintain foreign key relationships to preserve email associations
- Update email association batch process to exclude deleted email addresses
- Ensure reporting and search functions respect deleted status

#### First Email Address Behavior

- **Automatic Primary Status**: When the first email is added to a contact with no existing emails, it automatically receives **Primary** status
- **Status Lock**: This first email's Primary status cannot be changed except by adding a new Primary email (which triggers automatic demotion to Active)

## Display of Emails Associated with a Contact

All emails to and from all addresses are associated with the Contact (Primary, Invoice, Active and Dormant) and are then displayed in the emails communication tab for the contact. This allows users to see all email correspondence with the Contact, no matter what address has been used. 

**This solves the business problem identified above.**

## Migration

### Data Migration Strategy

We need to migrate the 3 existing email fields into the new email table and associate with the same contact and relevant status:

- **Primary Email** → **Primary** status
- **Secondary Email** → **Active** status  
- **Invoice Email** → **Invoice** status
- **Null/Empty fields** → Ignored (no email record created)

### Pre-Migration Validation

- **Duplicate Check**: Run validation to identify any duplicate email addresses across different contacts before migration
- **Duplicate Resolution**: Manual review required for any duplicates found
  - If few duplicates: Manual resolution process
  - If many duplicates discovered: Build conflict resolution interface for bulk handling
- **Data Quality Check**: Verify existing email format validation is sufficient
- **Constraint Verification**: Ensure no data conflicts will violate the new uniqueness constraints

### Code Impact Analysis

Multiple components reference the current 3 email fields and will require updates:

- **Invoicing System**: Currently checks `invoice_email` field, falls back to `primary_email`  
- **Reports & Analytics**: Various reports reference email fields directly
- **API Endpoints**: External integrations may consume email field data
- **Integration Points**: Email sending logic, contact exports, etc.
- **Analytics**: Email addressed are currently used as a parameter in many Metabase analytics reports

**Comprehensive Codebase Analysis Completed**: Detailed analysis of the ALP codebase identified extensive usage patterns across all application layers requiring migration updates:

#### Critical Business Logic Components

**1. Core Invoice Email Routing Logic**
- **Client.Email Property** (`ALP.Data/Models/Clients/Client.cs` lines 59-67): Computed property implementing core invoice routing logic that returns `InvoiceEmail` if not null/empty, otherwise falls back to `Email`
- **Invoice Services** - Multiple instances of email selection pattern:
  - `InvoiceService.cs` - `SendFriendlyReminderEmail`, `SendFirstReminderEmail`, `SendSecondReminderEmail`, `SendInvoiceStatementEmail`, `SendInvoice`, `ReSendInvoice`
  - All use pattern: `(PrimaryContact.InvoiceEmail != null && PrimaryContact.InvoiceEmail != "" ? PrimaryContact.InvoiceEmail : PrimaryContact.Email)`
- **Syntaq Integration** (`SyntaqDataService.cs` line 1206): External service integration providing email addresses for invoice processing

**2. Email Association Process**
- **Contact Matching Logic** (`EmailSyncService.cs` lines 127-146): Core email association logic that searches only against single `Email` field
- **Batch Processing** (`HangfireSyncUserEmailsJob.cs`): Email download and association process running every 1 hour
- **Contact Assignment** (`EmailSyncService.cs`): Logic for associating emails with contacts based on exact address matching

#### Entity Framework & Database Layer

**3. Core Entity Models**
- **Contact Entity** (`ALP.Data/Models/Contacts/Contact.cs` lines 48-56): Three email properties with validation attributes
- **Database Configuration** (`ContactConfiguration.cs` lines 10-11): Unique index and required constraints on Email field
- **Migration Files** (`20211101005031_Init.cs` lines 16-18): Database schema definitions for email columns

**4. Data Transfer Objects (DTOs)**
- **Backend C# DTOs**:
  - `ContactDto.cs` (lines 11-15): Three email properties
  - `CreateContactInput.cs` (lines 13-19): Email validation attributes
  - `UpdateContactInput.cs` (lines 11-17): Update operation DTOs
- **Frontend TypeScript DTOs**:
  - `contact-dto.ts` (lines 32-34): TypeScript interface definitions
  - All contact interfaces include three email fields

#### API Controllers & Endpoints

**5. Contact Management APIs**
- **ContactController.cs**: All CRUD operations exposing email fields
- **Contact Creation/Update**: Create and update endpoints with email validation
- **Contact Search**: Search functionality includes email field queries
- **Contact Import** (`ContactController.cs` line 316): CSV import functionality for contacts

**6. Email-Related Controllers**
- **ContactEmailController.cs**: API endpoints for contact email operations
- **Email attachment handling**: Download and import functionality
- **Email export operations**: PDF export functionality for contact emails

#### Frontend Vue.js Components

**7. Contact Management UI**
- **Contact Forms** (`ContactInfo.vue` lines 82-120): Inline editing for all three email fields
- **Contact Creation** (`CreateContact.vue` lines 84-120): Contact creation form with email inputs
- **Contact Tables** (`Contacts.vue`): Email display in contact lists and tables

**8. Contact Selectors & Displays**
- **ContactSelector.vue** (line 14-16): Dropdown components displaying contact with email
- **ContactSelectorNoCreate.vue**: Contact selection without creation options
- **MultiInputContactSelector.vue**: Multi-select contact components showing emails

**9. Email Communication Components**
- **ContactEmails.vue**: Email communication view for contacts
- **Email viewers**: Email communication displays in contact context

#### Import/Export & Data Processing

**10. Contact Data Import/Export**
- **Contact Import** (`Contacts.cs` lines 90-95, 154-160): Legacy data import with three email fields
- **CSV Import** (`ContactService.cs` lines 1343-1348): CSV import mapping classes with email fields
- **Contact Export**: Contact data export functionality including email fields

**11. Email Data Processing**
- **Email Import** (`Emails.cs`): Email data import with contact association logic
- **Email Export** (`EmailService.cs` lines 1264-1296): Contact email export to PDF functionality

#### External Integrations

**12. Third-Party Service Integrations**
- **ActiveCampaign Integration** (`ActiveCampaignService.cs` lines 59-62): Marketing platform sync using contact emails
- **Gravatar Integration**: Profile picture generation using email addresses
- **Microsoft 365**: Email sync and authentication using email addresses

#### Validation & Business Rules

**13. Email Validation Logic**
- **Regex Validation**: Consistent email format validation across all DTOs
- **Uniqueness Constraints**: Database-level unique constraints on primary email
- **Required Field Logic**: Business rules for required vs optional email fields

#### Search & Filtering Operations

**14. Contact Search Functionality**
- **Email-based Search** (`ContactService.cs` line 1142): Search queries including email field matching
- **Contact Filtering**: Filter operations across email addresses
- **Organisation Contact Lists**: Email display in organisation-related contact views

#### Reporting & Analytics

**15. Email-Related Reporting**
- **Contact Reports**: Various reports referencing email fields directly
- **Communication Analytics**: Email communication tracking and reporting
- **Invoice Reporting**: Invoice delivery tracking using email addresses

### Migration Process

Based on the comprehensive codebase analysis, the migration process requires coordinated updates across all application layers:

#### Phase 1: Database & Entity Layer Migration

1. **Pre-migration validation and duplicate resolution**
   - Run validation against all three email fields across contacts
   - Identify and resolve duplicate email addresses that would violate uniqueness constraints
   - Validate email format consistency across existing data

2. **Create new email address table with migrated data**
   - Design new `ContactEmails` table with `ContactId`, `EmailAddress`, `Status`, audit fields
   - Implement Entity Framework model and configuration
   - Create migration scripts to populate new table from existing fields
   - Maintain foreign key relationships and constraints

3. **Update Entity Framework Models**
   - Remove email properties from `Contact` entity (`ALP.Data/Models/Contacts/Contact.cs`)
   - Add `ContactEmails` navigation property to Contact entity
   - Update `ContactConfiguration.cs` to remove email constraints and add relationship mapping
   - Create new `ContactEmail` entity with status enumeration

#### Phase 2: Core Business Logic Updates

4. **Update Critical Business Logic Components**
   - **Client.Email Property**: Modify computed property to query new email table and implement status-based routing logic
   - **Invoice Services**: Update all invoice email selection patterns in:
     - `SendFriendlyReminderEmail`, `SendFirstReminderEmail`, `SendSecondReminderEmail`
     - `SendInvoiceStatementEmail`, `SendInvoice`, `ReSendInvoice`
   - **Syntaq Integration**: Update email data provision for external service integration

5. **Email Association Process Updates**
   - **EmailSyncService.AssignEmailToContacts**: Modify to search across all email addresses with Active/Primary/Invoice/Dormant status
   - **Contact Matching Logic**: Update to exclude "Ignore" status emails from matching process
   - **Batch Processing**: Ensure ~20-minute batch process works with new email table structure

#### Phase 3: Data Transfer Objects (DTOs)

6. **Backend C# DTO Restructuring**
   - **ContactDto**: Replace individual email properties with email collection
   - **CreateContactInput**: Update to handle email collection with status designation
   - **UpdateContactInput**: Modify for email management operations
   - **Import DTOs**: Update CSV import mapping classes for new structure

7. **Frontend TypeScript DTO Updates**
   - **contact-dto.ts**: Replace email properties with email array structure
   - Update all TypeScript interfaces to handle email collections
   - Maintain backward compatibility during transition period

#### Phase 4: API Controllers & Endpoints

8. **Contact Management API Updates**
   - **ContactController**: Update all CRUD operations to handle email collections
   - **Contact Creation/Update**: Implement email status management in create/update endpoints
   - **Contact Search**: Modify search functionality to work with new email table structure
   - **Contact Import**: Update CSV import to handle new email structure

9. **Email-Related Controller Updates**
   - **ContactEmailController**: Ensure compatibility with new email association structure
   - **Email attachment handling**: Verify functionality with updated contact-email relationships
   - **Email export operations**: Update PDF export to work with new email structure

#### Phase 5: Frontend Vue.js Components

10. **Contact Management UI Redesign**
    - **ContactInfo.vue**: Replace three email input fields with dynamic email management interface
    - **CreateContact.vue**: Update contact creation form for new email input system
    - **Contact Tables**: Modify contact list displays to show primary email appropriately

11. **Contact Selector Updates**
    - **ContactSelector.vue**: Update display logic to show primary email in dropdown
    - **ContactSelectorNoCreate.vue**: Ensure proper email display in selection components
    - **MultiInputContactSelector.vue**: Update multi-select components to work with new structure

12. **Email Communication Component Updates**
    - **ContactEmails.vue**: Verify compatibility with new contact-email association structure
    - Update email viewer components to work with enhanced contact relationships

#### Phase 6: Import/Export & Data Processing

13. **Data Import/Export Updates**
    - **Contact Import** (`Contacts.cs`): Update legacy data import to use new email table
    - **CSV Import** (`ContactService.cs`): Modify CSV import mapping for new email structure
    - **Contact Export**: Update export functionality to include email status information

14. **Email Data Processing Updates**
    - **Email Import**: Ensure email import processes work with new contact-email associations
    - **Email Export**: Update contact email export functionality for new structure

#### Phase 7: External Integrations

15. **Third-Party Service Integration Updates**
    - **ActiveCampaign Integration**: Update marketing platform sync to use primary email from new structure
    - **Gravatar Integration**: Ensure profile picture generation uses correct primary email
    - **Microsoft 365**: Verify email sync compatibility with new contact structure

#### Phase 8: Validation & Business Rules

16. **Validation Logic Updates**
    - Implement email status validation rules
    - Update uniqueness constraint validation
    - Implement business rules for Primary/Invoice email requirements

#### Phase 9: Search & Filtering Operations

17. **Search Functionality Updates**
    - **Contact Search**: Update email-based search to query across all email addresses
    - **Contact Filtering**: Modify filter operations for new email structure
    - **Organisation Contact Lists**: Update email display in organisation-related views

#### Phase 10: Reporting & Analytics

18. **Reporting System Updates**
    - **Contact Reports**: Update reports to use new email table structure
    - **Communication Analytics**: Ensure email tracking works with new associations
    - **Invoice Reporting**: Update invoice delivery tracking for new email routing logic

#### Phase 11: Testing & Rollback Preparation

19. **Comprehensive Testing Strategy**
    - **Unit Tests**: Update all unit tests for modified components
    - **Integration Tests**: Test email association process end-to-end
    - **UI Tests**: Verify all frontend components work with new email structure
    - **Performance Tests**: Ensure email association performance remains acceptable

20. **Rollback Capability**
    - **Deprecate old email fields** (keep temporarily for rollback capability)
    - Maintain data synchronization between old and new structures during transition
    - Prepare rollback scripts and procedures
    - Monitor system performance and data integrity post-migration

**Migration Benefit**: The existing email validation and input masking infrastructure can be leveraged, minimizing data cleanup requirements during the migration to the new email address table structure.

## Similar Development

We have done a similar functionality update to the phone numbers, where we have broken out a single phone number in the contact table to a one-to-many relationship in a separate phone number table.

## Success Metrics & Acceptance Criteria

### Key Performance Indicators

#### User Experience Metrics
- **Reduced Support Tickets**: Decrease in user complaints about "missing" email history by 80%+
- **Email History Completeness**: Increase in email associations per contact (baseline: current 3 emails → target: average 4-6 emails including historical)
- **User Adoption**: 90% of active users utilizing new email status features within 3 months
- **User Satisfaction**: Positive feedback on unified communication history visibility

#### System Performance Metrics
- **Email Association Performance**: Batch processing time remains within acceptable limits (≤ current ~1 hour frequency)
- **Database Performance**: Contact detail page load times remain ≤ current performance
- **Migration Success**: 100% data migration without loss or corruption
- **System Stability**: No degradation in overall system performance post-implementation

#### Business Value Metrics
- **Legal Case Management**: Improved client communication tracking completeness
- **Client Service Quality**: Faster access to complete communication history
- **Compliance**: Better email retention for regulatory requirements
- **Business Efficiency**: Reduced time spent searching for historical communications

### Acceptance Criteria

#### Data Migration Requirements
- [ ] All existing email data migrated without loss (Primary → Primary, Secondary → Active, Invoice → Invoice)
- [ ] Pre-migration duplicate validation completed with resolution plan
- [ ] Null/empty email fields properly handled (ignored during migration)
- [ ] Data integrity validation confirms all contacts maintain appropriate email relationships

#### Email Association Process
- [ ] Batch email matching process successfully searches across all email statuses (Primary, Invoice, Active, Dormant)
- [ ] Ignore status emails completely excluded from matching process
- [ ] Dormant emails continue to participate in email association
- [ ] Existing deduplication logic remains functional (first TO recipient wins)
- [ ] Performance monitoring confirms no material degradation in batch processing

#### User Interface Functionality
- [ ] Communications box UI accommodates multiple email addresses with scrolling
- [ ] Add email functionality using "+" button works correctly
- [ ] Inline delete functionality works for all non-Primary emails
- [ ] Inline dropdown status changes work for Active, Dormant, Ignore
- [ ] Primary/Invoice status locked in dropdown (cannot be changed directly)
- [ ] Automatic demotion works when new Primary/Invoice email is set
- [ ] Primary email addresses have no delete functionality (no delete button displayed)
- [ ] Primary email removal guidance dialog works correctly when users attempt deletion
- [ ] First email automatically set to Primary status  
- [ ] Maximum 6 email display with scrolling for additional addresses
- [ ] Email address deletion preserves existing email associations with contact
- [ ] Deleted email addresses excluded from future email association batch processing
- [ ] Deletion confirmation dialog provides clear guidance on deletion vs status change options
- [ ] Audit trail captures email address deletions with user, timestamp, and reason

#### Code Integration Requirements
- [ ] All identified codebase references updated to use new email table structure:

**Core Business Logic Integration:**
  - [ ] **Client.Email Property** (`ALP.Data/Models/Clients/Client.cs` lines 59-67): Update computed property to implement status-based email routing logic
  - [ ] **Invoice Services** - Update all email selection patterns:
    - [ ] `SendFriendlyReminderEmail` in `InvoiceService.cs`
    - [ ] `SendFirstReminderEmail` in `InvoiceService.cs` 
    - [ ] `SendSecondReminderEmail` in `InvoiceService.cs`
    - [ ] `SendInvoiceStatementEmail` in `InvoiceService.cs`
    - [ ] `SendInvoice` in `InvoiceService.cs`
    - [ ] `ReSendInvoice` in `InvoiceService.cs`
  - [ ] **Syntaq Integration** (`SyntaqDataService.cs` line 1206): Update external service email data provision

**Email Association Process Integration:**
  - [ ] **EmailSyncService.AssignEmailToContacts** (`EmailSyncService.cs` lines 127-146): Update to search across all active email statuses
  - [ ] **Contact Matching Logic**: Modify to exclude "Ignore" status emails from matching
  - [ ] **Batch Processing** (`HangfireSyncUserEmailsJob.cs`): Ensure ~20-minute email download process works with new structure

**Entity Framework & Database Integration:**
  - [ ] **Contact Entity** (`ALP.Data/Models/Contacts/Contact.cs` lines 48-56): Remove email properties, add ContactEmails navigation
  - [ ] **Database Configuration** (`ContactConfiguration.cs` lines 10-11): Remove email constraints, add relationship mapping
  - [ ] **Migration Files**: Create new migrations for ContactEmails table

**DTO & API Integration:**
  - [ ] **Backend C# DTOs**: 
    - [ ] `ContactDto.cs` (lines 11-15): Replace email properties with email collection
    - [ ] `CreateContactInput.cs` (lines 13-19): Update for email status management  
    - [ ] `UpdateContactInput.cs` (lines 11-17): Modify for email operations
  - [ ] **Frontend TypeScript DTOs**:
    - [ ] `contact-dto.ts` (lines 32-34): Replace email properties with array structure
    - [ ] All contact interfaces updated for email collections
  - [ ] **API Controllers**:
    - [ ] `ContactController.cs`: Update all CRUD operations for email collections
    - [ ] `ContactEmailController.cs`: Ensure compatibility with new associations

**Frontend Vue.js Component Integration:**
  - [ ] **Contact Management UI**:
    - [ ] `ContactInfo.vue` (lines 82-120): Replace three email fields with dynamic email management
    - [ ] `CreateContact.vue` (lines 84-120): Update creation form for new email system
    - [ ] `Contacts.vue`: Modify contact lists to show primary email appropriately
  - [ ] **Contact Selector Components**:
    - [ ] `ContactSelector.vue` (lines 14-16): Update display logic for primary email
    - [ ] `ContactSelectorNoCreate.vue`: Ensure proper email display
    - [ ] `MultiInputContactSelector.vue`: Update for new structure
  - [ ] **Email Communication Components**:
    - [ ] `ContactEmails.vue`: Verify compatibility with new contact-email associations

**Import/Export Integration:**
  - [ ] **Contact Import/Export**:
    - [ ] `Contacts.cs` (lines 90-95, 154-160): Update legacy import for new email table
    - [ ] `ContactService.cs` (lines 1343-1348): Modify CSV import mapping
    - [ ] Contact export functionality updated for email status
  - [ ] **Email Data Processing**:
    - [ ] `Emails.cs`: Update email import processes for new associations
    - [ ] `EmailService.cs` (lines 1264-1296): Update contact email export

**External Integration Updates:**
  - [ ] **ActiveCampaign Integration** (`ActiveCampaignService.cs` lines 59-62): Update to use primary email from new structure
  - [ ] **Gravatar Integration**: Ensure profile picture generation uses correct primary email
  - [ ] **Microsoft 365 Integration**: Verify email sync compatibility

**Search & Reporting Integration:**
  - [ ] **Contact Search** (`ContactService.cs` line 1142): Update email-based search for new table
  - [ ] **Contact Filtering**: Modify filter operations for new email structure
  - [ ] **Contact Reports**: Update reports to use new email table structure
  - [ ] **Communication Analytics**: Ensure email tracking works with new associations
  - [ ] **Invoice Reporting**: Update delivery tracking for new email routing

**Validation & Business Rules Integration:**
  - [ ] Email status validation rules implemented
  - [ ] Uniqueness constraint validation updated
  - [ ] Primary/Invoice email business rules enforced

#### Validation & Error Handling
- [ ] Email format validation blocks invalid emails from saving
- [ ] Uniqueness validation prevents duplicate email addresses across contacts
- [ ] Appropriate error messages for all edge cases
- [ ] Graceful handling of contact creation without email addresses

#### Business Logic Preservation
- [ ] Invoice email routing logic functions correctly (Invoice status → Primary fallback)
- [ ] Email communication history displays correctly for all email statuses
- [ ] Contact search and filtering works with new email structure
- [ ] External integrations continue to function with updated email logic

### Success Definition

**The enhancement is considered successful when:**

1. **Complete Data Preservation**: All historical email data successfully migrated with no loss of email communication history
2. **Functional Equivalence**: All existing email-related functionality works identically to current system
3. **Enhanced Capability**: Users can successfully manage multiple email addresses per contact with appropriate status designations
4. **Performance Maintained**: No degradation in system performance for email processing or UI responsiveness  
5. **User Adoption**: Clear evidence of users actively utilizing the new multi-email capabilities
6. **Business Value Delivered**: Demonstrated improvement in communication history completeness and user workflow efficiency

### Rollback Criteria

**Implementation should be rolled back if:**
- Any data loss or corruption occurs during migration
- System performance degrades beyond acceptable limits
- Critical business functionality breaks (especially invoice email routing)
- Email association process fails or significantly slows down
- User workflow becomes materially more complex than current system

## Appendix: Critical Codebase Analysis Findings

### Documentation Corrections Made

During comprehensive codebase analysis of the ALP Portal system, several critical discoveries were made that required corrections to this specification:

#### **Email Processing Frequency Correction**
- **Original Assumption**: Email processing every ~20-30 minutes
- **Actual Implementation**: Email processing every **1 hour** via Hangfire scheduled job
- **Code Location**: `ALP/Startup.cs` line 190: `Cron.HourInterval(1)`
- **Impact**: Adjusted performance expectations and system load calculations

#### **Email Matching Logic Correction**
- **Original Assumption**: Email matching against all 3 email fields (Primary, Secondary, Invoice)
- **Actual Implementation**: Email matching **only against single `Email` field** (Primary field only)
- **Code Location**: `ALP.Services/Emails/EmailSyncService.cs` lines 135-155
- **Discovery**: Secondary and Invoice email fields are completely ignored in current matching process
- **Impact**: This makes the multi-email enhancement even more critical than initially understood

#### **System Architecture Correction**
- **Original Assumption**: Email processing runs on separate server
- **Actual Implementation**: Email processing runs as **Hangfire background jobs within main Portal application**
- **Code Location**: `ALP.Services/Hangfire/HangfireSyncUserEmailsJob.cs`
- **Impact**: Simplified deployment and operational complexity for enhancement

#### **Database Relationship Correction**
- **Original Assumption**: Direct `contact_id` foreign key in emails table
- **Actual Implementation**: **Many-to-many relationship** via `ContactEmails` junction table
- **Discovery**: Emails can be associated with multiple contacts simultaneously
- **Impact**: Confirmed enhancement approach aligns with existing architecture

### Key Technical Insights

#### **Exact Code Location for Multi-Email Enhancement**
The critical method requiring update for multi-email support:
```csharp
// Current implementation (EmailSyncService.cs lines 135-155)
public async Task AssignEmailToContacts(Email email, IEnumerable<EmailRecipient> recipients)
{
    // Only searches: _context.Contacts.Where(e => emailAddresses.Contains(e.Email.Trim().ToUpper()))
    // Missing: Secondary and Invoice email fields completely ignored
}
```

#### **Hangfire Job Configuration**
```csharp
// Startup.cs line 190
RecurringJob.AddOrUpdate<HangfireSyncUserEmailsJob>(
    x => x.Execute(), 
    Cron.HourInterval(1), 
    new RecurringJobOptions { TimeZone = TimeZoneInfo.Local }
);
```

#### **Performance Characteristics Discovered**
- **Batch Size**: 10 emails per Microsoft Graph API call
- **Processing Method**: Sequential user processing, parallel email processing within user
- **Error Handling**: Individual email failures don't stop batch processing
- **Monitoring**: Available via `/hangfire` dashboard

### Implications for Enhancement Implementation

These findings confirm that:

1. **Enhancement is More Critical**: Current system misses even more emails than initially understood
2. **Implementation Scope**: No separate server deployment needed - all code is in main application
3. **Performance Impact**: 1-hour frequency provides sufficient processing window for new email table queries
4. **Architectural Alignment**: New multi-email table structure aligns perfectly with existing many-to-many relationship pattern

### Documentation Created

- **`EMAIL_PROCESSING_SYSTEM.md`**: Comprehensive documentation of current email processing system in `alp-business-logic` submodule
- **Specification Updates**: All timing, architecture, and processing details corrected based on actual codebase analysis

