# 13. Local Scopes

> **Goal:** Eliminate duplicated query logic by encapsulating reusable query constraints inside Eloquent Models using Local Scopes.

---

# Overview

One of the most common problems in growing Laravel applications is **duplicated query conditions**.

Example:

```php
User::where('status', 'ACTIVE')
    ->where('email_verified_at', '!=', null)
    ->where('deleted_at', null)
    ->get();
```

A few days later...

```php
User::where('status', 'ACTIVE')
    ->where('email_verified_at', '!=', null)
    ->paginate();
```

Later...

```php
User::where('status', 'ACTIVE')
    ->where('email_verified_at', '!=', null)
    ->count();
```

The same conditions are duplicated throughout the application.

Laravel solves this problem with **Local Scopes**.

---

# What is a Local Scope?

A Local Scope is a reusable query method defined inside an Eloquent Model.

Instead of repeatedly writing:

```php
->where(...)
```

You create a named scope.

Example

```php
User::active()
    ->verified()
    ->get();
```

This produces code that is:

- Cleaner
- Reusable
- Easier to read
- Easier to maintain
- Easier to test

---

# Architecture

```text
Controller
      │
      ▼
Service
      │
      ▼
Model
      │
      ▼
Local Scope
      │
      ▼
Database
```

Local Scopes belong inside Models because they are part of the persistence layer.

---

# When to Use Local Scopes

Create a Local Scope whenever a query condition is:

- Repeated
- Business meaningful
- Easy to name
- Reusable

Good examples:

- Active users
- Approved loans
- Paid installments
- Overdue loans
- Verified customers
- Available products

---

# When NOT to Use Local Scopes

Avoid creating a Local Scope for:

- One-time queries
- Highly dynamic queries
- Business workflows
- HTTP-specific filtering

Example

```php
Loan::approve()
```

❌ Wrong

Loan approval is business logic.

It belongs in

```
LoanService
```

---

# Basic Local Scope

Example

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    public function scopeActive(
        Builder $query
    ): Builder {

        return $query->where(
            'status',
            'ACTIVE'
        );

    }
}
```

Usage

```php
User::active()->get();
```

Laravel automatically removes the `scope` prefix.

---

# Naming Convention

Method

```php
scopeActive()
```

Usage

```php
User::active();
```

Method

```php
scopeApproved()
```

Usage

```php
Loan::approved();
```

Always use descriptive names.

---

# Parameterized Scopes

Scopes may accept parameters.

Example

```php
public function scopeRole(
    Builder $query,
    string $role
): Builder {

    return $query->where(
        'role',
        $role
    );

}
```

Usage

```php
User::role('ADMIN')
    ->get();
```

---

# Multiple Parameters

Example

```php
public function scopeBetweenDates(

    Builder $query,

    string $start,

    string $end

): Builder {

    return $query->whereBetween(

        'created_at',

        [$start, $end]

    );

}
```

Usage

```php
Loan::betweenDates(
    '2025-01-01',
    '2025-12-31'
);
```

---

# Chaining Scopes

Scopes are chainable.

```php
User::active()
    ->verified()
    ->role('ADMIN')
    ->latest()
    ->paginate();
```

This is one of the biggest advantages of Local Scopes.

---

# Example Model

```php
class Loan extends Model
{
    public function scopeApproved(
        Builder $query
    ): Builder {

        return $query->where(
            'status',
            LoanStatus::APPROVED
        );

    }

    public function scopePending(
        Builder $query
    ): Builder {

        return $query->where(
            'status',
            LoanStatus::PENDING
        );

    }

    public function scopeOverdue(
        Builder $query
    ): Builder {

        return $query->where(

            'due_date',

            '<',

            now()

        )

        ->where(

            'status',

            LoanStatus::UNPAID

        );

    }
}
```

Usage

```php
Loan::approved()->get();

Loan::pending()->count();

Loan::overdue()->paginate();
```

---

# Dynamic Filtering

Instead of

```php
if ($status) {

    $query->where(
        'status',
        $status
    );

}
```

Create

```php
public function scopeStatus(

    Builder $query,

    ?string $status

): Builder {

    return $query->when(

        $status,

        fn ($query) =>

            $query->where(
                'status',
                $status
            )

    );

}
```

Usage

```php
Loan::status(
    request('status')
);
```

---

# Search Scope

Example

```php
public function scopeSearch(

    Builder $query,

    ?string $search

): Builder {

    return $query->when(

        $search,

        function ($query) use ($search) {

            $query->where(function ($query) use ($search) {

                $query->where(

                    'name',

                    'LIKE',

                    "%{$search}%"

                )

                ->orWhere(

                    'email',

                    'LIKE',

                    "%{$search}%"

                );

            });

        }

    );

}
```

Usage

```php
User::search(
    request('search')
);
```

---

# Sorting Scope

```php
public function scopeLatestFirst(
    Builder $query
): Builder {

    return $query->latest(
        'created_at'
    );

}
```

Usage

```php
User::latestFirst()
    ->paginate();
```

---

# Active + Verified Example

Without Scopes

```php
User::where(
    'status',
    'ACTIVE'
)

->whereNotNull(
    'email_verified_at'
)

->whereNull(
    'deleted_at'
)

->get();
```

With Scopes

```php
User::active()

->verified()

->get();
```

Much easier to read.

---

# Service Example

```php
public function activeUsers()
{
    return User::active()

        ->verified()

        ->paginate();
}
```

Notice how the Service reads almost like English.

---

# Controller Example

```php
public function index()
{
    return UserResource::collection(

        $this->service->activeUsers()

    );
}
```

The Controller knows nothing about query details.

---

# Relationship Scopes

Scopes work perfectly with relationships.

Example

```php
Customer::with([

    'loans' => fn ($query) =>

        $query->approved()

]);
```

---

# Combining Scopes

```php
Loan::approved()

    ->overdue()

    ->latest()

    ->paginate();
```

Each scope remains focused.

---

# Local Scope Diagram

```mermaid
flowchart TD

A[Controller]

A --> B[Service]

B --> C[Loan::approved()]

C --> D[Loan::overdue()]

D --> E[(Database)]
```

---

# Scopes vs Services

| Local Scope | Service |
|--------------|----------|
| Filters queries | Executes business logic |
| Reusable SQL | Business workflows |
| Persistence | Domain operations |
| Query constraints | Transactions |
| Read operations | Write operations |

---

# Scopes vs Repository

Repositories retrieve data.

Scopes define reusable query constraints.

Repository

```php
findCustomer()
```

Scope

```php
active()

verified()

premium()
```

They complement each other.

---

# Performance

Good

```php
User::active()

    ->verified()

    ->paginate();
```

Laravel generates the same SQL as manually writing the `where` clauses.

There is no performance penalty.

---

# Best Practices

- Keep scopes small.
- One responsibility per scope.
- Use meaningful names.
- Always return the Builder instance.
- Use enums where possible.
- Prefer scopes over duplicated query conditions.
- Chain scopes instead of creating large scope methods.
- Keep business logic out of scopes.

---

# Common Anti-Patterns

Avoid

```php
scopeApproveLoan()
```

This performs a business operation.

Use

```
LoanService
```

instead.

Avoid

```php
Mail::send(...)
```

inside scopes.

Avoid

```php
Notification::send(...)
```

inside scopes.

Avoid

```php
Http::post(...)
```

inside scopes.

Avoid modifying records.

Scopes should only build queries.

---

# Testing Scopes

Example

```php
it('returns approved loans', function () {

    Loan::factory()->approved()->count(3)->create();

    Loan::factory()->pending()->count(2)->create();

    expect(

        Loan::approved()->count()

    )->toBe(3);

});
```

Scopes should have dedicated tests.

---

# Local Scope Checklist

- [ ] Scope has one responsibility
- [ ] Returns Builder
- [ ] Uses descriptive name
- [ ] No business logic
- [ ] No database updates
- [ ] No notifications
- [ ] No HTTP logic
- [ ] Reused across Services
- [ ] Covered by tests
- [ ] Improves readability

---

# Recommended Common Scopes

Most enterprise applications benefit from common scopes such as:

```php
active()

inactive()

verified()

approved()

pending()

rejected()

paid()

unpaid()

overdue()

latestFirst()

oldestFirst()

search()

status()

betweenDates()
```

Keep them generic and reusable.

---

# Summary

Local Scopes are one of Laravel's most powerful features for creating expressive, reusable database queries. By encapsulating common query constraints inside Models, you eliminate duplication, improve readability, and keep Services focused on business logic rather than SQL construction.

A well-designed application uses Local Scopes to describe **what** data is needed while Services describe **what business process** should happen with that data.

The next chapter explores **Eloquent Relationships**, covering one-to-one, one-to-many, many-to-many, polymorphic relationships, eager loading, lazy loading, and performance optimization.