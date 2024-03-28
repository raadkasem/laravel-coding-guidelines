# laravel-coding-guidelines
Welcome to Laravel Coding Guidelines! This repository contains a comprehensive guide to help developers follow best practices for writing clean, maintainable, and consistent code within Laravel projects.



## **1. Follow PSR Standards**
Follow the [PSR-2 coding style](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) to ensure your code is consistent and readable.

Also, follow naming conventions accepted by Laravel community:

What | How | Good | Bad
------------ | ------------- | ------------- | -------------
Controller | singular | ArticleController | ~~ArticlesController~~
Route | plural | articles/1 | ~~article/1~~
Route name | snake_case with dot notation | users.show_active | ~~users.show-active, show-active-users~~
Model | singular | User | ~~Users~~
hasOne or belongsTo relationship | singular | articleComment | ~~articleComments, article_comment~~
All other relationships | plural | articleComments | ~~articleComment, article_comments~~
Table | plural | article_comments | ~~article_comment, articleComments~~
Pivot table | singular model names in alphabetical order | article_user | ~~user_article, articles_users~~
Table column | snake_case without model name | meta_title | ~~MetaTitle; article_meta_title~~
Model property | snake_case | $model->created_at | ~~$model->createdAt~~
Foreign key | singular model name with _id suffix | article_id | ~~ArticleId, id_article, articles_id~~
Primary key | - | id | ~~custom_id~~
Migration | - | 2017_01_01_000000_create_articles_table | ~~2017_01_01_000000_articles~~
Method | camelCase | getAll | ~~get_all~~
Method in resource controller | [table](https://laravel.com/docs/master/controllers#resource-controllers) | store | ~~saveArticle~~
Method in test class | camelCase | testGuestCannotSeeArticle | ~~test_guest_cannot_see_article~~
Variable | camelCase | $articlesWithAuthor | ~~$articles_with_author~~
Collection | descriptive, plural | $activeUsers = User::active()->get() | ~~$active, $data~~
Object | descriptive, singular | $activeUser = User::active()->first() | ~~$users, $obj~~
Config and language files index | snake_case | articles_enabled | ~~ArticlesEnabled; articles-enabled~~
View | kebab-case | show-filtered.blade.php | ~~showFiltered.blade.php, show_filtered.blade.php~~
Config | snake_case | google_calendar.php | ~~googleCalendar.php, google-calendar.php~~
Contract (interface) | adjective or noun | AuthenticationInterface | ~~Authenticatable, IAuthentication~~
Trait | adjective | Notifiable | ~~NotificationTrait~~
Trait [(PSR)](https://www.php-fig.org/bylaws/psr-naming-conventions/) | adjective | NotifiableTrait | ~~Notification~~
Enum | singular | UserType | ~~UserTypes~~, ~~UserTypeEnum~~
FormRequest | singular | UpdateUserRequest | ~~UpdateUserFormRequest~~, ~~UserFormRequest~~, ~~UserRequest~~
Seeder | singular | UserSeeder | ~~UsersSeeder~~

<br>

## **2. Use Meaningful Variable and Function Names**
Clear and descriptive names enhance code readability

## **3. Leverage Laravel’s Eloquent ORM**
Use Eloquent for database operations. It provides an expressive and powerful way to interact with your database :

```
// Good
$users = User::where('active', true)
             ->orderBy('name', 'asc')
             ->take(10)
             ->get();

// Bad
$users = DB::table('users')
            ->where('active', 1)
            ->orderBy('name', 'asc')
            ->take(10)
            ->get();
```

## **4. Keep Routes Well organized**
Organize your routes to keep them manageable:

```
// Good
Route::prefix('admin')->group(function () {
    Route::get('dashboard', 'AdminController@dashboard');
    Route::get('users', 'AdminController@users');
});

// Bad
Route::get('admin-dashboard', 'AdminController@dashboard');
Route::get('admin-users', 'AdminController@users');
```

## **5. Break Down Complex Logic with Services**
Organize complex business logic using services :

```
// Good
class OrderService
{
    public function calculateTotal(Order $order)
    {
        // ...
    }

    public function processOrder(Order $order)
    {
        // ...
    }
}

// Bad
class OrderController extends Controller
{
  public function placeOrder(Request $request)
    {
        // Complex logic here
    }
}
```

## **6. Implement Caching Where Appropriate**
Use caching to improve performance :

```
// Good
$products = Cache::remember('products', 60, function () {
    return Product::all();
});

// Bad
$products = Product::all();
```

## **7. Fat models, skinny controllers**

```
// Good
public function index()
{
    return view('index', ['clients' => $this->client->getWithNewOrders()]);
}

class Client extends Model
{
    public function getWithNewOrders(): Collection
    {
        return $this->verified()
            ->with(['orders' => function ($q) {
                $q->where('created_at', '>', Carbon::today()->subWeek());
            }])
            ->get();
    }
}
```
```
// Bad
public function index()
{
    $clients = Client::verified()
        ->with(['orders' => function ($q) {
            $q->where('created_at', '>', Carbon::today()->subWeek());
        }])
        ->get();

    return view('index', ['clients' => $clients]);
}
```


## **8. Reuse code (Don't repeat yourself)**
Reuse code when you can. SRP is helping you to avoid duplication. Also, reuse Blade templates, use Eloquent scopes etc.

## **9. Mass assignment**
 the $fillable property on Eloquent models protects against mass assignment vulnerabilities


## **10. Prefer to use Eloquent**
 over using Query Builder and raw SQL queries. Prefer collections over arrays



## **11. Use shorter and more readable syntax where possible**

Bad:

```php
$request->session()->get('cart');
$request->input('name');
```

Good:

```php
session('cart');
$request->name;
```

More examples:

Common syntax | Shorter and more readable syntax
------------ | -------------
`Session::get('cart')` | `session('cart')`
`$request->session()->get('cart')` | `session('cart')`
`Session::put('cart', $data)` | `session(['cart' => $data])`
`$request->input('name'), Request::get('name')` | `$request->name, request('name')`
`return Redirect::back()` | `return back()`
`is_null($object->relation) ? null : $object->relation->id` | `optional($object->relation)->id` (in PHP 8: `$object->relation?->id`)
`return view('index')->with('title', $title)->with('client', $client)` | `return view('index', compact('title', 'client'))`
`$request->has('value') ? $request->value : 'default';` | `$request->get('value', 'default')`
`Carbon::now(), Carbon::today()` | `now(), today()`
`App::make('Class')` | `app('Class')`
`->where('column', '=', 1)` | `->where('column', 1)`
`->orderBy('created_at', 'desc')` | `->latest()`
`->orderBy('age', 'desc')` | `->latest('age')`
`->orderBy('created_at', 'asc')` | `->oldest()`
`->select('id', 'name')->get()` | `->get(['id', 'name'])`
`->first()->name` | `->value('name')`

<br>

## **12. Validation**
Move validation from controllers to Request classes.

```
// Bad
public function store(Request $request)
{
    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

    ...
}

// Good

public function store(PostRequest $request)
{
    ...
}

class PostRequest extends Request
{
    public function rules(): array
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
            'publish_at' => 'nullable|date',
        ];
    }
}

```

<br>

## **13. Use IoC / Service container instead of new Class**

new Class syntax creates tight coupling between classes and complicates testing. Use IoC container or facades instead.

Bad:

```php
$user = new User;
$user->create($request->validated());
```

Good:

```php
public function __construct(User $user)
{
    $this->user = $user;
}

...

$this->user->create($request->validated());
```
<br>

## **14. Do not get data from the `.env` file directly**

Pass the data to config files instead and then use the `config()` helper function to use the data in an application.

Bad:

```php
$apiKey = env('API_KEY');
```

Good:

```php
// config/api.php
'key' => env('API_KEY'),

// Use the data
$apiKey = config('api.key');
```

## **15. Write Unit and Feature Tests**

```
// Good
public function testOrderTotalCalculation()
{
    $order = factory(Order::class)->create();
    $order->items()->create(['price' => 50, 'quantity' => 2]);

    $total = (new OrderService)->calculateTotal($order);

    $this->assertEquals(100, $total);
}

// Bad
// No tests
```

# Other good practices

- Never put any logic in routes files.
- Do not override standard framework features to avoid problems related to updating the framework version and many other issues.
- Do not use DocBlocks, DocBlocks reduce readability. Use a descriptive method name and modern PHP features like return type hints instead.



# Articles
- [Laravel Best Practice-Coding Standards Part 01](https://dev.to/lathindu1/laravel-best-practice-coding-standards-part-01-304l)
- [Laravel Best Practice-Coding Standards Part 02](https://dev.to/lathindu1/laravel-best-practice-coding-standards-part-02-a40)
- [Laravel & PHP Guidelines](https://spatie.be/guidelines/laravel-php)
