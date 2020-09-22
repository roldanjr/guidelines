# Guidelines

- [Guidelines](#guidelines)
- [1. Visual Studio Code packages](#1-visual-studio-code-packages)
- [2. Modular](#2-modular)
- [3. Naming conventions](#3-naming-conventions)
- [4. Action classes pattern](#4-action-classes-pattern)
- [5. Form requests](#5-form-requests)
- [6. Models](#6-models)
  - [6.1 Scopes](#61-scopes)
  - [6.2 Relationships](#62-relationships)
- [7. Routes](#7-routes)
  - [7.1 Files](#71-files)
  - [7.2 Naming](#72-naming)
- [8. Don't repeat yourself (DRY)](#8-dont-repeat-yourself-dry)
- [9. General rules](#9-general-rules)
- [10. References](#10-references)

# 1. Visual Studio Code packages

Install the following packages:

- php-cs-fixer
- PHP Intelephense
- ESLint
- Prettier
- GitLens
- PHP DocBlocker

# 2. Modular

Files and folders must be segregated into groups and modules.

Example:

- `app\Http\Controllers\Staff\UserController`
- `app\Http\Controllers\Public\HomeController`
- `app\Http\Controllers\Ajax\UserController`
- `app\Http\Controllers\Api\UserController`

# 3. Naming conventions

| What               | How                  | Implemention              | Location                                                   | Other name rule                              |
| ------------------ | -------------------- | ------------------------- | ---------------------------------------------------------- | -------------------------------------------- |
| Controller         | PascalCase, singular | UserController            | app\Http\Controllers\Staff\UserController                  | Should based on model                        |
| Model              | PascalCase, singular | User                      | app\Models\User                                            |                                              |
| Action class       | PascalCase           | CreateUser, LoginUser     | app\Services\CreateUser                                    | Should be a very specific action name        |
| Form request       | PascalCase, singular | UserStoreRequest          | app\Http\Requests\UserStoreRequest                         | Should based on model                        |
| Trait              | PascalCase           | HasDatatable              | app\Traits\HasDatatable                                    | Should be very specific to avoid duplication |
| View main blade    | kebab-case           | show.blade.php            | templates\staff\users\show.blade.php                       | Should be descriptive                        |
| View partial blade | kebab-case           | contact-details.blade.php | templates\staff\users\partials\form.blade.php              | Should be descriptive                        |
| Component          | kebab-case           | accordion-item.vue        | resources\js\staff\components\accordion\accordion-item.vue | Should be very specific to avoid duplication |
| Lang               | plural, kebab-case   | tables.php                | resources\lang\en\staff\tables.php                         | Should based on section or group             |
| Assets             | kebab-case           | style.scss                | resources\assets\staff\scss\style.scss                     |                                              |
| Scss               | kebab-case           | style.scss                | resources\css\staff\style.scss                             |                                              |
| Script             | kebab-case           | sample-name-script.js     | resources\js\staff\pages\users\sample-name-script.js       | Should be very specific to avoid duplication |

# 4. Action classes pattern

Use action classes for bussiness logic codes rather than controllers.  
path: `app\Services\{ModuleName}`

Example:

```php
class GetUserByEmail
{
    /**
     * Get user by email
     *
     * @param string $email
     * @return App\Models\User
     */
    public function execute(string $email)
    {
        return User::where('email', $email)->first();
    }
}
```

# 5. Form requests

Use form request instead of validating per controller methods.  
path: `app\Http\Requests\{ModuleName}`

- 1 form request per item (create/update)
- Messages should be translated

Example:

```php
class GetUserByEmail
{
    /**
     * Get user by email
     *
     * @param string $email
     * @return App\Models\User
     */
    public function execute(string $email)
    {
        return User::where('email', $email)->first();
    }
}
```

# 6. Models

## 6.1 Scopes

Use scopes for reusability

```php
public function scopeActive($q)
{
    return $q->where('is_active', 1);
}
```

## 6.2 Relationships

- names should be descriptive
- proper grammer
  - `one-to-one` - singular
  - `one-to-many` - plural

# 7. Routes

## 7.1 Files

Proper segregation of route files

- `routes/staff.php`
- `routes/staff_ajax.php`
- `routes/public.php`
- `routes/public_ajax.php`
- `routes/api.php`

## 7.2 Naming

- public-facing urls must use kebab-case
- use `.` as a group separator
- use `_` as a name separator

Example:  
`Route::get('/group-companies', 'GroupCompanyController@index')->name('staff.group_companies.index')`

# 8. Don't repeat yourself (DRY)

A class and a method should have only one responsibility so that it is readable and helps us to avoid code duplication.

```php
public function getArticles()
{
    return $this->whereHas('user', function ($q) {
            $q->active();
        })->get();
}
```

# 9. General rules

- Proper indentions (per **_4 spaces_**)
- Proper whitespace between codes
- Proper singular/plural naming
- Remove unused codes
- Comment your codes using **_Docblock_**
- File/class names should be **_DESCRIPTIVE_**
- Method names should be **_DESCRIPTIVE_**
- All static texts/labels **_MUST_** be always translated
- **_DO NOT_** put logical codes in controllers, blades, and helpers

# 10. References

- https://github.com/alexeymezenin/laravel-best-practices
- https://guidelines.spatie.be/
