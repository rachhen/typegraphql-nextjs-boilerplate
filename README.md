# Type GraphQL Series

A quick introduction to type-graphql library : [https://typegraphql.ml/]

## Bootstrap with apollo-server-graphql

### installing

> yarn add reflect-metadata graphql type-graphql

### bootstrapping

```typescript
import { buildSchema, Resolver, Query } from "type-graphql";

@Resolver()
class HelloResolver {
    @Query(() => String, { name: "hello", nullable: true })
    async hello() {
        return "hello world";
    }
}

(async () => {
    const app = express();

    const schema = await buildSchema({ resolvers: [HelloResolver] });

    const apolloServer = new ApolloServer({ schema });

    apolloServer.applyMiddleware({ app });

    app.listen(4000, () => {
        console.log("Listening on port 4000...");
    });
})();
```

## write mutation

```typescript
@ObjectType()
@Entity()
export class User extends BaseEntity {
    @Field(() => ID)
    @PrimaryGeneratedColumn()
    id: number;

    @Field()
    @Column("text")
    firstName: string;

    @Field()
    @Column("text")
    lastName: string;

    @Field()
    fullName: string;

    @Field()
    @Column("text", { unique: true })
    email: string;

    @Field()
    @Column("text")
    password: string;

    @BeforeInsert()
    async hashPassword() {
        const hashedPassword = await hash(this.password, 12);
        this.password = hashedPassword;
    }
}
```

```typescript
@Resolver(User) // specified as @root
export class UserResolver {
    @Query(() => String)
    hello(): string {
        return "Hello from user";
    }

    @Mutation(() => User) // it is possible cuz we defined User Entity as an Object type.
    async register(
        @Arg("firstName") firstName: string,
        @Arg("lastName") lastName: string,
        @Arg("email") email: string,
        @Arg("password") password: string
    ): Promise<User> {
        const user = await User.create({
            firstName,
            lastName,
            email,
            password
        }).save();

        return user;
    }

    @resolveField(() => String) // it 's useful to query on fields that not specified as Entity Column
    fullName(@Root parent: User) {
        return `${parent.firstName} ${parent.lastName}`;
    }
}
```

_there is also a shorter way to **resolve fields** directly on entity_

./entity/User.ts

```typescript
@Entity()
export class User extends BaseEntity {
    // other class @Column()

    @Field()
    fullName(@Root() parent: User): string {
        return `${parent.firstName} ${parent.lastName}`;
    }
}
```

## Validation

### defining an @inputType and use it as @Arg

> yarn add class-validator

./modules/user/register/RegisterInput.ts

```typescript
import { Length, isEmail } from "class-validation";

@InputType()
export class RegisterInput {
    @Field()
    @Length(3, 255)
    firstName: string;

    @Field()
    @Length(3, 255)
    lastName: string;

    @Field()
    @isEmail()
    email: string;

    @Field()
    password: string;
}
```

./modules/user/register/userResolver

```typescript
@Resolver()
export class UserResolver {
    @Mutation(() => User)
    register(@Arg("input")
    {
        firstName,
        lastName,
        email,
        password
    }: RegisterInput): Promise<User> {
        // doing register stuff
    }
}
```

here is how args looks like:

```graphql
mutation Register {
    register(registerInput: {firstName, lastName, email, password}){
        id
        .
        .
        .
    }
}
```

## Sundry

### ts-node-dev

Tweaked version of node-dev that uses ts-node under the hood.

> yarn add -D ts-node-dev

```json
{
    "start": "tsnd --respawn  src/index.ts"
}
```
