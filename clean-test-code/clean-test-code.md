# Clean Test Code

## Method Name Expressing The Intention
When reading a test method -- written by somebody else or yourself -- it is important to know the intention of the test. The test method name should be clear enough so that the reader can easily understand the method of interest, the condition, and the expectation of the test outcomes. Having this information makes it easy for developers to maintain the test code. 

One common pattern used to name a test method is: `<method-being-tested>_<condition>_<expected-outcome>`.
For example, you are testing a method called `Divide` accepting 2 `double` arguments and returning the sum of the two numbers. You can name the test methods as such:

```C#
[TestMethod]
public void Divide_WithZeroAndOne_ReturnsZero() { ... }

[TestMethod]
public void Divide_WithTenAndTwo_ReturnsFive() { ... }

[TestMethod]
public void Divide_WithZeroAndZero_Throws() { ... }
```

Some people like the underscores as separators, and some prefer not to use underscores. This is just a matter of personal/team preference. Personally, I like the underscore since it gives you a direct separation between the _what_, the _when_ and the _outcome_.

## Single Responsibility
A test should have a single responsibility, that is to test one thing only. You can translate this as _one assert per test_. Just as a piece of production code should have a single responsibility, the test code should too for its readibility. A test method that tests multiple things will have a complicated method name, or an inconsistency between the method name and what it actually is testing. Simple names and simple tests are easier to comprihen. 

```C#
// Don't - test 2 things at the same time
[TestMethod]
public void TryDivide_WithOneAndZero_ReturnsFalseAndResultIsZero()
{
    // arrange
    // act
    var actual = Sut.TryDivide(1, 0, out var result);

    // assert
    Assert.IsFalse(actual);
    Assert.AreEqual(0, result);
}

// Do - split it to 2 methods
[TestMethod]
public void TryDivide_WithOneAndZero_ReturnsFalse()
{
    // arrange
    // act
    var actual = Sut.TryDivide(1, 0, out var _result);

    // assert
    Assert.IsFalse(actual);
}

[TestMethod]
public void TryDivide_WithOneAndZero_ShouldOutResultZero()
{
    // arrange
    // act
    Sut.TryDivide(1, 0, out var result);

    // assert
    Assert.AreEqual(0, result);
}
```

Some multiple assertions are still acceptable in some situations. The first exceptional occasion is when you want to assert the outcome but the outcome is a complex type (e.g. an object) that it cannot be asserted at once. 

```C#
// It's OK - asserting one complex type
[TestMethod]
public void CreateCircle_WithAnyCoordinateAndRadius_ReturnsWithGivenCoordinateAndRadius()
{
   // arrange
   // act
   var actual = Sut.CreateCircle(x: 5, y: 10, radius: 12);

   // assert 
   Assert.AreEqual(5, actual.X);
   Assert.AreEqual(10, actual.Y);
   Assert.AreEqual(12, actual.Radius);
}
```

Another exception when multiple assertions is still acceptable when the final outcome is achieved through a series of mocked function, then you may also verify the mock invocations. This is to express to the reader that the outcome does not appear out of nowhere but rather achieved through some other mocked service and making sure that it is achieved from the mocked service should be part of the test.

```C#
// It's OK - asserting the outcome and verify the mocked method invocations
[TestMethod]
public void ()
{
    // arrange
    var personId = 15;
    var rawPhoneNumber = "06 45872365";
    var formattedPhoneNumber = "+31-6-45872365";

    mockPhoneNumberRepository
        .Setup(m => m.Get(personId))
        .Returns(rawPhoneNumber);

    mockPhoneNumberFormatter
        .Setup(m => m.Format(rawPhoneNumber))
        .Returns(formattedPhoneNumber);

    // act
    var actual = Sut.GetPhoneNumber(personId);

    // assert
    Assert.AreEqual(formattedPhoneNumber, actual);
    mockPhoneNumberRepository.VerifyAll();
    mockPhoneNumberFormatter.VerifyAll();
}
```
## Favor Readibility to Elimination of Code Duplication
In contrast to production code, the readibility is enhance by hiding the detail to own method. In test code detail is important. Reading the test with all its detail in one place is very confinient. 

```C#
// Production code - Don't: 
// giving all the detail will make it harder to read
public void Authenticate(string username, string encryptedPassword)
{
    const string secret = "xdfs$dsf9vd!21cd0";

    const EncryptionMethod encryptionMethod = EncryptionMethod.RC4;

    var decryptedPassword = encryptionService.Decrypt(encryptedPassword, secret, encryptionMethod);

    var calculatedHash = hashService.CalculateHash(decryptedPassword);
    
    var storedPasswordHash = passwordHashRepository.GetByUsername(username);

    if (calculatedHash != storedPasswordHash);
    {
        throw new InvalidOperationException("Incorrect password");
    }

}

// Production code - Do:
// information hiding is beneficial for the reader
public void Authenticate(string username, string encryptedPassword)
{
    var decryptedPassword = DecryptPassword(encryptedPassword);
    
    if (CalculateHash(decryptedPassword) != GetStoredPasswordHash(username))
    {
        throw new InvalidOperationException("Incorrect password");
    }
}

```

But in the test code having all information in the test method body helps the reader to figure out which information is important for the test.

```C#
// Test code - Don't do this
[TestMethod]
public void Authenticate_WithCorrectPassword_ShouldNotThrow()
{
    // arrange
    var username = "john.doe";
    var encryptedPassword = "ab2d309vsdf12d93sfds";

    // Setting up the mocks, although the reader will clearly understand 
    // the intention but some details are hidden.
    // It is unclear which setups are made and which mocks are incorporated.
    // In case of fixing the broken code, the coder should jump back and forth
    // from the test code and the SetupMocks method. It is easier when everyting
    // can be read from a single place.
    SetupMocks(username, encryptedPassword); 

    // act
    Sut.Authenticate(username, encryptedPassword);
    
    // assert
    mockDecryptionService.VerifyAll();
    mockHashService.VerifyAll();
    mockPasswordHashRepository.VerifyAll();
}

// Test code - Do this
[TestMethod]
public void Authenticate_WithCorrectPassword_ShouldNotThrow()
{
    // arrange
    var username = "john.doe";
    var encryptedPassword = "ab2d309vsdf12d93sfds";
    var decryptedPassword = "decrypted-password";
    var hash = "the-hash";

    mockDecryptionService
        .Setup(m => m.Decrypt(
            value: encryptedPassword, 
            secret: "xdfs$dsf9vd!21cd0", 
            method: EncryptionMethod.RC4))
        .Returns(decryptedPassword);

    mockHashService
        .Setup(m => m.CalculateHash(decryptedPassword))
        .Returns(hash);
    
    mockPasswordHashRepository
        .Setup(m => m.GetHash(username))
        .Returns(hash);

    // act
    Sut.Authenticate(username, encryptedPassword);
    
    // assert
    mockDecryptionService.VerifyAll();
    mockHashService.VerifyAll();
    mockPasswordHashRepository.VerifyAll();
}
```

## Setup Mock Sepcifically 
- don't use It.IsAny<>
- make it difficult for any other conditions

## Verify Negative Invocation Generally
- use It.IsAny<>
- make it easy for any other conditions

## Setup Mock Within Test
- Mock setup in the test-setup-section is difficult to maintain
- Common mock setup is difficult to read

## Minimal Mock Setup
- Don't create mock setups more than it's needed

## Pros and Cons of Builders (for mock or input)
As for mock
Pros:
- Enhance the readibility
Cons:
- If you are the one who is developing the interface (not an existing interface) then creating a builder is an indication that the interface might be too complicated, probably from having too many responsibilities.

As for input
Pros:
- Enhance the readibility, simplify a lot
Cons:
- Hiding some details about the input
- Builder can internally setup more than it's needed for the test


## Time-Box While Creating A Test
- too long indicates that the production code might be too complex --> simplify by creating a mock of a service



