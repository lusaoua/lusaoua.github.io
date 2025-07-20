# [现代PHP项目中的依赖注入与容器实现](https://lusaoua.github.io/)
依赖注入(Dependency Injection, DI)是实现松耦合和可测试代码的重要模式。在现代PHP开发中，理解并正确应用DI是每个开发者的必备技能。本文将探讨DI的核心概念及其在PHP中的实现方式。

1. 依赖注入基础
什么是依赖注入？

将对象的依赖通过构造函数、方法或属性从外部传入，而不是在内部创建。

传统方式 vs DI方式：

php
// 传统方式 - 紧耦合
class OrderService {
    private $repository;
    
    public function __construct() {
        $this->repository = new OrderRepository();
    }
}

// DI方式 - 松耦合
class OrderService {
    private $repository;
    
    public function __construct(OrderRepositoryInterface $repository) {
        $this->repository = $repository;
    }
}
2. DI容器的实现
简单容器示例：

php
class Container {
    private $services = [];
    
    public function register(string $name, callable $resolver): void {
        $this->services[$name] = $resolver;
    }
    
    public function resolve(string $name): object {
        if (!isset($this->services[$name])) {
            throw new RuntimeException("Service $name not found");
        }
        
        return $this->services[$name]($this);
    }
}

// 使用示例
$container = new Container();
$container->register('db', fn() => new PDO('mysql:host=localhost;dbname=test', 'user', 'pass'));
$container->register('orderRepo', fn($c) => new OrderRepository($c->resolve('db')));
3. 使用PSR-11标准容器
现代PHP项目通常遵循PSR-11(Container Interface)标准：

php
use Psr\Container\ContainerInterface;

class OrderController {
    private $orderService;
    
    public function __construct(ContainerInterface $container) {
        $this->orderService = $container->get(OrderService::class);
    }
}
推荐容器实现：

PHP-DI

Symfony DependencyInjection

Laravel容器

4. 自动装配(Autowiring)
现代DI容器通常支持自动解析依赖：

php
// PHP-DI示例
$container = new DI\Container();
$orderService = $container->get(OrderService::class); // 自动解析所有依赖
优势：

减少样板代码

更易于重构

提高开发效率

5. 最佳实践
面向接口编程：依赖应该基于接口而非具体实现

构造函数注入：首选方式，保证对象完整性和不变性

避免服务定位器模式：明确依赖而非从容器中获取

合理使用单例：无状态服务适合单例，有状态服务需谨慎

6. 在流行框架中的应用
Laravel：

php
class UserController extends Controller {
    public function __construct(protected UserRepository $users) {}
    
    public function show($id) {
        return view('user.profile', [
            'user' => $this->users->find($id)
        ]);
    }
}
Symfony：

yaml
# config/services.yaml
services:
    App\Repository\UserRepository:
        arguments:
            $entityManager: '@doctrine.orm.entity_manager'
结语
依赖注入是现代PHP架构的核心模式之一。通过合理应用DI和容器，可以创建出更灵活、可测试和可维护的应用程序。建议从简单的DI开始，逐步过渡到使用成熟的DI容器解决方案。
