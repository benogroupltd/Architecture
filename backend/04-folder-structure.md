# 04. Folder Structure

> **Goal:** Establish a consistent project structure that improves maintainability, discoverability, scalability, and team collaboration.

---

# Overview

A well-organized folder structure is one of the most important aspects of a maintainable Laravel application.

As a project grows, hundreds of controllers, models, services, jobs, and notifications are added. Without a consistent structure, developers spend more time searching for files than building features.

This documentation defines the recommended directory layout for enterprise Laravel applications.

---

# Design Principles

The folder structure is designed around the following principles:

- Single Responsibility Principle (SRP)
- Separation of Concerns
- Feature Independence
- High Cohesion
- Low Coupling
- Predictable Organization
- Easy Navigation
- Scalability

---

# Recommended Project Structure

```text
project/
│
├── app/
├── bootstrap/
├── config/
├── database/
├── public/
├── resources/
├── routes/
├── storage/
├── tests/
├── vendor/
└── artisan
```

---

# The `app/` Directory

The majority of application code lives inside the `app/` directory.

```text
app/
│
├── Console/
├── Enums/
├── Events/
├── Exceptions/
├── Helpers/
├── Http/
├── Jobs/
├── Listeners/
├── Mail/
├── Models/
├── Notifications/
├── Observers/
├── Policies/
├── Providers/
├── Services/
├── Traits/
├── ViewModels/
└── Rules/
```

---

# Complete Enterprise Folder Structure

```text
app/
│
├── Console/
│   └── Commands/
│
├── Enums/
│
├── Events/
│
├── Exceptions/
│
├── Helpers/
│
├── Http/
│   ├── Controllers/
│   │
│   ├── Middleware/
│   │
│   ├── Requests/
│   │
│   └── Resources/
│
├── Jobs/
│
├── Listeners/
│
├── Mail/
│
├── Models/
│
├── Notifications/
│
├── Observers/
│
├── Policies/
│
├── Providers/
│
├── Rules/
│
├── Services/
│
├── Traits/
│
└── ViewModels/
```

---

# Directory Responsibilities

## Console

```text
app/Console/
```

Contains Artisan commands and scheduled tasks.

Example

```text
Console/
└── Commands/
    ├── ImportCustomers.php
    ├── GenerateReports.php
    └── SyncProducts.php
```

Use for:

- Custom Artisan commands
- Scheduled jobs
- Batch processing

---

# Enums

```text
app/Enums/
```

Contains application enumerations.

Example

```text
Enums/

LoanStatus.php

PaymentStatus.php

UserRole.php

PermissionEnum.php
```

Example

```php
enum LoanStatus: string
{
    case PENDING='pending';

    case APPROVED='approved';

    case REJECTED='rejected';
}
```

---

# Events

```text
app/Events/
```

Contains application events.

Example

```text
UserRegistered.php

LoanApproved.php

PaymentReceived.php
```

Events should represent something that has already happened.

---

# Exceptions

```text
app/Exceptions/
```

Contains business-specific exceptions.

Example

```text
LoanApprovalException.php

PaymentException.php

InsufficientBalanceException.php
```

Avoid throwing generic exceptions for business rules.

---

# Helpers

```text
app/Helpers/
```

Contains stateless utility classes.

Example

```text
CurrencyHelper.php

PhoneHelper.php

DateHelper.php

ImageHelper.php
```

Helpers should never:

- Access the database
- Execute business logic
- Send notifications

---

# Http

```text
app/Http/
```

Contains everything related to HTTP communication.

---

## Controllers

```text
app/Http/Controllers/
```

Example

```text
API/

Admin/

Auth/

Web/
```

Controllers should only:

- Receive requests
- Call Services
- Return responses

---

## Middleware

```text
app/Http/Middleware/
```

Example

```text
Authenticate.php

CheckPermission.php

SetLocale.php

EnsureTenant.php
```

---

## Requests

```text
app/Http/Requests/
```

Contains Form Requests.

Example

```text
StoreUserRequest.php

UpdateLoanRequest.php

LoginRequest.php
```

---

## Resources

```text
app/Http/Resources/
```

Contains API Resources.

Example

```text
UserResource.php

LoanResource.php

CustomerCollection.php
```

---

# Jobs

```text
app/Jobs/
```

Contains queued jobs.

Example

```text
SendWelcomeEmail.php

GenerateLoanSchedule.php

ExportCustomers.php
```

Jobs should perform one responsibility.

---

# Listeners

```text
app/Listeners/
```

Contains event listeners.

Example

```text
SendWelcomeEmailListener.php

CreateAuditLogListener.php

SyncCRMListener.php
```

---

# Mail

```text
app/Mail/
```

Contains Mailables.

Example

```text
WelcomeMail.php

InvoiceMail.php

PasswordResetMail.php
```

---

# Models

```text
app/Models/
```

Contains Eloquent models.

Example

```text
User.php

Loan.php

Payment.php

Customer.php
```

Models should contain:

- Relationships
- Casts
- Accessors
- Mutators
- Local Scopes

---

# Notifications

```text
app/Notifications/
```

Contains notification classes.

Example

```text
LoanApprovedNotification.php

WelcomeNotification.php

PaymentReminderNotification.php
```

---

# Observers

```text
app/Observers/
```

Contains model observers.

Example

```text
UserObserver.php

LoanObserver.php
```

Observers respond to model lifecycle events.

---

# Policies

```text
app/Policies/
```

Contains authorization policies.

Example

```text
LoanPolicy.php

UserPolicy.php

CustomerPolicy.php
```

---

# Providers

```text
app/Providers/
```

Contains Service Providers.

Example

```text
AppServiceProvider.php

AuthServiceProvider.php

RouteServiceProvider.php
```

---

# Rules

```text
app/Rules/
```

Contains reusable validation rules.

Example

```text
ValidPhoneNumber.php

StrongPassword.php
```

---

# Services

```text
app/Services/
```

Contains business logic.

Example

```text
LoanService.php

UserService.php

PaymentService.php

InventoryService.php
```

A service should represent a business domain.

---

# Traits

```text
app/Traits/
```

Contains reusable behaviors.

Example

```text
HasUuid.php

HasAudit.php

LogsActivity.php
```

Traits should remain focused on reusable behavior.

---

# ViewModels

```text
app/ViewModels/
```

Contains classes that prepare complex data for Blade views.

Example

```text
DashboardViewModel.php

LoanSummaryViewModel.php
```

---

# Other Important Directories

## config/

```text
config/
```

Application configuration.

Example

```text
app.php

cache.php

database.php

queue.php
```

Never hardcode configuration values.

Use

```php
config('app.name');
```

instead.

---

# database/

```text
database/

migrations/

seeders/

factories/
```

Contains database-related files.

---

# public/

Contains publicly accessible assets.

Example

```text
index.php

images/

storage/

favicon.ico
```

Never place sensitive files here.

---

# resources/

```text
resources/

views/

css/

js/

lang/
```

Contains frontend resources.

---

# routes/

```text
routes/

web.php

api.php

console.php

channels.php
```

Keep routes organized by domain when the application grows.

Example

```text
routes/

api/

admin/

customer/

loan/
```

---

# storage/

Contains:

- Logs
- Cache
- Uploaded files
- Framework cache
- Sessions

Never commit generated storage files to version control.

---

# tests/

```text
tests/

Feature/

Unit/
```

Feature tests verify complete application behavior.

Unit tests verify isolated classes.

---

# Naming Conventions

## Controllers

```
UserController
LoanController
PaymentController
```

---

## Services

```
UserService
LoanService
PaymentService
```

---

## Policies

```
UserPolicy
LoanPolicy
```

---

## Jobs

```
GenerateReportJob
SyncCustomerJob
SendInvoiceJob
```

---

## Events

```
LoanApproved
UserRegistered
OrderCompleted
```

---

## Listeners

```
SendWelcomeEmail
CreateAuditLog
NotifyAdministrator
```

---

## Requests

```
StoreUserRequest

UpdateLoanRequest

LoginRequest
```

---

## Resources

```
UserResource

LoanResource

CustomerCollection
```

---

# Feature Growth Example

As the application grows, folders should remain organized.

```text
app/

Services/
    UserService.php
    LoanService.php
    PaymentService.php

Models/
    User.php
    Loan.php
    Payment.php

Policies/
    UserPolicy.php
    LoanPolicy.php

Jobs/
    ExportLoans.php
    GenerateStatements.php

Notifications/
    LoanReminderNotification.php
```

Every new feature should follow the same organizational pattern.

---

# Best Practices

- Group files by responsibility.
- Use meaningful directory names.
- Keep directories focused.
- One class per file.
- Follow PSR-4 autoloading.
- Avoid dumping unrelated classes into generic folders.
- Keep Services business-oriented.
- Keep Helpers stateless.
- Keep Models persistence-focused.
- Keep Controllers thin.

---

# Common Anti-Patterns

Avoid:

```text
Helpers/
    LoanHelper.php
```

if it contains business logic.

Avoid:

```text
Controllers/

LoanController.php

(500+ lines)
```

Avoid:

```text
Models/

Loan.php

(send emails)

(call APIs)

(generate reports)
```

Avoid:

```text
Misc/

Utils/

Common/

Stuff/
```

Generic folders quickly become disorganized.

---

# Folder Structure Checklist

- [ ] Services separated from Controllers
- [ ] Validation inside Form Requests
- [ ] Policies used for authorization
- [ ] Jobs handle background processing
- [ ] Events and Listeners decouple side effects
- [ ] Helpers remain stateless
- [ ] Traits encapsulate reusable behavior
- [ ] One class per file
- [ ] PSR-4 naming conventions followed
- [ ] No business logic inside Controllers or Helpers

---

# Summary

A consistent folder structure is the foundation of a scalable Laravel application. Every class should have a clear, predictable location, allowing developers to quickly find, understand, and extend the codebase. By organizing files according to their responsibilities, the project remains maintainable as it grows from a small application into a large enterprise system.

The next chapter explores **Routing**, including route organization, versioning, naming conventions, route model binding, and best practices for API and web applications.