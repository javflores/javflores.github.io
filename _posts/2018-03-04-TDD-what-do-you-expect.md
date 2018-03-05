---
layout: post
title: "TDD, what do you expect?"
author: jflores
date: 2018-03-04 09:00:00
categories: [tdd, xp]
summary: "Think about what do you expect when doing TDD (and a few other thoughts along the way)"
---

TDD, Test driven development.
If you don't know what it is I envy and salute you.
These days TDD is like one of those other terms, eg. Agile, MVC, Lean, SOLID.
We love acronyms in our profession, and as developers we need to know them all, even if we don't use it.
In my opinion that is because it is used by people that don't fully understand the value of them and they use it as a weapon against us.
They are even making millions by using them, can you imagine?

Worst of all, we have created a culture of *you need to know this* so we've ended with a massive population of people repeating the acronyms without really knowing them:

> Me, of course I know TDD, I use it every day.

*Comment of a person who has just checked in some to Github without a single unit test.*

For me TDD always was about stepping into the unknown. 
Similar to a baby who starts with her first steps. She sees something spiky in front, she doesn't put the whole foot in it. She approaches the surface, while building an expectation (*how is it going to feel? Cold? Is it going to hurt? How much?*)
In order to prove that expectation she slightly touches the surface with the big toe and checks if the expectation still hold.

That's what we should be doing with software, do baby steps while we step into the unknown. 
TDD helps me in that regard, I implement one little step at a time as I learn more about the system. 
I know some people are proud to say they never follow TDD, maybe I'll be the same if I were to build exactly the same system over and over again (that's why [Dave Thomas does not do TDD anymore](https://youtu.be/a-BOSpxYJ9M?t=1130) is proud to be a not TDD person).

## TDD: the Triple-A

Arrange, Act, Assert, AAA. I was introduced to TDD by following that practice: 

- Arrange: Establish the context where something is going to happen.
- Act: Make that thing happen.
- Assert: Did that thing happen?

I even put comments in my code to let others know that I was following TDD:

```cs
public void Some_test_name_I_will_rename_in_the_future_because_now_I_am_too_lazy_to_think_about_it ()
{
  //Arrange
  
  //Act
  
  //Assert
} 
```

I think I even checked some code to Version Control with those comments. 

Sometimes I see people leaving those comments in.

In this post I wanted to show you a subtle but different way to write your unit tests.

## What is the problem?

Again and again, when I pair with people we start with the Arrange part of the test. 
What is the problem with that?

- With TDD one of the goals is to write the minimum code that satisfies a expectation. If we start by writing the Arrange we have already broken the rule: we don't have yet an expectation and we have some code.

- People tend to start the Arrange of some new test by copying the arrange of a previous test. In many occasions we leave some code that wasn't really needed.

- In some frameworks like Mocha in Javascript, some part of the Arrange is defined in a `before` (`beforeEach`) that only allows to set a function. That is even worst since we can't name the Context properly.

## What do you expect?

For some time now, I have been practicing a different way to do TDD. We still follow the AAA but backwards, starting with the Assert. 
It seems silly but I found multiple advantages. It seems easy but you'll have to fight against your normal tendency.

Let's do an example:

---

*When a customer buys a subscription with Paypal we store his email in a table. If we renew his subscription later on, we don't save his paypal email again as this wasn't a manual payment.
We found a bug in a situation where we need to retrieve his paypal email but we are getting an empty email.*

---

Let's fix this bug together. The code is in C# and the tests are written in Machine Spec, let's follow those for now.

This is the buggy code:

```cs
public string GetPayPalEmailByMemberId(int member)
{
    return this._transRepository
        .FindAllSubscriptionsForMember(member)
        .OrderByDescending(m => m.DateAdded)
        .FirstOrDefault(m => m.PaymentProvider == PaymentProviderEnum.PP)?
        .PayPalEmail;
}
```

I know, it is legacy code, it takes a bit to understand (although we've seen worst). Let's explore (and fix) this code through Unit tests.

### Expectations, the What

As sometimes happens with Legacy code, the class that has the bug doesn't have any unit tests around it. 
Let's start with an empty file and we put empty blocks that will compose the test:

```cs
public class PaymentRepositoryTests
{
    public class when_user_has_bought_subscription_through_paypal
    {
        Establish context;

        Because of;

        It should_find_their_paypal_email;
    }
}
```

We have described what we expect, now we can nail the expectation:

```cs
public class PaymentRepositoryTests
{
    public class when_user_has_bought_subscription_through_paypal
    {
        Establish context;

        Because of;

        It should_find_their_paypal_email = () => _paypalEmail.ShouldEqual("paypalEmail@fmp.com");
    }
}
```

You don't see it but my intellisense is painting the variable `_paypalEmail` with red, it doesn't exist. 
At this point proably you are feeling the urge to create that variable, you are fighting against the autocompletion and that doesn't feel good.

Keep going! In a moment you'll see the value of this.

### Act, the How

Now we need to retrieve that paypal email, we are ready to invoke the logic that needs to be tested.

```cs
public class when_user_has_bought_subscription_through_paypal
{
    Establish context;

    Because of = () => _paypalEmail = _paymentRepository.GetPayPalEmailByMemberId(_memberId);

    It should_find_their_paypal_email = () => _paypalEmail.ShouldEqual("paypalEmail@fmp.com");
}
```

We now have called the buggy piece of functionality. We will retrieve the paypal email for a given member using the payment repository. There are a couple more variables not yet defined, the code does not compile, but that's fine, we are still building our test.

### Arrange, the When

In order to be able to find a member's paypal email we need a payment made by that member.
Our payment repository has a dependency in a module that returns all the payments made by a member.
We will make the Arrange happen by mocking that other module to return a payment made with paypal:

```cs
public class PaymentRepositoryTests
{
    public class when_user_has_bought_subscription_through_paypal
    {
        const int MemberId = 123;
        const string ExpectedPaypalEmail = "paypalEmail@fmp.com";

        Establish context = () =>
            {
                var paypalTransaction = new MemberTrans()
                                      {
                                          PayPalEmail = ExpectedPaypalEmail,
                                          PaymentProvider = PaymentProviderEnum.PP
                                      };
                var memberTransactions = new List<MemberTrans>(){ paypalTransaction };

                MockOf<IMemberTransRepository>()
                .Stub(x => x.FindAllSubscriptionsForMember(MemberId))
                .Return(memberTransactions.AsQueryable());
            };

        Because of = () => _paypalEmail = _paymentRepository.GetPayPalEmailByMemberId(MemberId);

        It should_find_their_paypal_email = () => _paypalEmail.ShouldEqual(ExpectedPaypalEmail);
    }
}
```

When asked about transactions for a member, the mock will return a transaction with the expected member.
We have defined a couple of constants to remove duplication and make the code easier to read.

Finally we need to get our `subject under test, sut`, the payment repository.
To do that we are going to inherit from an utility class that will create our SUT with all the dependencies mocked. It is an internal implementation but it is not difficult to guess how it works.

```cs
public class PaymentRepositoryTests
{
    public class when_user_has_bought_subscription_through_paypal : TestsContext<PaymentRepository>
    {
        const int MemberId = 123;
        const string ExpectedPaypalEmail = "paypalEmail@fmp.com";

        Establish context = () =>
            {
                var paypalTransaction = new MemberTrans()
                                      {
                                          PayPalEmail = ExpectedPaypalEmail,
                                          PaymentProvider = PaymentProviderEnum.PP
                                      };
                var memberTransactions = new List<MemberTrans>(){ paypalTransaction };

                MockOf<IMemberTransRepository>()
                .Stub(x => x.FindAllSubscriptionsForMember(MemberId))
                .Return(memberTransactions.AsQueryable());
            };

        Because of = () => _paypalEmail = sut.GetPayPalEmailByMemberId(MemberId);

        It should_find_their_paypal_email = () => _paypalEmail.ShouldEqual(ExpectedPaypalEmail);
        
        static readonly string _paypalEmail;
    }
}
```

TestsContext will provide a sut that matches the generic parameter provided, in this case the type we want to test, PaymentRepository, 
i.e. sut will be an new instance of PaymentRepository.

We are ready to run our test and... it passes!

> What? Shouldn't your test fail and then make it pass.

Right, yes and no. Since this is legacy code without any test, before fixing the bug I want to have a safety net of tests that check the expected behaviour.
These tests also allow us to understand a bit better how the legacy code behaves.
This methodology is described by Michael Feathers in his book, [Working effectively with Legacy code](https://www.amazon.co.uk/Working-Effectively-Legacy-Robert-Martin-ebook/dp/B005OYHF0A/ref=sr_1_1?ie=UTF8&qid=1520168147&sr=8-1&keywords=working+effectively+with+legacy+code).

More importantly we have found our way to writing this test the opposite way to the Classic TDD way, Arrange-Act-Assert. Even though our Arrange has 9 lines of code, we know it is the minimum code to ensure that will verify the expectation.
In my opinion it is not hard to understand this code since it is not loaded with setup code that we don't really need.

### Assert-Act-Arrange

Since the main focus is the expectation, we can do one more thing to make our code easier to follow. Let's rearrange the elements of the test:

```cs
public class PaymentRepositoryTests
{
    public class when_user_has_bought_subscription_through_paypal : TestsContext<PaymentRepository>
    {
        It should_find_their_paypal_email = () => _paypalEmail.ShouldEqual(ExpectedPaypalEmail);

        Because of = () => _paypalEmail = sut.GetPayPalEmailByMemberId(MemberId);

        Establish context = () => /.../

        private static string _paypalEmail;
        const int MemberId = 123;
        const string ExpectedPaypalEmail = "paypalEmail@fmp.com";
    }
}
```

For the person learning TDD now this change may seem crazy and it goes against our traditional thinking.
For the new person to this code, this test is a lot easier to understand as it focus in what we expect from our test module.

### Reproducing the bug

Now that we have a basic happy path cover we can add a unit test that reproduces the bug:

```cs
public class PaymentRepositoryTests
{
    public class when_user_has_bought_subscription_through_paypal : TestsContext<PaymentRepository>
    {
        [...]

        public class and_the_subscription_has_been_renewed
        {
            It should_find_their_paypal_email;

            Because of;

            Establish context;
        }
    }
}
```

I have omitted the previous test to make it easier to follow to the reader.
Notice how I have added the new test inside the other test. The reason is that we are composing our scenarios with other scenarios, redefining the initial statement with a more specific case.

Let's add the Assertion:

```cs
public class PaymentRepositoryTests
{
    public class when_user_has_bought_subscription_through_paypal : TestsContext<PaymentRepository>
    {
        [...]

        public class and_the_subscription_has_been_renewed
        {
            It should_find_their_paypal_email = () => _paypalEmail.ShouldEqual(ExpectedPaypalEmail);

            Because of;

            Establish context;
        }
    }
}
```

And the Act:

```cs
public class PaymentRepositoryTests
{
    public class when_user_has_bought_subscription_through_paypal : TestsContext<PaymentRepository>
    {
        [...]

        public class and_the_subscription_has_been_renewed
        {
            It should_find_their_paypal_email = () => _paypalEmail.ShouldEqual(ExpectedPaypalEmail);

            Because of = () => _paypalEmail = sut.GetPayPalEmailByMemberId(MemberId);

            Establish context;
        }
    }
}
```

I know it looks the same, let's wait to have a failing test to refactor.

We are ready to create the specific Arrange of this scenario. 
We know that when a user is renewed after buying a subscription through paypal, there will be a later transaction for that member without a paypal email:


```cs
Establish context = () =>
{
    var paypalTransaction = new MemberTrans()
    {
        PayPalEmail = ExpectedPaypalEmail,
        PaymentProvider = PaymentProviderEnum.PP,
        DateAdded = DateTime.Today.AddDays(-10)
    };
    
    var renewalTransaction = new MemberTrans()
    {
        PayPalEmail = null,
        PaymentProvider = PaymentProviderEnum.PP,
        DateAdded = DateTime.Today
    };
    
    var memberTransactions = new List<MemberTrans>() { paypalTransaction, renewalTransaction };

    MockOf<IMemberTransRepository>()
    .Stub(x => x.FindAllSubscriptionsForMember(MemberId))
    .Return(memberTransactions.AsQueryable());
};
```

In this case I have only put the Arrange part to make it easier to read.
Loads of duplication going on. It is good to remember that when following TDD it is ok to have bad code for a while, we have the refactor step to improve the code structure.

I try to run the test and I get this error from the test runner:

```
ERROR Machine.Specifications.SpecificationUsageException: 
There can only be one Because clause. Found 2 Becauses in the type hierarchy
```

Now it is time to refactor our two tests to make the runner happy:

```cs
public class PaymentRepositoryTests
{
    public class when_user_has_bought_subscription_through_paypal : TestsContext<PaymentRepository>
    {
        public class and_it_is_the_only_transaction
        {
            It should_find_their_paypal_email = () => _paypalEmail.ShouldEqual(ExpectedPaypalEmail);

            Because of = () => _paypalEmail = sut.GetPayPalEmailByMemberId(MemberId);

            Establish context = () => /.../
        }

        public class and_the_subscription_has_been_renewed
        {
            It should_find_their_paypal_email = () => _paypalEmail.ShouldEqual(ExpectedPaypalEmail);

            Because of = () => _paypalEmail = sut.GetPayPalEmailByMemberId(MemberId);

            Establish context = () => /.../
        }
    
        private static string _paypalEmail;
        const int MemberId = 123;
        const string ExpectedPaypalEmail = "paypalEmail@fmp.com";
    }
}
```

We have separated every scenario in two subclasses, still duplication but we can run the test.
It fails!

```cs
Expected: "paypalEmail@fmp.com"
But was:  [null]
```

Now we can go and fix the code:

```cs
public string GetPayPalEmailByMemberId(int member)
{
    return this._transRepository
        .FindAllSubscriptionsForMember(member)
        .OrderByDescending(m => m.DateAdded)
        .FirstOrDefault(m => 
            m.PaymentProvider == PaymentProviderEnum.PP && 
            !m.PayPalEmail.IsNullOrEmpty()
        )?
        .PayPalEmail;
}
```

Specifically we are interested in those paypal transactions that contain a paypal email, since those are real payments by the user.

Finally let's remove that ugly duplication in our tests:

```cs
public class PaymentRepositoryTests
{
    public class when_user_has_bought_subscription_through_paypal : TestsContext<PaymentRepository>
    {
        public class and_it_is_the_only_transaction
        {
            It should_find_their_paypal_email = () => 
              _paypalEmail.ShouldEqual(ExpectedPaypalEmail);

            Establish context = () =>
            {
                MockMemberTransactions(new List<MemberTrans>() 
                { 
                  _paypalTransaction 
                });
            };
        }

        public class and_the_subscription_has_been_renewed
        {
            It should_find_their_paypal_email = () => 
              _paypalEmail.ShouldEqual(ExpectedPaypalEmail);
            
            Establish context = () =>
            {
                MockMemberTransactions(new List<MemberTrans>() { 
                  _paypalTransaction, 
                  _renewalTransaction 
                });
            };
        }

        Because of = () => _paypalEmail = sut.GetPayPalEmailByMemberId(MemberId);

        Establish context = () =>
        {
            _paypalTransaction = new MemberTrans()
            {
                PayPalEmail = ExpectedPaypalEmail,
                PaymentProvider = PaymentProviderEnum.PP,
                DateAdded = DateTime.Today.AddDays(-10)
            };

            _renewalTransaction = new MemberTrans()
            {
                PayPalEmail = null,
                PaymentProvider = PaymentProviderEnum.PP,
                DateAdded = DateTime.Today
            };
        };

        private static void MockMemberTransactions(List<MemberTrans> transactions)
        {
            MockOf<IMemberTransRepository>()
                .Stub(x => x.FindAllSubscriptionsForMember(MemberId))
                .Return(transactions.AsQueryable());
        }

        private static string _paypalEmail;
        private static MemberTrans _paypalTransaction;
        private static MemberTrans _renewalTransaction;
        const int MemberId = 123;
        const string ExpectedPaypalEmail = "paypalEmail@fmp.com";
    }
}
```

Now we have different contexts for different scenarios.
From here we can even add further tests to ensure that the legacy functionality behaves as expected. I think I'll do that right after finishing this post.

### Recap

That was a long example, thanks for getting this far.
I hope you could follow and see the advantages of rearranging your AAA when doing TDD.

If you want to take only one thing from this post, take this:

> Every now and then change the way you do things, you will find something you may not expect.



