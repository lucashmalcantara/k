---
title: Workshop - Unit Testing with .NET - 2024.08.07
draft: false
tags:
  - software-development
  - back-end
  - dotnet
  - csharp
---
## Preface

This is the script for the workshop I presented at [[Wake Experience]], a division of [[LWSA]] (formerly Locaweb), on August 07, 2024. At the time, I was working as a Software Developer Specialist at this company.

## Personal presentation

- 10 months at Wake Experience.
- Backend software developer in Squad SMB.
- 6 years of experience.
- I have worked at other major companies, including BTG Pactual and Localiza.

## Introduction

This workshop aims to introduce basic concepts about software testing and develop a .NET API project applying some of these concepts with a focus on writing unit tests.

## Testing Pyramid

The [[Testing Pyramid]] is a metaphor for thinking about testing in software. [[Mike Cohn]] came up with this concept in his book [[Succeeding with Agile]]. It's a great visual representation that helps us think about different layers of testing. It also tells you how much testing to do on each layer [[#1]] [[#2]]. 

Regarding the Test Pyramid, it is also important to highlight that the higher the layer:
-   More effort.
-   Slower.
-   Higher maintenance.


![[Test Pyramid.png]]_The Test Pyramid (adapted)_

The Test Pyramid offers a rule of thumb for establishing the test suite [[#1]]:
1. Write tests with different granularity.
2. The more high-level you get the fewer tests you should have.

## Unit tests

[[Unit Testing|Unit tests]] are used to test isolated software components, such as individual class methods [[#5]]. Here are the main points about unit testing [[#1]] [[#3]]:
- The base of the test suite is composed of unit tests.
- It tests the **smallest part of the code**: function/method.
- **More numerous** than any other type of test.
- **Should be fast** (execution in milliseconds).
- Should avoid accessing databases, file systems, or HTTP requests (using mocks and stubs for these parts).
- Tests public interfaces, not private methods.
- When to use? It is used in all classes with production code, regardless of their functionality or which layer they belong to.

> It is the **most important class of tests** because it is **quick to develop**, requires **little maintenance**, provides **fast feedback**, and helps with **code documentation**.

## Integration tests

[[Integration Testing|Integration tests]] evaluate a software's components on a broader level than [[Unit Testing|unit tests]] [[#5]]. Here are the main points about integration testing [[#1]] [[#4]] [[#5]]:

- Integration tests confirm that two or more software components work together to produce and expected result.
-  Can be broad (broad integration test) or narrow (narrow integration test).
	- **Narrow integration test** (*teste de integração estreito*, in Portuguese): when writing narrow integration tests, you should aim to **run external dependencies locally**, using local instances or fake versions of services to mimic real interactions.
	- **Broad integration test** (*teste de integração abrangente*, in Portuguese): broad integration tests **require live versions of all services** and substantial test environment and network access, exercising code paths through all services, not just interactions.
- **Prefer to perform integration tests as narrowly as possible** by replacing services and databases with test doubles.
- Can be **more complex** to write than small, isolated unit tests.
- Provides the advantage of **greater confidence** that the software will function correctly when integrating various components.
- Should be used primarily when there is a need for data serialization or deserialization.

## End-to-End (E2E) tests 

[[End-to-end (E2E) testing]] evaluates an application's entire workflow, from start to finish, integrating all components and services [[#6]]. Here are the main points about integration testing [[#1]]:

- Among various testing methods, E2E testing offers the **highest level of assurance** that the software is functioning correctly.
- However, it is prone to **several issues**, often failing due to unexpected and unpredictable factors. These tests can frequently **yield false positives**, caused by network failures, browser-specific quirks, or runtime issues.
- Due to its **high maintenance** costs and slower execution, e2e testing should be minimized.
- When to use it? Employ E2E testing for **high-value user integrations**. For instance, in an e-commerce site, it is critical for testing the entire process of adding items to the cart and completing the checkout.

> Remember: you have lots of lower levels in your test pyramid where you already tested all sorts of edge cases and integrations with other parts of the system. There's no need to repeat these tests on a higher level. High maintenance effort and lots of false positives will slow you down and cause you to lose trust in your tests, sooner rather than later.
> 
> H. Vocke, 2018

## Test Doubles

[[Test Double Patterns]] is a term used to describe any object or component that helps us in software testing. They are used to replace production code for testing purposes or to verify if the software is behaving as expected. This term was created by [[Gerard Meszaros]] in this book [[XUnit Test Patterns - Refactoring Test Code]].

![[Pasted image 20240729201544.png]]
_Test Doubles_ [[#8]]

There are various kinds of double that Gerard lists [[#9]]:
- **Dummy** objects are passed around but never actually used. Usually they are just used to fill parameter lists.
- **Fake** objects actually have working implementations, but usually take some shortcut which makes them not suitable for production (an InMemoryTestDatabase is a good example).
- **Stubs** provide canned answers to calls made during the test, usually not responding at all to anything outside what's programmed in for the test.
- **Spies** are stubs that also record some information based on how they were called. One form of this might be an email service that records how many messages it was sent.
- **Mocks** are pre-programmed with expectations which form a specification of the calls they are expected to receive. They can throw an exception if they receive a call they don't expect and are checked during verification to ensure they got all the calls they were expecting.

At times, it can be difficult to define the limits of each type of Test Double. In his article on the subject [[#10]], Mark Seemann illustrates the extremes of test doubles through a spectrum, as shown in the image below.

![[Spectrum of Test Doubles.gif]]
_Spectrum of Test Doubles_

### Dummy object

A Dummy is used to fill in parameters or variables that are not actively used in the test.

```csharp
public class DummyTest
{
    [Fact]
    public void DummyExample()
    {
        // Arrange
        var dummyEmailService = new Mock<IEmailService>(); // Dummy, not used in the test
        var orderProcessor = new OrderProcessor(dummyEmailService.Object);

        // Act
        var result = orderProcessor.ProcessOrder(new Order());

        // Assert
        result.Should().BeTrue();
    }
}
```

### Fake object

A Fake has a working implementation but is not suitable for production.

```csharp
public class FakeOrderRepository : IOrderRepository
{
    private readonly List<Order> _orders = new List<Order>();

    public void Add(Order order)
    {
        _orders.Add(order);
    }

    public IEnumerable<Order> GetAll()
    {
        return _orders;
    }
}

public class FakeTest
{
    [Fact]
    public void FakeExample()
    {
        // Arrange
        var fakeOrderRepository = new FakeOrderRepository();
        var orderProcessor = new OrderProcessor(fakeOrderRepository);

        // Act
        orderProcessor.ProcessOrder(new Order());
        var orders = fakeOrderRepository.GetAll();

        // Assert
        orders.Should().ContainSingle();
    }
}
```

### Test Stub

A Stub provides predefined responses to specific calls made during the test.

```csharp
public class StubTest
{
    [Fact]
    public void StubExample()
    {
        // Arrange
        var order = new Order();
        var stubOrderRepository = new Mock<IOrderRepository>();
        stubOrderRepository.Setup(repo => repo.GetOrder(It.IsAny<int>())).Returns(order);
        var orderProcessor = new OrderProcessor(stubOrderRepository.Object);

        // Act
        var result = orderProcessor.GetOrder(1);

        // Assert
        result.Should().Be(order);
    }
}
```

### Test Spy

A Spy records information about the calls it receives, allowing verification of interactions.

```csharp
public class SpyTest
{
    [Fact]
    public void SpyExample()
    {
        // Arrange
        var spyEmailService = new Mock<IEmailService>();
        var orderProcessor = new OrderProcessor(spyEmailService.Object);

        // Act
        orderProcessor.ProcessOrder(new Order());

        // Assert
        spyEmailService.Verify(service => service.Send(It.IsAny<string>(), It.IsAny<string>()), Times.Once);
    }
}
```

### Mock object

A Mock is pre-programmed with expectations about how it should be used and verifies that those interactions occurred.

```csharp
public class MockTest
{
    [Fact]
    public void MockExample()
    {
        // Arrange
        var mockEmailService = new Mock<IEmailService>();
        var orderProcessor = new OrderProcessor(mockEmailService.Object);

        // Act
        orderProcessor.ProcessOrder(new Order());

        // Assert
        mockEmailService.Verify(service => service.Send(It.IsAny<string>(), It.IsAny<string>()), Times.Once);
    }
}
```

## Package list for software testing in .NET

There is a list of packages that can help us write tests in .NET. Below is a selection of some of the key packages available on NuGet.
- [Moq](https://github.com/devlooped/moq/wiki/Quickstart): It is a framework for creating test doubles (mocks, stubs, fakes, etc.)
	- NuGet: https://www.nuget.org/packages/Moq/
- [AutoFixture.AutoMoq](https://github.com/AutoFixture/AutoFixture): automatically generates object instances, filling them with appropriate values based on type, without the need for manual configuration for each test. It is useful, for example, for automatically injecting dependencies into objects (such as services, repositories, etc.) or for populating property values of instances (such as models, DTOs, etc.). More information is available in the [AutoFixture Documentation](https://autofixture.github.io/docs/quick-start/).
	- NuGet: https://www.nuget.org/packages/AutoFixture.AutoMoq/
- [FluentAssertions](https://fluentassertions.com/): makes reading and writing asserts easier.
	- NuGet: [https://www.nuget.org/packages/FluentAssertions](https://www.nuget.org/packages/FluentAssertions)​
- [FluentAssertions.Web](https://github.com/adrianiftode/FluentAssertions.Web): makes reading and writing asserts for HTTP requests easier.
	- NuGet: [https://www.nuget.org/packages/FluentAssertions.Web](https://www.nuget.org/packages/FluentAssertions.Web)​
- [AutoBogus](https://github.com/nickdodd79/AutoBogus): simplifies the generation of test data. Using a builder, it is possible to configure a set of rules to automatically create class instances, making it easier to create data sets for tests.
	- NuGet: [https://www.nuget.org/packages/AutoBogus](https://www.nuget.org/packages/AutoBogus)
- [MockQueryable](https://github.com/romantitov/MockQueryable): allows creating fake implementations for Entity Framework Core.
	- NuGet: [https://www.nuget.org/packages/MockQueryable.FakeItEasy/](https://www.nuget.org/packages/MockQueryable.FakeItEasy/)
- - [RichardSzalay.MockHttp](https://github.com/richardszalay/mockhttp): provides a set of testing features that make HttpClient testing easier. It is possible to stub and assert HTTP requests using a class called `MockHttpMessageHandler`.
	- NuGet: [https://www.nuget.org/packages/RichardSzalay.MockHttp](https://www.nuget.org/packages/RichardSzalay.MockHttp)
## Example project

Available on GitHub: [lucashmalcantara/2024-08-workshop-unit-testing-dotnet](https://github.com/lucashmalcantara/2024-08-workshop-unit-testing-dotnet)

## References

### 1

H. Vocke. “The practical test pyramid.” martinfowler.com. Accessed: Jul. 28, 2024. [Online.] Available:
https://martinfowler.com/articles/practical-test-pyramid.html

### 2

T. Fernandez and D. Ackerson. “The Testing Pyramid: How To Structure Your Test Suite.” semaphoreci.com. Accessed: Jul. 28, 2024. [Online.] Available: https://semaphoreci.com/blog/testing-pyramid

### 3

A. Okazaki. "Teste UNITÁRIO, teste de INTEGRAÇÃO e teste FUNCIONAL: QUANDO e COMO USAR?" youtube.com. Accessed: Jul. 28, 2024. [Online.] Available: https://youtu.be/35nlBX5_GzE

### 4

M. Fowler. “Integration Test.” martinfowler.com. Accessed: Jul. 28, 2024. [Online.] Available: [Integration Test (martinfowler.com)](https://martinfowler.com/bliki/IntegrationTest.html)

### 5

Microsoft. “Integration tests in ASP.NET Core.” microsoft.com. Accessed: Jul. 28, 2024. [Online.] Available: https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-8.0

### 6

F. Bellato. “A Evolução dos testes E2E no Zé Delivery.” medium.com. Accessed: Jul. 28, 2024. [Online.] Available: https://medium.com/rez%C3%A9nha/a-evolu%C3%A7%C3%A3o-dos-testes-e2e-no-z%C3%A9-delivery-894ac3781bbf

### 7

T. Fernandez and D. Ackerson. “The Testing Pyramid: How to Structure Your Test Suite.” semaphoreci.com. Accessed: Jul. 28, 2024. [Online.] Available: https://semaphoreci.com/blog/testing-pyramid

### 8

G. Meszaros. “Test Double.” xunitpatterns.com. Accessed: Jul. 29, 2024. [Online.] Available: http://xunitpatterns.com/Test%20Double.html

### 9

M. Fowler. “Test Double.” martinfowler.com. Accessed: Jul. 29, 2024. [Online.] Available: https://martinfowler.com/bliki/TestDouble.html

### 10

M. Seemann. “Unit Testing: Exploring The Continuum Of Test Doubles.” microsoft.com. Accessed: Jul. 29, 2024. [Online.] Available: https://learn.microsoft.com/en-us/archive/msdn-magazine/2007/september/unit-testing-exploring-the-continuum-of-test-doubles

