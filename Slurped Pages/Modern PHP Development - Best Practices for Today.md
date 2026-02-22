---
link: https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f
byline: Sepehr Mohseni
site: DEV Community
date: 2026-01-04T14:51
excerpt: PHP has evolved dramatically in recent years. PHP 8.x brings powerful features that enable cleaner,...
twitter: https://twitter.com/@
tags:
  - php
  - best_pratices
slurped: 2026-02-08T10:06
title: "Modern PHP Development: Best Practices for Today"
---
PHP has evolved dramatically in recent years. PHP 8.x brings powerful features that enable cleaner, more performant code. This guide covers modern PHP practices that every developer should know.

## [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#php-83-84-and-85-features-you-should-use)PHP 8.3, 8.4, and 8.5 Features You Should Use

### [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#property-hooks-php-84)Property Hooks (PHP 8.4)

```
<?php
class User
{
    public string $name {
        // Custom getter logic
        get => trim($this->name ?? '');
        // Custom setter logic
        set {
            if (strlen(value) < 3) {
                throw new InvalidArgumentException('Name too short');
            }
            $this->name = value;
        }
    }
}

$user = new User();
$user->name = '  John  '; // Setter trims and validates
echo $user->name; // 'John'
```

 

### [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#asymmetric-visibility-php-84)Asymmetric Visibility (PHP 8.4)

```
<?php
class Product
{
    public readonly string $id; // Public read, private set

    public function __construct(string $id)
    {
        $this->id = $id; // Allowed in class scope
    }
}

$product = new Product('abc123');
echo $product->id; // 'abc123'
// $product->id = 'xyz'; // Error: Cannot modify readonly property
```

 

### [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#pipe-operator-php-85)Pipe Operator (PHP 8.5)

```
<?php
$result = $input
    |> trim(?)
    |> strtolower(?)
    |> ucwords(?)
    |> str_replace(' ', '-', ?);

// Equivalent to:
// $result = str_replace(' ', '-', ucwords(strtolower(trim($input))));
```

 

### [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#constructor-property-promotion)Constructor Property Promotion

```
<?php
// Before PHP 8.0
class Product
{
    private string $name;
    private float $price;
    private int $stock;
    public function __construct(string $name, float $price, int $stock)
    {
        $this->name = $name;
        $this->price = $price;
        $this->stock = $stock;
    }
}
// PHP 8.0+ - Much cleaner!
class Product
{
    public function __construct(
        private string $name,
        private float $price,
        private int $stock = 0,
    ) {}
}
```

 

  
Constructor property promotion reduces boilerplate significantly. Combine with readonly for immutable value objects.  

### [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#named-arguments)Named Arguments

```
<?php
function createUser(
    string $name,
    string $email,
    ?string $phone = null,
    bool $newsletter = false,
    string $role = 'user',
): User {
    // ...
}
// Clear and self-documenting
$user = createUser(
    name: 'John Doe',
    email: 'john@example.com',
    newsletter: true,
    // Skip phone, use default role
);
```

 

### [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#match-expression)Match Expression

```
<?php
// Before: switch statement
function getStatusLabel(string $status): string
{
    switch ($status) {
        case 'pending':
            return 'Awaiting Review';
        case 'approved':
            return 'Approved';
        case 'rejected':
            return 'Rejected';
        default:
            return 'Unknown';
    }
}
// PHP 8.0+: match expression
function getStatusLabel(string $status): string
{
    return match ($status) {
        'pending' => 'Awaiting Review',
        'approved' => 'Approved',
        'rejected' => 'Rejected',
        default => 'Unknown',
    };
}
// Match with multiple conditions
function getDiscount(string $customerType): float
{
    return match ($customerType) {
        'vip', 'premium' => 0.20,
        'regular' => 0.10,
        'new' => 0.05,
        default => 0.0,
    };
}
```

 

### [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#enums)Enums

```
<?php
// Basic enum
enum Status: string
{
    case Pending = 'pending';
    case Approved = 'approved';
    case Rejected = 'rejected';
    public function label(): string
    {
        return match ($this) {
            self::Pending => 'Awaiting Review',
            self::Approved => 'Approved',
            self::Rejected => 'Rejected',
        };
    }
    public function color(): string
    {
        return match ($this) {
            self::Pending => 'yellow',
            self::Approved => 'green',
            self::Rejected => 'red',
        };
    }
}
// Usage
$status = Status::Pending;
echo $status->value; // 'pending'
echo $status->label(); // 'Awaiting Review'
// Type-safe function parameters
function updateOrderStatus(Order $order, Status $status): void
{
    $order->status = $status;
    $order->save();
}
```

 

  
Use backed enums (with string or int values) when you need to store values in a database or serialize to JSON.  

## [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#design-patterns-in-modern-php)Design Patterns in Modern PHP

### [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#repository-pattern)Repository Pattern

```
<?php
namespace App\Repositories;
use App\Models\User;
use Illuminate\Support\Collection;
interface UserRepositoryInterface
{
    public function find(int $id): ?User;
    public function findByEmail(string $email): ?User;
    public function all(): Collection;
    public function create(array $data): User;
    public function update(User $user, array $data): User;
    public function delete(User $user): bool;
}
class EloquentUserRepository implements UserRepositoryInterface
{
    public function __construct(
        private readonly User $model,
    ) {}
    public function find(int $id): ?User
    {
        return $this->model->find($id);
    }
    public function findByEmail(string $email): ?User
    {
        return $this->model->where('email', $email)->first();
    }
    public function all(): Collection
    {
        return $this->model->all();
    }
    public function create(array $data): User
    {
        return $this->model->create($data);
    }
    public function update(User $user, array $data): User
    {
        $user->update($data);
        return $user->fresh();
    }
    public function delete(User $user): bool
    {
        return $user->delete();
    }
}
```

 

### [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#service-layer-pattern)Service Layer Pattern

```
<?php
namespace App\Services;
use App\DTOs\CreateOrderDTO;
use App\Events\OrderCreated;
use App\Models\Order;
use App\Repositories\OrderRepositoryInterface;
use App\Repositories\ProductRepositoryInterface;
use Illuminate\Support\Facades\DB;
class OrderService
{
    public function __construct(
        private readonly OrderRepositoryInterface $orderRepository,
        private readonly ProductRepositoryInterface $productRepository,
        private readonly PaymentService $paymentService,
    ) {}
    public function createOrder(CreateOrderDTO $dto): Order
    {
        return DB::transaction(function () use ($dto) {
            // Validate stock availability
            foreach ($dto->items as $item) {
                $product = $this->productRepository->find($item->productId);

                if ($product->stock < $item->quantity) {
                    throw new InsufficientStockException($product);
                }
            }
            // Create order
            $order = $this->orderRepository->create([
                'user_id' => $dto->userId,
                'status' => OrderStatus::Pending,
                'total' => $this->calculateTotal($dto->items),
            ]);
            // Add items and update stock
            foreach ($dto->items as $item) {
                $order->items()->create([
                    'product_id' => $item->productId,
                    'quantity' => $item->quantity,
                    'price' => $item->price,
                ]);
                $this->productRepository->decrementStock(
                    $item->productId,
                    $item->quantity
                );
            }
            event(new OrderCreated($order));
            return $order;
        });
    }
}
```

 

### [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#data-transfer-objects-dtos)Data Transfer Objects (DTOs)

```
<?php
namespace App\DTOs;
use Illuminate\Http\Request;
readonly class CreateUserDTO
{
    public function __construct(
        public string $name,
        public string $email,
        public string $password,
        public ?string $phone = null,
        public bool $newsletter = false,
    ) {}
    public static function fromRequest(Request $request): self
    {
        return new self(
            name: $request->validated('name'),
            email: $request->validated('email'),
            password: $request->validated('password'),
            phone: $request->validated('phone'),
            newsletter: $request->boolean('newsletter'),
        );
    }
    public static function fromArray(array $data): self
    {
        return new self(
            name: $data['name'],
            email: $data['email'],
            password: $data['password'],
            phone: $data['phone'] ?? null,
            newsletter: $data['newsletter'] ?? false,
        );
    }
}
```

 

## [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#testing-best-practices)Testing Best Practices

### [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#unit-testing-with-phpunit)Unit Testing with PHPUnit

```
<?php
namespace Tests\Unit\Services;
use App\DTOs\CreateOrderDTO;
use App\Models\Order;
use App\Models\Product;
use App\Repositories\OrderRepositoryInterface;
use App\Repositories\ProductRepositoryInterface;
use App\Services\OrderService;
use App\Services\PaymentService;
use PHPUnit\Framework\TestCase;
use Mockery;
class OrderServiceTest extends TestCase
{
    private OrderService $service;
    private $orderRepository;
    private $productRepository;
    private $paymentService;
    protected function setUp(): void
    {
        parent::setUp();
        $this->orderRepository = Mockery::mock(OrderRepositoryInterface::class);
        $this->productRepository = Mockery::mock(ProductRepositoryInterface::class);
        $this->paymentService = Mockery::mock(PaymentService::class);
        $this->service = new OrderService(
            $this->orderRepository,
            $this->productRepository,
            $this->paymentService,
        );
    }
    public function test_creates_order_successfully(): void
    {
        // Arrange
        $product = new Product(['id' => 1, 'stock' => 10, 'price' => 99.99]);
        $order = new Order(['id' => 1, 'status' => 'pending']);
        $this->productRepository
            ->shouldReceive('find')
            ->with(1)
            ->andReturn($product);
        $this->orderRepository
            ->shouldReceive('create')
            ->once()
            ->andReturn($order);
        $dto = new CreateOrderDTO(
            userId: 1,
            items: [new OrderItemDTO(productId: 1, quantity: 2, price: 99.99)],
        );
        // Act
        $result = $this->service->createOrder($dto);
        // Assert
        $this->assertInstanceOf(Order::class, $result);
        $this->assertEquals('pending', $result->status);
    }
    public function test_throws_exception_when_insufficient_stock(): void
    {
        // Arrange
        $product = new Product(['id' => 1, 'stock' => 1]);
        $this->productRepository
            ->shouldReceive('find')
            ->andReturn($product);
        $dto = new CreateOrderDTO(
            userId: 1,
            items: [new OrderItemDTO(productId: 1, quantity: 5, price: 99.99)],
        );
        // Assert & Act
        $this->expectException(InsufficientStockException::class);
        $this->service->createOrder($dto);
    }
    protected function tearDown(): void
    {
        Mockery::close();
        parent::tearDown();
    }
}
```

 

### [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#feature-testing)Feature Testing

```
<?php
namespace Tests\Feature\Api;
use App\Models\User;
use App\Models\Product;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;
class OrderApiTest extends TestCase
{
    use RefreshDatabase;
    public function test_authenticated_user_can_create_order(): void
    {
        // Arrange
        $user = User::factory()->create();
        $product = Product::factory()->create(['stock' => 10, 'price' => 49.99]);
        // Act
        $response = $this->actingAs($user)
            ->postJson('/api/orders', [
                'items' => [
                    ['product_id' => $product->id, 'quantity' => 2],
                ],
            ]);
        // Assert
        $response->assertCreated()
            ->assertJsonStructure([
                'data' => ['id', 'status', 'total', 'items'],
            ]);
        $this->assertDatabaseHas('orders', [
            'user_id' => $user->id,
            'status' => 'pending',
        ]);
        $this->assertDatabaseHas('products', [
            'id' => $product->id,
            'stock' => 8, // 10 - 2
        ]);
    }
    public function test_unauthenticated_user_cannot_create_order(): void
    {
        $response = $this->postJson('/api/orders', [
            'items' => [],
        ]);
        $response->assertUnauthorized();
    }
}
```

 

## [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#performance-optimization)Performance Optimization

### [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#opcache-configuration)OPcache Configuration

```
; php.ini production settings
opcache.enable=1
opcache.memory_consumption=256
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=20000
opcache.validate_timestamps=0
opcache.revalidate_freq=0
opcache.save_comments=1
opcache.jit_buffer_size=256M
opcache.jit=1255
```

 

### [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#preloading)Preloading

```
<?php
// preload.php
require __DIR__ . '/vendor/autoload.php';
// Preload frequently used classes
$classesToPreload = [
    \App\Models\User::class,
    \App\Models\Product::class,
    \App\Models\Order::class,
    \App\Services\OrderService::class,
    \App\Repositories\EloquentUserRepository::class,
];
foreach ($classesToPreload as $class) {
    class_exists($class);
}
```

 

```
; php.ini
opcache.preload=/var/www/app/preload.php
opcache.preload_user=www-data
```

 

## [](https://dev.to/sepehr/modern-php-development-best-practices-for-today-322f#conclusion)Conclusion

Modern PHP is a powerful, elegant language when used correctly. By leveraging PHP 8.x features, following design patterns, writing comprehensive tests, and optimizing performance, you can build applications that are maintainable, scalable, and fast.

Key takeaways:

- Use PHP 8.x features like property hooks, asymmetric visibility, pipe operator, enums, and match expressions
- Implement design patterns for clean architecture
- Write comprehensive unit and feature tests
- Configure OPcache and JIT for production performance
- Use DTOs for type-safe data transfer