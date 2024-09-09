+++
authors = ["Yadullah Duman"]
title = "How to turn off logging in NestJS tests"
date = "2024-09-09"
description = "Small guide on how to deal with the NestJS logger on unit tests"
categories = ["NestJS", "Node.js"]
tags = [
    "NestJS",
    "Node.js",
    "Backend",
    "JS"
]
+++

This post will be very short. I want to share on how to deal with the Logger class when running NestJS unit tests. In one of our projects, we had a small issue, that when running tests, the logger would start logging when certain branches of the code. Obviously, these branches should log when running on production, but for tests, we didn't care and it was noise. The solution was fairly simple.

## Example Scenario

Imagine this very simplified scenario. You want to test all branches of `getAllUsers`. The error case test would always execute the error log.

```ts
export class UserService {
    private readonly logger = new Logger(UserService.name);

    constructor(private userRepo: UserRepo) {}

    async getAllUsers() {
        try {
            return await this.userRepo.fetchAll();
        } catch (error) {
            this.logger.error("oops")
            throw error;
        }
    }
}
```

## Injecting the Logger

Instead of initializing the logger directly, we inject it to the service as an optional dependency. The reason behind it will be clear in the next example.

```ts
export class MyService {
    private readonly logger: Logger;

    constructor(
        private userRepo: UserRepo, 
        @Optional() logger?: Logger
    ) {
        this.logger = logger ? logger : new Logger(MyService.name);
    }

    async getAllUsers() {
        try {
            return await this.userRepo.fetchAll();
        } catch (error) {
            this.logger.error("oops")
            throw error;
        }
    }
}
```

## Disable desired log levels in tests

Here, we inject "our own" logger and define how it should behave for error logs. In this case, it does nothing, as we wanted. So for tests, the dependency injection will select our modified logger and on production, we will use the default logger.

```ts
describe("UserService Tests", () => {
  let service: UserService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UserService,
        {
          provide: Logger,
          useValue: {
            error: () => {}, // do not log errors
            // add more log levels if needed
          },
        },
      ],
    }).compile();

    service = module.get<UserService>(UserService);
  });
}
```