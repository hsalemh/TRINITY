Collecting workspace information

Le backend de notre projet `trinity-devops-app` est construit avec le framework [NestJS](https://nestjs.com/), qui est un framework Node.js progressif pour la construction d'applications côté serveur efficaces et évolutives. Voici une explication détaillée de la structure et du fonctionnement du backend, ainsi que des relations entre les différents fichiers.

### Structure du Backend

La structure du backend est organisée de manière modulaire, ce qui permet de séparer les différentes fonctionnalités en modules distincts. Voici un aperçu des principaux fichiers et dossiers :

- `src/`
  - 

app.module.ts


  - 

app.controller.ts


  - 

app.service.ts


  - `auth/`
    - `auth.module.ts`
    - `auth.service.ts`
    - `auth.controller.ts`
    - `jwt.strategy.ts`
  - `users/`
    - `users.module.ts`
    - 

users.service.ts


    - `users.controller.ts`
    - 

user.entity.ts


  - `products/`
    - `products.module.ts`
    - `products.service.ts`
    - `products.controller.ts`
    - `product.entity.ts`
    - 

category.entity.ts


  - `orders/`
    - `orders.module.ts`
    - `orders.service.ts`
    - `orders.controller.ts`
    - `order.entity.ts`
    - `order-item.entity.ts`
  - `cart/`
    - `cart.module.ts`
    - `cart.service.ts`
    - `cart.controller.ts`
    - `cart.entity.ts`
    - `cart-item.entity.ts`
  - 

main.ts



### Fonctionnement des Fichiers

#### 

main.ts



Le fichier 

main.ts

 est le point d'entrée de l'application. Il initialise l'application NestJS et configure certaines options globales comme CORS et Swagger pour la documentation de l'API.

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import * as dotenv from 'dotenv';

dotenv.config();

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Autorise CORS pour les requêtes API
  app.enableCors({
    origin: 'http://localhost:8080',
    methods: 'GET,HEAD,PUT,PATCH,POST,DELETE',
    allowedHeaders: 'Content-Type, Authorization',
    credentials: true,
  });

  await app.listen(process.env.PORT ?? 4000);

  // Configuration de Swagger pour la documentation de l'API
  const config = new DocumentBuilder()
    .setTitle('Trinity API')
    .setDescription('API pour la gestion des stocks')
    .setVersion('1.0')
    .build();
  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, document);
}
bootstrap();
```

#### 

app.module.ts



Le fichier 

app.module.ts

 est le module racine de l'application. Il importe et configure tous les autres modules nécessaires à l'application.

```ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigModule } from '@nestjs/config';
import { AuthModule } from './auth/auth.module';
import { UsersModule } from './users/users.module';
import { ProductsModule } from './products/products.module';
import { OrdersModule } from './orders/orders.module';
import { CartModule } from './cart/cart.module';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { User } from './users/user.entity';
import { Product } from './products/product.entity';
import { Category } from './products/category.entity';
import { Order } from './orders/order.entity';
import { OrderItem } from './orders/order-item.entity';
import { Cart } from './cart/cart.entity';
import { CartItem } from './cart/cart-item.entity';

@Module({
  imports: [
    ConfigModule.forRoot(),
    TypeOrmModule.forRoot({
      type: 'mssql',
      host: 'mytrinityserver.database.windows.net',
      port: 1433,
      username: 'trinityRegistry',
      password: 'Epi-@tech242424@@epitech',
      database: 'mytrinitysqldatabaseprod',
      entities: [
        User,
        Product,
        Category,
        Order,
        OrderItem,
        Cart,
        CartItem,
      ],
      synchronize: true,
      extra: {
        encrypt: true,
        trustServerCertificate: false,
      },
      retryAttempts: 3,
      retryDelay: 3000,
      autoLoadEntities: true,
      keepConnectionAlive: true,
      verboseRetryLog: true,
      manualInitialization: false,
    }),
    TypeOrmModule.forFeature([
      User,
      Product,
      Category,
      Order,
      OrderItem,
      Cart,
      CartItem,
    ]),
    AuthModule,
    UsersModule,
    ProductsModule,
    OrdersModule,
    CartModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

#### 

app.controller.ts

 et 

app.service.ts



Ces fichiers définissent le contrôleur et le service de l'application principale. Le contrôleur gère les requêtes HTTP entrantes et le service contient la logique métier.

```ts
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```

### Modules et Services

#### Module d'Authentification (`auth/`)

Le module d'authentification gère la logique d'authentification et d'autorisation. Il utilise JWT pour sécuriser les endpoints.

- `auth.module.ts`: Configure le module d'authentification.
- `auth.service.ts`: Contient la logique d'authentification.
- `auth.controller.ts`: Gère les requêtes HTTP liées à l'authentification.
- `jwt.strategy.ts`: Définit la stratégie JWT pour l'authentification.

#### Module Utilisateurs (`users/`)

Le module utilisateurs gère les opérations liées aux utilisateurs.

- `users.module.ts`: Configure le module utilisateurs.
- 

users.service.ts

: Contient la logique métier pour les utilisateurs.
- `users.controller.ts`: Gère les requêtes HTTP liées aux utilisateurs.
- 

user.entity.ts

: Définit l'entité utilisateur pour TypeORM.

#### Module Produits (`products/`)

Le module produits gère les opérations liées aux produits et aux catégories.

- `products.module.ts`: Configure le module produits.
- `products.service.ts`: Contient la logique métier pour les produits.
- `products.controller.ts`: Gère les requêtes HTTP liées aux produits.
- `product.entity.ts`: Définit l'entité produit pour TypeORM.
- 

category.entity.ts

: Définit l'entité catégorie pour TypeORM.

#### Module Commandes (`orders/`)

Le module commandes gère les opérations liées aux commandes et aux éléments de commande.

- `orders.module.ts`: Configure le module commandes.
- `orders.service.ts`: Contient la logique métier pour les commandes.
- `orders.controller.ts`: Gère les requêtes HTTP liées aux commandes.
- `order.entity.ts`: Définit l'entité commande pour TypeORM.
- `order-item.entity.ts`: Définit l'entité élément de commande pour TypeORM.

#### Module Panier (`cart/`)

Le module panier gère les opérations liées aux paniers et aux éléments de panier.

- `cart.module.ts`: Configure le module panier.
- `cart.service.ts`: Contient la logique métier pour les paniers.
- `cart.controller.ts`: Gère les requêtes HTTP liées aux paniers.
- `cart.entity.ts`: Définit l'entité panier pour TypeORM.
- `cart-item.entity.ts`: Définit l'entité élément de panier pour TypeORM.

### Fonctionnement des Services

Les services contiennent la logique métier de l'application. Par exemple, le service des produits (`products.service.ts`) contient des méthodes pour créer, rechercher, filtrer et supprimer des produits.

```ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Product } from './product.entity';
import { Category } from './category.entity';

@Injectable()
export class ProductsService {
  constructor(
    @InjectRepository(Product)
    private readonly productsRepository: Repository<Product>,
    @InjectRepository(Category)
    private readonly categoriesRepository: Repository<Category>,
  ) {}

  async create(createProductDto: CreateProductDto): Promise<Product> {
    const product = this.productsRepository.create(createProductDto);
    return this.productsRepository.save(product);
  }

  async searchProducts(query: string): Promise<Product[]> {
    return this.productsRepository.find({
      where: { name: query },
    });
  }

  async filterProducts(categoryId: number, minPrice: number, maxPrice: number, minRating: number): Promise<Product[]> {
    const queryBuilder = this.productsRepository.createQueryBuilder('product');
    if (categoryId) {
      queryBuilder.andWhere('product.categoryId = :categoryId', { categoryId });
    }
    if (minPrice) {
      queryBuilder.andWhere('product.price >= :minPrice', { minPrice });
    }
    if (maxPrice) {
      queryBuilder.andWhere('product.price <= :maxPrice', { maxPrice });
    }
    if (minRating) {
      queryBuilder.andWhere('product.rating >= :minRating', { minRating });
    }
    return queryBuilder.getMany();
  }

  async remove(id: number): Promise<void> {
    const product = await this.productsRepository.findOne(id);
    await this.productsRepository.remove(product);
  }
}
```

### Conclusion

Le backend de `trinity-devops-app` est bien structuré et utilise les meilleures pratiques de développement avec NestJS. Chaque module est responsable d'une fonctionnalité spécifique, ce qui rend le code maintenable et évolutif. Les services contiennent la logique métier, tandis que les contrôleurs gèrent les requêtes HTTP. Les entités définissent la structure des données et sont utilisées par TypeORM pour interagir avec la base de données.

Similar code found with 1 license type