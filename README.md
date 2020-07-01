# Coding Guidelines <!-- omit in toc -->

- [1. Visual Studio Code Packages](#1-visual-studio-code-packages)
- [2. Coding Specifications](#2-coding-specifications)
  - [2.1. Single responsibility principle](#21-single-responsibility-principle)
  - [2.2. Fat models, skinny controllers](#22-fat-models-skinny-controllers)
  - [2.3. Validation](#23-validation)
  - [2.4. Business logic should be in service class](#24-business-logic-should-be-in-service-class)
  - [2.5. Don't repeat yourself (DRY)](#25-dont-repeat-yourself-dry)
  - [2.6. Prefer to use Eloquent over using Query Builder and raw SQL queries. Prefer collections over arrays](#26-prefer-to-use-eloquent-over-using-query-builder-and-raw-sql-queries-prefer-collections-over-arrays)
  - [2.7. Mass assignment](#27-mass-assignment)
  - [2.8. Do not execute queries in Blade templates and use eager loading (N + 1 problem)](#28-do-not-execute-queries-in-blade-templates-and-use-eager-loading-n--1-problem)
  - [2.9. Comment your code, but prefer descriptive method and variable names over comments](#29-comment-your-code-but-prefer-descriptive-method-and-variable-names-over-comments)
  - [2.10. Use config and language files, constants instead of text in the code.](#210-use-config-and-language-files-constants-instead-of-text-in-the-code)
  - [2.11. Follow Laravel naming conventions](#211-follow-laravel-naming-conventions)
  - [2.12. Use shorter and more readable syntax where possible](#212-use-shorter-and-more-readable-syntax-where-possible)
  - [2.13. Use IoC container or facades instead of new Class](#213-use-ioc-container-or-facades-instead-of-new-class)
  - [2.14. Do not get data from the .env file directly](#214-do-not-get-data-from-the-env-file-directly)
  - [2.15. Use Single action class pattern](#215-use-single-action-class-pattern)
- [3. Coding Style Guide](#3-coding-style-guide)
  - [3.1. Strings](#31-strings)
  - [3.2. Ternary operators](#32-ternary-operators)
  - [3.3. If statements](#33-if-statements)
      - [3.3.0.1. HAPPY PATH](#3301-happy-path)
      - [3.3.0.2. AVOID ELSE](#3302-avoid-else)
      - [3.3.0.3. COMPOUND IFS](#3303-compound-ifs)
  - [3.4. Whitespace](#34-whitespace)
  - [3.5. Configuration](#35-configuration)
  - [3.6. Artisan commands](#36-artisan-commands)
  - [3.7. Routing](#37-routing)
  - [3.8. Views](#38-views)
  - [3.9. Blade Templates](#39-blade-templates)
  - [3.10. Authorization](#310-authorization)
  - [3.11. Translations](#311-translations)
  - [3.12. Naming Classes](#312-naming-classes)
      - [3.12.0.1. CONTROLLERS](#31201-controllers)
      - [3.12.0.2. RESOURCES (AND TRANSFORMERS)](#31202-resources-and-transformers)
      - [3.12.0.3. JOBS](#31203-jobs)
      - [3.12.0.4. EVENTS](#31204-events)
      - [3.12.0.5. LISTENERS](#31205-listeners)
      - [3.12.0.6. COMMANDS](#31206-commands)
      - [3.12.0.7. MAILABLES](#31207-mailables)
  - [3.13. File Structure](#313-file-structure)
  - [3.14. General](#314-general)
  - [3.15. Databases](#315-databases)
  - [3.16. Controllers](#316-controllers)
  - [3.17. Models](#317-models)
  - [Services](#services)
  - [3.18. Modules](#318-modules)

# 1. Visual Studio Code Packages
Install the following packages
 - php-cs-fixer
 - PHP Intelephense
 - ESLint
 - Prettier
 - GitLens
 - php-docblock

# 2. Coding Specifications

## 2.1. Single responsibility principle
A class and a method should have only one responsibility.
```
public function getFullNameAttribute()
{
    return $this->isVerifiedClient() ? $this->getFullNameLong() : $this->getFullNameShort();
}

public function isVerifiedClient()
{
    return auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified();
}

public function getFullNameLong()
{
    return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
}

public function getFullNameShort()
{
    return $this->first_name[0] . '. ' . $this->last_name;
}
```

## 2.2. Fat models, skinny controllers
Put all DB related logic into Eloquent models or into Repository classes if you're using Query Builder or raw SQL queries
```
public function index()
{
    return view('index', ['clients' => $this->client->getWithNewOrders()]);
}

class Client extends Model
{
    public function getWithNewOrders()
    {
        return $this->verified()
            ->with(['orders' => function ($q) {
                $q->where('created_at', '>', Carbon::today()->subWeek());
            }])
            ->get();
    }
}
```

## 2.3. Validation
- Move validation from controllers to Request classes. 
- When using multiple rules for one field in a form request, avoid using |, always use array notation. Using an array notation will make it easier to apply custom rule classes to a field.
```
public function store(PostRequest $request)
{    
    ....
}

class PostRequest extends Request
{
    public function rules()
    {
        return [
            'title' => ['required', 'unique:posts', 'max:255'],
            'body' => 'required',
            'publish_at' => ['nullable', 'date'],
        ];
    }
}
```
- All custom validation rules must use snake_case:
```
Validator::extend('organisation_type', function ($attribute, $value) {
    return OrganisationType::isValid($value);
});
```

## 2.4. Business logic should be in service class
A controller must have only one responsibility, so move business logic from controllers to service classes.
```
public function store(Request $request)
{
    $this->articleService->handleUploadedImage($request->file('image'));

    ....
}

class ArticleService
{
    public function handleUploadedImage($image)
    {
        if (!is_null($image)) {
            $image->move(public_path('images') . 'temp');
        }
    }
}
```

## 2.5. Don't repeat yourself (DRY)
Reuse code when you can. SRP is helping you to avoid duplication. Also, reuse Blade templates, use Eloquent scopes etc.
```
public function scopeActive($q)
{
    return $q->where('verified', 1)->whereNotNull('deleted_at');
}

public function getActive()
{
    return $this->active()->get();
}

public function getArticles()
{
    return $this->whereHas('user', function ($q) {
            $q->active();
        })->get();
}
```

## 2.6. Prefer to use Eloquent over using Query Builder and raw SQL queries. Prefer collections over arrays
Eloquent allows you to write readable and maintainable code. Also, Eloquent has great built-in tools like soft deletes, events, scopes etc.
```
    Article::has('user.profile')->verified()->latest()->get();
```

## 2.7. Mass assignment
```
    $category->article()->create($request->validated());
```

## 2.8. Do not execute queries in Blade templates and use eager loading (N + 1 problem)
Bad (for 100 users, 101 DB queries will be executed):
```
$users = User::with('profile')->get();

...

@foreach ($users as $user)
    {{ $user->profile->name }}
@endforeach
```

## 2.9. Comment your code, but prefer descriptive method and variable names over comments
 - Comment your code, but prefer descriptive method and variable names over comments.
 - Don't use docblocks for methods that can be fully type hinted (unless you need a description).
 - Only add a description when it provides more context than the method signature itself. Use full sentences for descriptions, including a period at the end.
```
// Good
class Url
{
    public static function fromString(string $url): Url
    {
        // ...
    }
}

```
 - Always use fully qualified class names in docblocks.
```
/**
 * @param string $url
 *
 * @return \Spatie\Url\Url
 */
```
 - Docblocks for class variables are required, as there's currently no other way to typehint these.
```
class Foo
{
    /** @var \Spatie\Url\Url */
    private $url;

    /** @var string */
    private $name;
}
```
 - When possible, docblocks should be written on one line.
```
/** @var string */
/** @test */
```
 - If a variable has multiple types, the most common occurring type should be first.
```
/** @var \Spatie\Goo\Bar|null */
```

## 2.10. Use config and language files, constants instead of text in the code.
```
public function isNormal()
{
    return $article->type === Article::TYPE_NORMAL;
}

return back()->with('message', __('app.article_added'));
```

## 2.11. Follow Laravel naming conventions
| What | How | Implemention |
|------|-----|-----------|
| Controller | singular | ArticleController |
| Route | plural | articles/1 |
| Named route | snake_case with dot notation | users.show_active |
| Model | singular | User |
| hasOne or belongsTo relationship | singular | articleComment |
| All other relationships | plural | articleComments |
| Table | plural | article_comments |
| Pivot table | singular model names in alphabetical order | article_user |
| Table column | snake_case without model name | meta_title |
| Model property | snake_case | $model->created_at |
| Foreign key | singular model name with _id suffix | article_id |
| Migration | - | 2020_01_01_000000_create_articles_table |
| Method | camelCase | getAll |
| Method in resource controller | table | store |
| Method in test class | camelCase | testGuestCannotSeeArticle |
| Variable | camelCase | $articlesWithAuthor |
| Collection | descriptive, plural | $activeUsers = User::active()->get() |
| Object | descriptive, singular | $activeUser = User::active()->first() |
| Config and language files index | snake_case | articles_enabled |
| View | kebab-case | show-filtered.blade.php |
| Config | snake_case | google_calendar.php |
| Contract (interface) | adjective or noun | Authenticatable |
| Trait | adjective | Notifiable |


## 2.12. Use shorter and more readable syntax where possible
```
session('cart');
$request->name;
```

## 2.13. Use IoC container or facades instead of new Class
new Class syntax creates tight coupling between classes and complicates testing. Use IoC container or facades instead.
```
public function __construct(User $user)
{
    $this->user = $user;
}

....

$this->user->create($request->validated());
```

## 2.14. Do not get data from the .env file directly
```
// config/api.php
'key' => env('API_KEY'),

// Use the data
$apiKey = config('api.key');
```

## 2.15. Use Single action class pattern
- The main purpose of using this is to separate the application’s business logic and also to avoid direct access to the data. You may refer the full documentation [here](https://medium.com/@remi_collin/keeping-your-laravel-applications-dry-with-single-action-classes-6a950ec54d1d)
  - File/class names should be descriptive
  - Method names should be descriptive


# 3. Coding Style Guide

## 3.1. Strings
When possible prefer string interpolation above sprintf and the . operator.
```
$greeting = "Hi, I am {$name}.";
```

## 3.2. Ternary operators
Every portion of a ternary expression should be on its own line unless it's a really short expression.
```
$result = $object instanceof Model
    ? $object->name
    : 'A default value';

$name = $isFoo ? 'foo' : 'bar';
```

## 3.3. If statements
#### 3.3.0.1. HAPPY PATH
  ```
  if (! $goodCondition) {
      throw new Exception;
  }

  //do something
  ```

#### 3.3.0.2. AVOID ELSE
 In general, else should be avoided because it makes code less readable. In most cases it can be refactored using early returns. This will also cause the happy path to go last, which is desirable.
  ```
  if (! $conditionBA) {
      // conditionB A failed
      
      return;
  }

  if (! $conditionB) {
      // conditionB A passed, B failed
      
      return;
  }

  // condition A and B passed
  ```

#### 3.3.0.3. COMPOUND IFS
In general, separate if statements should be preferred over a compound condition. This makes debugging code easier.
```
if (! $conditionA) {
   return;
}

if (! $conditionB) {
   return;
}

if (! $conditionC) {
   return;
}

// do stuff
```

## 3.4. Whitespace
Statements should have to breathe. In general always add blank lines between statements, unless they're a sequence of single-line equivalent operations. This isn't something enforceable, it's a matter of what looks best in its context.
```
public function getPage($url)
{
    $page = $this->pages()->where('slug', $url)->first();

    if (! $page) {
        return null;
    }

    if ($page['private'] && ! Auth::check()) {
        return null;
    }

    return $page;
}
```
```
// Good: A sequence of single-line equivalent operations.
public function up()
{
    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
        $table->string('name');
        $table->string('email')->unique();
        $table->string('password');
        $table->rememberToken();
        $table->timestamps();
    });
}
```
 - Don't add any extra empty lines between {} brackets.
```
if ($foo) {
    $this->foo = $foo;
}
```

## 3.5. Configuration
Configuration files must use kebab-case.
```
config/
  pdf-generator.php
```

## 3.6. Artisan commands
- The names given to artisan commands should all be kebab-cased.
```
php artisan delete-old-records
```

- A command should always give some feedback on what the result is. Minimally you should let the handle method spit out a comment at the end indicating that all went well.
```
public function handle()
{
    // do some work

    $this->comment('All ok!');
}
```
If possible use a descriptive success message eg. `Old records deleted.`

## 3.7. Routing
Public-facing urls must use kebab-case.
```
https://devversions.com/project/open-source
https://devversions.com/project/jobs/front-end-developer
```
- Route names must use camelCase.
```
Route::get('open-source', 'OpenSourceController@index')->name('openSource');
```
```
<a href="{{ route('openSource') }}">
    Open Source
</a>
```
- All routes have an http verb, that's why we like to put the verb first when defining a route. It makes a group of routes very readable. Any other route options should come after it.
```
// good: all http verbs come first
Route::get('/', 'HomeController@index')->name('home');
Route::get('open-source', 'OpenSourceController@index')->middleware('openSource');
```
- Route parameters should use camelCase.
```
Route::get('news/{newsItem}', 'NewsItemsController@index');
```
- A route url should not start with / unless the url would be an empty string.
```
Route::get('/', 'HomeController@index');
Route::get('open-source', 'OpenSourceController@index');
```
- Try to keep controllers simple and stick to the default CRUD keywords (`index`, `create`, `store`, `show`, `edit`, `update`, `destroy`). Extract a new controller if you need other actions.
- Separate route files if needed (public, admin, api, etc.)
- Proper groupings of routes to avoid confusion
- Proper naming of routes. Do not forget to put the group and modules as prefix. Resource/module name should be plural since it refers to all.
  > Ex:
    > - admin.countries.{action}
    > - admin.ajax.staff.{action}
    > - public.home
    > - public.ajax.countries.{action}

- Check if the route already exists before creating a new one to avoid redundancy.
- Use get and post properly.
- Create a different session middleware for admin panel and public page


## 3.8. Views
- Move blade files from views to templates folder
```
app/
resource/
templates/
...
```
- View files must use camelCase.
```
templates/
  admin/
    openSource.blade.php
```
```
class OpenSourceController
{
    public function index() {
        return view('openSource');
    }
}
```

## 3.9. Blade Templates
- Indent using four spaces.
```
<a href="/open-source">
    Open Source
</a>
```
- Don't add spaces after control structures.
```
@if($condition)
    Something
@endif
```

## 3.10. Authorization
- Policies must use camelCase.
```
Gate::define('editPost', function ($user, $post) {
    return $user->id == $post->user_id;
});
```
```
@can('editPost', $post)
    <a href="{{ route('posts.edit', $post) }}">
        Edit
    </a>
@endcan
```
- Try to name abilities using default CRUD words. One exception: replace show with view. A server `shows` a resource, a user `views` it.

## 3.11. Translations
Translations must be rendered with the `__` function. We prefer using this over `@lang` in Blade views because `__` can be used in both Blade views and regular PHP code. Here's an example:
```
<h2>{{ __('newsletter.form.title') }}</h2>

{!! __('newsletter.form.description') !!}
```

## 3.12. Naming Classes
Naming things is often seen as one of the harder things in programming. That's why we've established some high level guidelines for naming classes.

#### 3.12.0.1. CONTROLLERS
- Controllers should be in singular case, no spacing between words, and end with "Controller".Also, each word should be capitalised (i.e. BlogController, not blogcontroller).
- **DO NOT** put logical codes on controllers, use [single action classes](https://medium.com/@remi_collin/keeping-your-laravel-applications-dry-with-single-action-classes-6a950ec54d1d) instead. 
- Keep codes on controller skinny.
- Use a separate controller folder for Ajax and Api.
- Use Composer for global variables rather than inserting/attaching it to on every controller’s return view.
- You may also refer [here](#follow-laravel-naming-conventions).


#### 3.12.0.2. RESOURCES (AND TRANSFORMERS)
Both Eloquent resources and Fractal transformers are plural resources suffixed with `Resource` or `Transformer` accordingly. This is to avoid naming collisions with models.

#### 3.12.0.3. JOBS
A job's name should describe its action.

E.g. `CreateUser` or `PerformDatabaseCleanup`

#### 3.12.0.4. EVENTS
Events will often be fired before or after the actual event. This should be very clear by the tense used in their name.

E.g. `ApprovingLoan` before the action is completed and `LoanApproved` after the action is completed.

#### 3.12.0.5. LISTENERS
Listeners will perform an action based on an incoming event. Their name should reflect that action with a `Listener` suffix. This might seem strange at first but will avoid naming collisions with jobs.

E.g. `SendInvitationMailListener`

#### 3.12.0.6. COMMANDS
To avoid naming collisions we'll suffix commands with `Command`, so they are easiliy distinguisable from jobs.

e.g. `PublishScheduledPostsCommand`

#### 3.12.0.7. MAILABLES
Again to avoid naming collisions we'll suffix mailables with `Mail`, as they're often used to convey an event, action or question.

e.g. `AccountActivatedMail` or `NewEventMail`


## 3.13. File Structure
- Development files should only be under `resources` folder, not under `public` folder.
- Proper grouping of development files
  - `assets` – theme files
  - `css/scss` – developer’s stylesheets
  - `js` – developer’s scripts
- Segregate files according to modules and sections
    ```
    app/Http/Controllers/Admin/User

    /* Admin is the Section name */
    /* User is the module name */

    ```

## 3.14. General
- Remove unused codes
- There should be only two compiled files for stylesheets and 
scripts.
    - common – applicable on all pages
    - per page – applicable only on specific page
- **DO NOT** put logical codes on helpers
- Method name should be descriptive.
- Use `Service Layer Design Pattern`
- Use `single action class` in defining data services.
- Business logic will be in services

## 3.15. Databases
- Follow [naming convention](#follow-laravel-naming-conventions).
- Use doctrine/dbal
- In preparation for unit testing, avoid using of sql raw queries because dbal will almost do the work in order to ensure migrations will work on different data source.
- Modifications on table that has enum will not work and thus recommended to re-create the table.

## 3.16. Controllers
- Refer [here](#31201-controllers)

## 3.17. Models
- Proper assignment of relationships especially one-to-one and one-to-many. Do not assign a one-to-one relationship that should be a one-to-many because it might confuse other developers and the codes are a little bit different.
- Proper naming of relationships. You may also refer [here](#follow-laravel-naming-conventions).
    - Singular - one to one
    - Plural – one to many
- For pivot models, extend them as Pivot rather than Model.
- Use scopes for model-specific logic so that it’s flexible and can be reused.
- Additional attributes should be a general data
- User Service pattern with single action class. You can find the detailed explanation [here](https://medium.com/@remi_collin/keeping-your-laravel-applications-dry-with-single-action-classes-6a950ec54d1d).
  - Put the Service classes in `app/Services/{module-name}` folder

## Services
- Implement `Service Layer Design Pattern`
- Use `Single Action Class` in defining data services
- Class actions will be in `app\Services\{module-name}`

## 3.18. Modules
- Module will be in folders
- Module folder name will be `plural`


