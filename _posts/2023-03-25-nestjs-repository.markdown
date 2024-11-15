# Nest.js repository

What is purpose of a nest.js repository.

In Nest.js, a repository is a design pattern used to abstract away the data persistence logic from the rest of the application. It provides a clean separation of concerns, making it easier to maintain and scale the application. The repository pattern is particularly useful when using TypeORM with Nest.js.

A Nest.js repository typically:

Encapsulates the data access logic: A repository isolates the details of how data is stored and retrieved, providing a consistent API for the rest of the application to interact with the data.

Promotes clean architecture: By separating the data access logic from the business logic, you can make changes to the data source or underlying storage technology without affecting the rest of the application.

Enables easier testing: With a repository in place, you can mock the data access layer when writing tests for your application, allowing you to test your services and controllers without needing to interact with the actual database.

Provides a centralized location for queries: By placing queries inside the repository, you can avoid duplicating query logic throughout your application.

In summary, a Nest.js repository helps you maintain a clean architecture, simplifies testing, and provides a centralized location for data access logic and queries, making it easier to maintain and scale your application.

## How to create a Nest.js repository

To create a Nest.js repository, you need to create a new service and inject the TypeORM repository into it. You can then use the repository to perform CRUD operations on the database.

Let’s create a new Nest.js application and add a new service to it.

```bash
nest new nestjs-repository
cd nestjs-repository
nest generate service users
```

Now, let’s add a new user entity to our application.

```bash
nest generate class users/user
```

Open the newly created user.entity.ts file and add the following code to it.

```typescript
import { Entity, Column, PrimaryGeneratedColumn } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  @Column()
  password: string;
}
```

Next, let’s add a new module to our application.

```bash
nest generate module users
```

Open the newly created users.module.ts file and add the following code to it.

```typescript
import { Module } from "@nestjs/common";
import { TypeOrmModule } from "@nestjs/typeorm";
import { User } from "./user.entity";
import { UsersService } from "./users.service";

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

Now, let’s add a new controller to our application.

```bash
nest generate controller users
```

Open the newly created users.controller.ts file and add the following code to it.

```typescript
import { Controller, Get, Post, Body } from "@nestjs/common";
import { UsersService } from "./users.service";
import { User } from "./user.entity";

@Controller("users")
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  async findAll(): Promise<User[]> {
    return this.usersService.findAll();
  }

  @Post()
  async create(@Body() user: User): Promise<User> {
    return this.usersService.create(user);
  }
}
```

Finally, let’s add a new repository to our application.

```bash
nest generate class users/user
```

Open the newly created user.repository.ts file and add the following code to it.

```typescript
import { EntityRepository, Repository } from "typeorm";
import { User } from "./user.entity";

@EntityRepository(User)
export class UserRepository extends Repository<User> {}
```

Now, let’s update the users.service.ts file to use the newly created repository.

```typescript
import { Injectable } from "@nestjs/common";
import { InjectRepository } from "@nestjs/typeorm";
import { User } from "./user.entity";
import { UserRepository } from "./user.repository";

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(UserRepository)
    private userRepository: UserRepository
  ) {}

  async findAll(): Promise<User[]> {
    return this.userRepository.find();
  }

  async create(user: User): Promise<User> {
    return this.userRepository.save(user);
  }
}
```

Now, let’s update the app.module.ts file to import the newly created users module.

```typescript
import { Module } from "@nestjs/common";
import { TypeOrmModule } from "@nestjs/typeorm";
import { UsersModule } from "./users/users.module";

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: "sqlite",
      database: "database.sqlite",
      entities: [__dirname + "/**/*.entity{.ts,.js}"],
      synchronize: true,
    }),
    UsersModule,
  ],
})
export class AppModule {}
```

Now, let’s start the application and test it.

```bash
npm run start
```

Open http://localhost:3000/users in your browser and you should see an empty array.

Now, let’s create a new user.

```bash
curl -X POST -H "Content-Type: application/json" -d '{"name": "John Doe", "email": "

", "password": "password"}' http://localhost:3000/users
```

Open http://localhost:3000/users in your browser and you should see the newly created user.

## Conclusion

In this article, we learned how to create a Nest.js repository. We also learned how to use the repository to perform CRUD operations on the database.

## References

- [Nest.js](https://nestjs.com/)
- [TypeORM](https://typeorm.io/#/)
- [Nest.js repository pattern](https://docs.nestjs.com/recipes/sql-typeorm#repository-pattern)
- [Nest.js repository](
  [...]
