# Coding Guidelines <!-- omit in toc -->

- [Visual Studio Code Packages](#visual-studio-code-packages)
- [Coding Specifications](#coding-specifications)
  - [Single responsibility principle](#single-responsibility-principle)
  - [Fat models, skinny controllers](#fat-models-skinny-controllers)
  - [Validation](#validation)
  - [Business logic should be in service class](#business-logic-should-be-in-service-class)
  - [Don't repeat yourself (DRY)](#dont-repeat-yourself-dry)
  - [Prefer to use Eloquent over using Query Builder and raw SQL queries. Prefer collections over arrays](#prefer-to-use-eloquent-over-using-query-builder-and-raw-sql-queries-prefer-collections-over-arrays)
  - [Mass assignment](#mass-assignment)
  - [Do not execute queries in Blade templates and use eager loading (N + 1 problem)](#do-not-execute-queries-in-blade-templates-and-use-eager-loading-n--1-problem)
  - [Comment your code, but prefer descriptive method and variable names over comments](#comment-your-code-but-prefer-descriptive-method-and-variable-names-over-comments)
  - [Use config and language files, constants instead of text in the code.](#use-config-and-language-files-constants-instead-of-text-in-the-code)
  - [Follow Laravel naming conventions](#follow-laravel-naming-conventions)
  - [Use shorter and more readable syntax where possible](#use-shorter-and-more-readable-syntax-where-possible)
  - [Use IoC container or facades instead of new Class](#use-ioc-container-or-facades-instead-of-new-class)
  - [Do not get data from the .env file directly](#do-not-get-data-from-the-env-file-directly)
- [Coding Style Guide](#coding-style-guide)
  - [Strings](#strings)
  - [Ternary operators](#ternary-operators)
  - [If statements](#if-statements)
      - [HAPPY PATH**](#happy-path)
      - [AVOID ELSE](#avoid-else)
      - [COMPOUND IFS](#compound-ifs)
  - [Whitespace](#whitespace)
  - [Configuration](#configuration)
  - [Artisan commands](#artisan-commands)
  - [Routing](#routing)
  - [Controllers](#controllers)
  - [Views](#views)
  - [Blade Templates](#blade-templates)
  - [Authorization](#authorization)
  - [Translations](#translations)
  - [Naming Classes](#naming-classes)
      - [CONTROLLERS](#controllers-1)
      - [RESOURCES (AND TRANSFORMERS)](#resources-and-transformers)
      - [JOBS](#jobs)
      - [EVENTS](#events)
      - [LISTENERS](#listeners)
      - [COMMANDS](#commands)
      - [MAILABLES](#mailables)
  - [File Structure](#file-structure)

# Visual Studio Code Packages
Install the following packages
 - php-cs-fixer
 - PHP Intelephense
 - ESLint
 - Prettier
 - GitLens
 - php-dockblock

# Coding Specifications

## Single responsibility principle
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

## Fat models, skinny controllers
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

## Validation
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

## Business logic should be in service class
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

## Don't repeat yourself (DRY)
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

## Prefer to use Eloquent over using Query Builder and raw SQL queries. Prefer collections over arrays
Eloquent allows you to write readable and maintainable code. Also, Eloquent has great built-in tools like soft deletes, events, scopes etc.
```
    Article::has('user.profile')->verified()->latest()->get();
```

## Mass assignment
```
    $category->article()->create($request->validated());
```

## Do not execute queries in Blade templates and use eager loading (N + 1 problem)
Bad (for 100 users, 101 DB queries will be executed):
```
$users = User::with('profile')->get();

...

@foreach ($users as $user)
    {{ $user->profile->name }}
@endforeach
```

## Comment your code, but prefer descriptive method and variable names over comments
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

## Use config and language files, constants instead of text in the code.
```
public function isNormal()
{
    return $article->type === Article::TYPE_NORMAL;
}

return back()->with('message', __('app.article_added'));
```

## Follow Laravel naming conventions
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


## Use shorter and more readable syntax where possible
```
session('cart');
$request->name;
```

## Use IoC container or facades instead of new Class
new Class syntax creates tight coupling between classes and complicates testing. Use IoC container or facades instead.
```
public function __construct(User $user)
{
    $this->user = $user;
}

....

$this->user->create($request->validated());
```

## Do not get data from the .env file directly
```
// config/api.php
'key' => env('API_KEY'),

// Use the data
$apiKey = config('api.key');
```

# Coding Style Guide

## Strings
When possible prefer string interpolation above sprintf and the . operator.
```
$greeting = "Hi, I am {$name}.";
```

## Ternary operators
Every portion of a ternary expression should be on its own line unless it's a really short expression.
```
$result = $object instanceof Model
    ? $object->name
    : 'A default value';

$name = $isFoo ? 'foo' : 'bar';
```

## If statements
#### HAPPY PATH**
  ```
  if (! $goodCondition) {
      throw new Exception;
  }

  //do something
  ```

#### AVOID ELSE
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

#### COMPOUND IFS
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

## Whitespace
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

## Configuration
Configuration files must use kebab-case.
```
config/
  pdf-generator.php
```

## Artisan commands
The names given to artisan commands should all be kebab-cased.
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

## Routing
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

## Controllers
Controllers that control a resource must use the plural resource name.
```
class PostsController
{
    // ...
}
```

Try to keep controllers simple and stick to the default CRUD keywords (`index`, `create`, `store`, `show`, `edit`, `update`, `destroy`). Extract a new controller if you need other actions.

## Views
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

## Blade Templates
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

## Authorization
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

## Translations
Translations must be rendered with the `__` function. We prefer using this over `@lang` in Blade views because `__` can be used in both Blade views and regular PHP code. Here's an example:
```
<h2>{{ __('newsletter.form.title') }}</h2>

{!! __('newsletter.form.description') !!}
```

## Naming Classes
Naming things is often seen as one of the harder things in programming. That's why we've established some high level guidelines for naming classes.

#### CONTROLLERS
Generally controllers are named by the plural form of their corresponding resource and a `Controller` suffix. This is to avoid naming collisions with models that are often equally named.

e.g. `UsersController` or `EventDaysController`

When writing non-resourceful controllers you might come across invokable controllers that perform a single action. These can be named by the action they perform again suffixed by Controller.

e.g. `PerformCleanupController`

#### RESOURCES (AND TRANSFORMERS)
Both Eloquent resources and Fractal transformers are plural resources suffixed with `Resource` or `Transformer` accordingly. This is to avoid naming collisions with models.

#### JOBS
A job's name should describe its action.

E.g. `CreateUser` or `PerformDatabaseCleanup`

#### EVENTS
Events will often be fired before or after the actual event. This should be very clear by the tense used in their name.

E.g. `ApprovingLoan` before the action is completed and `LoanApproved` after the action is completed.

#### LISTENERS
Listeners will perform an action based on an incoming event. Their name should reflect that action with a `Listener` suffix. This might seem strange at first but will avoid naming collisions with jobs.

E.g. `SendInvitationMailListener`

#### COMMANDS
To avoid naming collisions we'll suffix commands with `Command`, so they are easiliy distinguisable from jobs.

e.g. `PublishScheduledPostsCommand`

#### MAILABLES
Again to avoid naming collisions we'll suffix mailables with `Mail`, as they're often used to convey an event, action or question.

e.g. `AccountActivatedMail` or `NewEventMail`


## File Structure
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


