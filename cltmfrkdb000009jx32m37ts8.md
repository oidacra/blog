---
title: "How the Unit Tests transform your Angular's applications."
datePublished: Mon Mar 11 2024 04:21:09 GMT+0000 (Coordinated Universal Time)
cuid: cltmfrkdb000009jx32m37ts8
slug: how-the-unit-tests-transform-your-angulars-applications
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/4JxV3Gs42Ks/upload/4f507f172d7784577ca6d2969a331258.jpeg
tags: unit-testing, angular, best-practices

---

**Unit testing** is an essential part of software development, along with End-to-End (E2E) and Integration Testing. It plays a key role in ensuring our application's quality. For software developers, this can be one of the most challenging aspects of Angular. It involves deeply simulating how Angular operates, grasping the workings of Dependency Injection, the Lifecycles of Angular, and how to simulate (mock) services. Initially, understanding and crafting a good unit test can be quite difficult.

### Testing Pyramid

![Testing Pyramid](https://cdn.hashnode.com/res/hashnode/image/upload/v1710130321400/6b629374-5950-4a7e-a11a-912ba685b08e.png align="center")

The objective of the testing pyramid is to optimize the quantity of rapid and cost-effective tests (unit tests) while reducing the frequency of slower and more costly tests (E2E tests). This approach not only enhances the efficiency of testing but also facilitates prompt identification of issues. In essence, the testing pyramid serves as a strategic framework for effectively and efficiently allocating testing resources, thereby ensuring the software's quality in a scalable manner.

### But, Why is Unit Testing important?

* **Find errors early:** Unit tests let you spot mistakes in your code as you create it, so it's easier to fix them before users are affected.
    
* **Make better code:** Writing unit tests makes you think about how your code works and helps ensure it's well-made.
    
* **Refactor safely:** Unit tests give you confidence that you can change your code without causing problems.
    
* **Boost code coverage:** Code coverage is the percentage of code that unit tests run. High code coverage shows that your code is well-tested (And that's where the importance of doing a good Unit Test lies).
    

### **What tools do you have to do unit tests in Angular?**

Angular has built-in tools for unit tests. The main ones are:

* **Karma:** A test runner that runs unit tests in the browser.
    
* **Jasmine:** A framework for writing unit tests.
    
* **TestBed:** A module that helps you set up a testing environment for your component.
    
* [**Jest**](https://jestjs.io/)**:** A framework for writing tests in the browser and can be used in replace of Karma.
    

### **What makes a test a good Unit Test?**

Some recommended practices for conducting a Unit Test include:

* Arrange, Act, Assert (AAA)
    
    * **Arrange**: This is the setup so that our test has everything it needs to function.
        
    * **Act**: Execute the action
        
    * **Assert**: Verify the result of the Act.
        
* Isolate the component to be tested
    
* For components that depend on services for their behavior, use “Test Doubles” (mocks, stubs, doubles).
    
* Use spies to verify the behavior of methods or functions.
    
* Always use Mock data. This makes the tests predictable.
    
* Use a good name for the Unit Test.
    
* Test the expected interactions: Click on our buttons, don't call methods directly
    

```typescript
it('should emit form value when the user clicks the button and form is valid', async () => {
      const expectedEmittedQueryValue = 'test';
      component.form.setValue({ query: expectedEmittedQueryValue });
      component.form.markAsTouched();

      // Spies
      const submitSpy = jest.spyOn(component, 'submitForm');
      const outputSpy = jest.spyOn(component.query, 'emit');

      // Find the button
      const button = de.nativeElement.querySelector(
        '[data-testId="submit-button"]'
      );

      // Always check the null
      expect(button).not.toBeNull();

      // Simulate the click of the user at the button
      button.click();

      expect(component.form.valid).toBe(true); 
      expect(submitSpy).toHaveBeenCalled();
      expect(outputSpy).toHaveBeenCalledWith(expectedEmittedQueryValue);
    });
```

* Find elements with **data-testId**, not with CSS classes.
    

```xml
 <span
    data-testId="reset-icon"
    nz-icon
    class="ant-input-clear-icon"
    nzTheme="fill"
    nzType="close-circle"
    (click)="resetForm()">
</span>
```

```typescript
it('form should be reset when click clear icon', async () => {
        // Add something to the form
        component.form.setValue({ query: 'test' });
        fixture.detectChanges();

        await fixture.whenStable();
        
        // Use the data-testId not the class name of the element
        const clearIcon = de.nativeElement.querySelector(
          '[data-testId="reset-icon"]'
        );
        // Check the null
        expect(clearIcon).not.toBeNull();

        // Act
        jest.spyOn(component, 'resetForm'); // Spi the method
        clearIcon.click(); // Simulate the click the element

        // Assert
        expect(component.resetForm).toHaveBeenCalled();
        expect(component.form.controls['query'].value).toBeNull();
      });
```

* Test all possible code paths (both true and false)
    

```typescript
it('should show loading button when input is loading `true`', () => {
      // arrange
      const button = fixture.debugElement.query(
        By.directive(NzButtonComponent)
      );
      expect(button).not.toBeNull();

      // checking the false
      fixture.componentRef.setInput('isLoading', false);
      fixture.detectChanges();

      // assert
      expect(button.injector.get(NzButtonComponent).nzLoading).toBe(false);

      // checking the true
      fixture.componentRef.setInput('isLoading', true);
      fixture.detectChanges();

      // assert
      expect(button.injector.get(NzButtonComponent).nzLoading).toBe(true);
    });
```

* Mock a **service** used in a component to verify the message displayed in the component
    

```typescript
describe('UserComponent', () => {
  let component: UserComponent;
  let fixture: ComponentFixture<UserComponent>;
  let userServiceMock: Partial<UserService>;

  beforeEach(async () => {
    // create a mock of the service
    userServiceMock = {
      isLoggedIn: jest.fn(),
    };

    await TestBed.configureTestingModule({
      declarations: [UserComponent],

      // Replace the UserService with the mock
      providers: [{ provide: UserService, useValue: userServiceMock }],
    }).compileComponents();

    fixture = TestBed.createComponent(UserComponent);
    component = fixture.componentInstance;
  });

  it('should show "Logged In" if the user is logged in', () => {
    // manipulate the value of isLoggedIn to return true
    userServiceMock.isLoggedIn.mockReturnValue(true);
    fixture.detectChanges(); 
    expect(component.message).toBe('Logged In');
  });

  it('should show "Not Logged In" if the user is not logged in', () => {
    // manipulate the value of isLoggedIn to return false
    userServiceMock.isLoggedIn.mockReturnValue(false);
    fixture.detectChanges(); 
    expect(component.message).toBe('Not Logged In');
  });
});
```

### What other tools do I use for Unit Test

1. [**Spectator**](https://github.com/ngneat/spectator): Simplifies Angular testing by removing repetitive tasks, making unit tests clear and efficient. It enables testing components, directives, services, and more without needing deep knowledge of TestBed, ComponentFixture, and DebugElement APIs. Features include simple DOM querying, event triggering, routing and HTTP testing support, auto-mocking providers, and Jest compatibility.
    
2. [NgMock](https://ng-mocks.sudo.eu/): This is a library for Angular that simplifies the creation of mocks in tests, allowing the simulation of components, directives, pipes, services, modules, and tokens. It makes configuring TestBed easier, reduces repetitive code in tests, and offers a simple interface to access declarations. Compatible with various versions of Angular, NgMocks supports both Jasmine and Jest, helping to create a more efficient and less tedious testing environment.
    

### Conclusion

In conclusion, unit testing is crucial for creating robust and reliable Angular applications. It allows developers to catch errors early, enhance code quality, ensure safe refactoring, and achieve high code coverage. With Angular's tools like Karma, Jasmine, TestBed, and optionally Jest, developers can follow best practices such as Arrange, Act, Assert (AAA), component isolation, using test doubles, and testing all code paths. These strategies lead to functional, maintainable, and scalable applications, improving software quality and the development process.