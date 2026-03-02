---
title: Building Confidence in APIs with Contract Testing and Pact
draft: true
description: TO UPDATE LATER
slug: building-confidence-in-apis-with-contract-testing-and-pact
date: 2025-03-06 14:00:00+0000
categories:
  - general
tags:
  - Showcase
  - General
weight: 1
---

# Introduction

This is the start of the post!

> This can probably be done a little later once I've added more detail to this post.

## What is contract testing?

In a world of microservices, it can be increasingly challenging to make changes to an API with confidence, without
breaking future consumers. Contract testing enables project teams to define a contract between a consumer and a
provider,
which allows them to verify that the consumer is compatible with the provider.

There are different methods of contract testing, but within this post I will be talking about the use of a Pact Broker
to perform contract testing.

- Can this not be covered by integration testing?

### Common terminology

| Term         | Description                                                                                                                                                                 |
|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Pact         | A contract that describes how a consumer expects to communicate with a provider.                                                                                            |
| Consumer     | A service that calls another service, turning it into a dependent service, for example a front end website.                                                                 |
| Provider     | A service that provides responses to another dependent service, for example a RESTful API.                                                                                  |
| Pacticipant  | Any service which is involved in the contract testing process. Both a consumer and provider are pacticipants.                                                               |
| Verification | A published test result created by a provider. This explains which versions of both provider and consumer were tested against each other, and if they're compatible or not. |

## Contract testing in practice

Contract testing can be done in many different ways, and even the Pact Foundation documents at least 10+ different
languages/frameworks for performing contract testing. Throughout this post, I will be focusing on how it's used via
Java.

### What is the Pact Broker?

To perform pact testing effectively, a Pact Broker is used to store and manage Pacts between consumers and providers. It
provides a centralized location for storing and retrieving Pacts, making it easier to manage and maintain contract
testing across multiple services. Pact Broker is an open source application freely available for everyone to use for
free. [Pactflow](https://www.pactflow.io/) is also available for use and is a fork of Pact Broker.

### Generating Pacts and publishing them

> Talk about flexibility

To begin any form of contract testing, a Pact must be generated. So what is a Pact?

To state the obvious, a Pact is a contract between a consumer and a provider. This contract defines the expected
responses it expects from a RESTful API, and how it should be formatted. To talk very high level, it's actually just
a JSON file that defines the expected behavior of a RESTful API. However, as developers, we don't need to manually
create
these files. Instead, we can use Pact's Java DSL to generate them automatically.

There are some rules that should be followed when generating Pacts:

- The Pact should be defined using the Provider's API specification. Any deviation from the actual behaviour can cause
  issues.
- Hard coded values should be avoided as this makes it difficult to maintain and update the Pact over time.
- Only outline values that are relevant to the consumer. Multiple consumers may use the same provider, so it's important
  to make sure that your Pact remains only relevant to the consumer.

Using the above rules, and with the Pact Consumer library, we can generate a Pact that can be published to a Pact
Broker.

To set up the gradle project for generating Pacts using the Java library, the plugin and dependency will need adding to
the gradle build file:

```gradle
plugins {
  id 'au.com.dius.pact.consumer' version '4.1.1'
}

dependencies {
  testImplementation 'au.com.dius.pact.consumer:junit5:4.6.19'
}
```

Then in the below example, I have created a Pact for GOV UK's `https://www.gov.uk/bank-holidays.json` API:

> **Complete this example!**

```java
@SpringBootTest(
    webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT,
    properties = {"app.claims-api.url=http://localhost:1232"})
@PactConsumerTest
@PactTestFor(providerName = "govuk-bank-holidays")
@MockServerConfig(port = "1232")
public final class GetBankHolidaysPactTest {

  @Autowired BankHolidayRestClient bankHolidaysRestClient;

  @SneakyThrows
  @Pact(consumer = CONSUMER)
  public RequestResponsePact getBankHolidays(PactDslWithProvider builder) {
    return builder
        .given("there are upcoming bank holidays")
        .uponReceiving("a request to fetch all upcoming bank holidays")
        .matchPath("/bank-holidays.json")
        .method("GET")
        .willRespondWith()
        .status(200)
        .headers(Map.of("Content-Type", "application/json"))
        .body(
            LambdaDsl.newJsonBody(
                    body -> {
                      body.uuid("id", CLAIM_ID);
                      body.object(
                          "england-and-wales",
                          region -> {
                            fee.stringType("division", "england-and-wales");
                                                
                          })
                .build())
        .toPact();
  }
```

In the above example, I have defined a Pact for a Provider called "govuk-bank-holidays". This tells Pact Broker which
service will be the provider as Pact Broker can host multiple microservice ecosystems. This is the value which would be
used in future consumers, and also the provider itself.

I have also defined `given` with the value `there are upcoming bank holidays`. This will be used as a
state for how the provider should be setup for the test. If you have similar use cases, make sure to reuse the same
`given` value to aid in the Pact verification process.

Then using the Pact Consumer gradle plugin we can publish this Pact to a Pact Broker using `./gradlew pactPublish`. The
below example shows how to
configure the plugin:

```gradle

pact {
    publish {
        pactBrokerUrl = 'https://localhost:8080'
        pactBrokerUsername = System.getenv('PACT_BROKER_USERNAME')
        pactBrokerPassword = System.getenv('PACT_BROKER_PASSWORD')

        pactDirectory = layout.buildDirectory.dir("pacts").get().asFile.path

        def projectVersion = project.version != 'unspecified' ? project.version : '1.0.0';
        def gitSha = providers.environmentVariable('GIT_SHA').getOrNull()
        
        // Define the branch from a property, environment variable, or fallback to 'main'
        def branchName = providers.environmentVariable('GIT_BRANCH').getOrElse('main')

        // If we have a SHA, append it to the version for uniqueness
        consumerVersion = gitSha ? gitSha : projectVersion

        // For newer Pact Brokers (Recommended)
        consumerBranch = branchName

        // For older Brokers/compatibility
        tags = [branchName, projectVersion]

        // Add 'release' tag if branchName matches semantic version pattern (X.X.XXX)
        if (branchName ==~ /\d+\.\d+\.\d+/) {
            tags.add('release')
        }
        // Optional: add a generic 'dev' tag for anything not on main
        else if (branchName != 'main' && branchName != 'master') {
            tags.add('dev')
        }
    }
}
```

It's important to highlight that publishing the Pact for a consumer is not enough to verify that the consumer is
compatible with the provider. This only defines the contract between the consumer and provider. The provider will use
this Pact to verify that the consumer is compatible with the provider.

### Verifying Pacts against an API

Now that we have a Pact published on the Pact Broker, we can use the Pact Provider Gradle plugin to verify that the
Pact is compatible with the provider. In the provider project, we can add the following to the `build.gradle` file:

```gradle
plugins {
    id 'au.com.dius.pact' version '4.6.19'
}

dependencies {
    // Source: https://mvnrepository.com/artifact/au.com.dius.pact.provider/spring7
    testImplementation 'au.com.dius.pact.provider:spring7:4.7.0-beta.3'
}

tasks.register('pactTest', Test) {
    description = 'Runs the pact tests.'
    group = 'verification'
    useJUnitPlatform()

    testClassesDirs = sourceSets.pactTest.output.classesDirs
    classpath = sourceSets.pactTest.runtimeClasspath

    // Publish if both username and password environment variables are provided
    def canPublish =
            providers.environmentVariable('PACT_BROKER_USERNAME').isPresent() &&
            providers.environmentVariable('PACT_BROKER_PASSWORD').isPresent()

    if (canPublish) {
        systemProperty 'pact.verifier.publishResults', 'true'
    }

    // Pass environment variables into the test process as System Properties
    systemProperty 'pact.provider.version', providers.environmentVariable('CI_COMMIT_SHA').getOrElse('1.0.0')
    systemProperty 'pactbroker.url', 'https://localhost:8080'
    systemProperty 'pactbroker.auth.username', providers.environmentVariable('PACT_BROKER_USERNAME')
    systemProperty 'pactbroker.auth.password', providers.environmentVariable('PACT_BROKER_PASSWORD')

    // Only set the consumer filter and branch selectors if PACT_CONSUMER_NAME is provided
    def consumerName = providers.environmentVariable('PACT_CONSUMER_NAME').orNull

    if (consumerName) {
        systemProperty 'pact.filter.consumers', consumerName
    }
    def branch = providers.environmentVariable('PACT_CONSUMER_BRANCH').getOrElse('main')
    systemProperty 'pactbroker.consumerversionselectors.rawjson', "[{\"branch\":\"${branch}\"}]"

    // Resolve the directory path to a String for the system property
    systemProperty 'pact.rootDir', layout.buildDirectory.dir("pacts").get().asFile.path
}
```

Then for the test itself, we just need a test which will utilise the Pact Provider library to verify that the Pact
is compatible with the provider. By telling the Pact Provider library to use my Pact Broker, it should automatically
pull any Pact published on the Pact Broker that has been linked to this provider using the value "govuk-bank-holidays"
as the provider name. Also in this example, the API is created using Spring Boot, so this will be created as a
`SpringBootTest`:

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@Provider(value = "laa-data-claims-api")
@PactBroker
public class DataClaimsApiProviderTests {

  @LocalServerPort private int port;
  
  /** Use MockitoBean to mock any services that are used by the API. **/

  @BeforeEach
  void setUp(PactVerificationContext context) {
    HttpTestTarget target = new HttpTestTarget("localhost", port);
    context.setTarget(target);
  }
  
  @TargetRequestFilter
  public void requestFilter(HttpRequest request) {
    request.addHeader("Authorization", "00000000-0000-0000-0000-000000000000");
  }

  @TestTemplate
  @ExtendWith(PactVerificationInvocationContextProvider.class)
  void pactVerificationTestTemplate(PactVerificationContext context) {
    context.verifyInteraction();
  }
}
```

The above example seems pretty small, but that is all that is required to pull and test itself against every published
Pact. However, it will be missing some important setups. As previously mentioned when creating the Pact, we need to
define states for the provider to satisfy the various "given" values. Each state will be used to tell the test how to
setup the provider for the specific test. These states can be defined in the same test class:

```java
... 

@MockitoBean
BankHolidayService bankHolidayService;

@State("there are upcoming bank holidays")
public void validationMessagesExistForTheSearchCriteria() {
  // Mock the BankHolidayService to return a list of BankHolidays for the given state
  when(bankHolidayService.getBankHolidays(any(BankHolidaySearchCriteria.class)))
      .thenReturn(
          new BankHolidayResponse(
              Arrays.asList(
                  getBankHoliday(BankHolidayType.BANK_HOLIDAY),
                  getBankHoliday(BankHolidayType.BANK_HOLIDAY))));
}

...
```

By defining the states, this should allow the test to run successfully. Also note in the `gradle.build` file, I 
specified that `pact.verifier.publishResults` is set to `true`. This means that after each test run, the results
will be published to the Pact Broker whether it be successful or not. This can be used to help with understanding
if the consumer is compatible with the current state of the provider or not.

## Applying contract testing in a CI pipeline

## What does this enable us to do?

> Keep this section about how contract testing can enable us, and introduce confidence in our APIs.

- From a Consumer perspective
- From a Provider perspective

## Conclusion

> The three golden rules of contract testing.

- Keep `main` the source of truth.
- Make sure API changes are backwards compatible, and only additive changes.
- Keep contracts flexible, and only test behaviour that is relevant to the consumer.
