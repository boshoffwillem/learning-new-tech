# SpecFlow

## Why use SpecFlow

SpecFlow is used to write business readable automated tests.

### Business readable automated tests

Tests that run automatically to verify a system is working as expected,
and that document the system in a way that non-technical people
can understand and contribute to.

### Why business readable automated tests

"Build the right thing, and build it right."

- **Documentation**: once we have our tests, they help
document the system in a high-level way.
- **Execution**: this documentation is also executable,
so it is like "living" documentation.
- **Communication**: because these tests are high-level
and readable, they help with communication between all the
stakeholders to ensure that we are all building the same thing.

#### Documentation

These tests help document:

- What features does the system have?
- How do those features work?
- What different usage scenarios does the system support?
- What does the system not support?

These tests lives alongside the source code.

They help with the on-boarding of new developers.

They help with the on-boarding of new business people.

They provide benefits in audit scenarios.

#### Execution

These tests verify that the system behaves as expected and forms
part of the whole test suite.

This living documentation automatically stays up to date with
the actual code.

It is business readable documentation mapping to actual testable code.

#### Communication

Enables better communication between developers and business.

A common language that everyone understands.

This ensures that the correct features are being built.

This enhanced communication ensures we cover all the different usage
scenarios and any edge cases.

## Types of automated tests

We can use SpecFlow to write:

- Unit tests
- Integration tests
- API tests
- Functional UI tests

Ultimately SpecFlow can sit on type of automated test.

## Types of testing workflows

SpecFlow can fit into the following testing workflows:

- BDD
- ATDD
- Add SpecFlow tests to existing code

Fortunately SpecFlow is not an all-or-nothing approach.
You can start by focussing on the gaps or critical areas.

### Example workflow

Suppose where are in an agile environment.
The team discusses the items of the next sprint.

For each item a list of SpecFlow scenarios are written.
Then the whole team reviews those scenarios (especially reviewed
from a business point-of-view).

Now coding starts.

A ticket is considered as complete when the SpecFlow tests pass,
not when a PR is approved.

## Living documentation

### Conventional way

Usually you have your source code which is the actual source of truth
for a system.

Then you have your documentation in a separate place, a Wiki, network
share, etc.

The problem is that the documentation is a separate place and there are
no strong links between the code and the documentation. So keeping the
documentation up-to-date is very manual, and usually gets out of sync
very quickly.

This creates uncertainty and ambiguity between developers, testers, and
business people, and impacts the team in a negative way.

### SpecFlow way

We have our source code.

But, alongside that, in the same repo, we have our specifications/documentation.

We now have a single source of truth we both the code
and the specifications stays in sync.

Not only do the specifications act as documentation because they are readable,
they also acts as tests because they are executable.

Disjoints in the code and the documentation is immediately caught as a failing test.

## SpecFlow fundamentals

### Feature files

Feature files structure:

- **Header**: Name and description of feature.
- **Scenario**: We can have multiple scenarios.
- **Steps**: Logical steps described in a high-level way.

#### Feature headers

When writing feature headers take into consideration:

- Who benefits or is interested in the feature.
- What is required/necessary?
- Why is this feature important/valuable?

#### Steps

SpecFlow uses a DSL called Gherkin.
Gherkin uses certain keywords to define steps:

- **Given**: Setup starting point of the system.
- **When**: Describe the action that takes place.
- **Then**: Observe and verify outcomes/end state.

SpecFlow also has **And** and **But** steps.

You can also add comments, lines starting `#`,
and tags, using the `@` symbol.

Tags provide a way to organize/group tests.
If you add tags to scenarios or features, you can specify
to only run scenarios or features of a specific tag.
If you had tags at a feature level (the highest level)
all the scenarios in that feature inherit the tags.

##### Example

```Gherkin
@health @fighting
Scenario: Starting health
    Given I'm a new player
        And I'm an Elf
        But I'm not wearing any armour
    # All players start with 100 health
    When I take 40 damage
    Then my health should now be 60
        And I should still be standing
```

## SpecFlow to code binding

All the SpecFlow steps need to bind actual methods in code.
There are two ways two bind to methods:

- With **regular expressions**: This requires usage of attributes on the methods,
but provides flexibility with refactoring and naming.
- By **naming convention**: This removes the need for attributes, but can make
refactoring and naming a bit harder. Naming convention must be **Pascal case**
and/or **Underscore** style.

### Underscore binding style

```csharp
[Given]
public void Given_I_m_a_new_player()
{
    _player = new PlayerCharacter();
}
```

### Pascal case binding style

```csharp
[Given]
public void GivenIMANewPlayer()
{
    _player = new PlayerCharacter();
}
```

### Regular expression binding style

```csharp
[Given(@"I'm a new player")]
public void GivenIMANewPlayer()
{
    _player = new PlayerCharacter();
}
```

## Increasing maintainability

There are four ways to improve maintainability:

- **Shared step definitions**: allows for code reuse and parameters
- **Scenario outlines**: avoids duplication in the scenario text.
Replace multiple similar scenarios with a scenario outline. We can then
execute this scenario multiple times with multiple sets of test data.
- **Scenario step data tables**: help improve readability by replacing
multiple Ands with a table of data. This tabular data can be passed
through to the step definition and the individual rows and columns
can be accessed.
- **Scenario backgrounds**: help improve readability by replacing
multiple Givens with a scenario background.

### Shared step definitions

Shared steps not only allow for code reuse, but you can also configure
steps with parameters that you can change for every scenario. This is
almost like using `Theory` with the `xUnit` testing library.

Parameterized step definitions allow values to be passed from scenario
steps to test automation code.

Using parameterized step definitions promotes the reuse of step definition
code by allowing scenario steps to share code where the only difference is
in the value of data items.

Depending on the binding style, the parameters are added differently.

#### Regular expression binding parameters

Add "holes" in attribute. "Holes" take the form of regular expression matches.
Values are passed to step method parameters. For example,

```csharp
[When(@"I take (.*) damage")]
public void WhenITakeDamage(int damage) {}
```

or

```csharp
[When(@"I take (.*) damage from a (.*)")]
public void WhenITakeDamageFromASword(int damage, string weaponType) {}
```

#### Underscore binding parameters

Add "holes" in uppercase.
Parameters can be positional, e.g., `P0`, `P1`, etc.
Or, they can be named, e.g., `DAMAGE`, `WEAPONTYPE`, etc.

```csharp
[When]
public void When_I_take_P0_damage(int p0) {}
```

or

```csharp
[When]
public void When_I_take_DAMAGE_damage(int damage) {}
```

Multiple parameters:

```csharp
[When]
public void When_I_take_P0_damage_from_a_P1(int p0, string p1) {}
```

or

```csharp
[When]
public void When_I_take_DAMAGE_damage_from_a_WEAPONTYPE(
    int damage, string weaponType) {}
```

#### Pascal case binding parameters

Add "holes" in uppercase.
Parameters can be positional, e.g., `P0`, `P1`, etc.
Or, they can be named, e.g., `DAMAGE`, `WEAPONTYPE`, etc.

```csharp
[When]
public void WhenITake_P0_Damage(int p0) {}
```

or

```csharp
[When]
public void WhenITake_DAMAGE_Damage(int damage) {}
```

Multiple parameters:

```csharp
[When]
public void WhenITake_P0_DamageFromA_P1(int p0, string p1) {}
```

or

```csharp
[When]
public void WhenITake_DAMAGE_DamageFromA_WEAPONTYPE(
    int damage, string weaponType) {}
```

### Scenario outlines

Another way to increase maintainability is with scenario outlines.

Scenario outlines allow the same basic scenario to be executed multiple
times, each time with different test data.

Using scenario outlines allows the reduction or elimination of repeated
scenarios where the only difference between them are the inputs
or expected outcomes.

#### Creating a scenario outline

You specify values that are variables with `<>`. Then you add an `Examples`
keyword and a table structure that contains the values. Every row in the
table will be the values for a new scenario, every column is one of the
variables, for example,

```gherkin
Scenario Outline: Health reduction
Given I'm a new player
When I take <damage> damage
Then my health should now be <remainingHealth>
Examples:
    | damage | remainingHealth |
    | 0      | 100             |
    | 40     | 60              |
```

#### Better example

Let's suppose we have the following scenarios:

``` Gherkin
Feature: PlayerCharacter
    In order to play the game
    As a human player
    I want my character attributes to be correctly represented
    
Scenario: Taking no damage when hit doesn't affect health
    Given I'm a new player
    When I take 0 damage
    Then My health should now be 100
    
Scenario: Starting health is reduced when hit
    Given I'm a new player
    When I take 40 damage
    Then my health should now be 60
    
Scenario: Taking too much damage results in player death
    Given I'm a new player
    When I take 100 damage
    Then I should be dead
```

We can see that there is some duplication in the first two scenarios.
We'll address this by creating a scenario outline.

``` Gherkin
Feature: PlayerCharacter
    In order to play the game
    As a human player
    I want my character attributes to be correctly represented
    
Scenario Outline: Health reduction
    Given I'm a new player
    When I take <damage> damage
    Then my health should now be <remainingHealth>
    Examples:
    | damage | remainingHealth |
    | 0      | 100             |
    | 40     | 60              |
    
Scenario: Taking too much damage results in player death
    Given I'm a new player
    When I take 100 damage
    Then I should be dead
```

### Scenario backgrounds

Scenario backgrounds allow the moving of duplicated, non-essential,
identical, repeated `Given` steps to a common section.

Steps within the background section will be automatically executed
before each scenario or scenario outline.

#### Scenario background example

Remove duplicated `Given I'm a new player` step.
Add a scenario background.
Move given step to background.
Taking the previous example:

``` Gherkin
Feature: PlayerCharacter
    In order to play the game
    As a human player
    I want my character attributes to be correctly represented
    
Scenario Outline: Health reduction
    Given I'm a new player
    When I take <damage> damage
    Then my health should now be <remainingHealth>
    Examples:
    | damage | remainingHealth |
    | 0      | 100             |
    | 40     | 60              |
    
Scenario: Taking too much damage results in player death
    Given I'm a new player
    When I take 100 damage
    Then I should be dead
```

`Given I'm a new player` is both repeated in a scenario outline
and in a scenario. Let's fix this by adding a `Background` section
after `Feature` and place our `Given`s there.

``` Gherkin
Feature: PlayerCharacter
    In order to play the game
    As a human player
    I want my character attributes to be correctly represented

Background:
    Given I'm a new player
    
Scenario Outline: Health reduction
    When I take <damage> damage
    Then my health should now be <remainingHealth>
    Examples:
    | damage | remainingHealth |
    | 0      | 100             |
    | 40     | 60              |
    
Scenario: Taking too much damage results in player death
    When I take 100 damage
    Then I should be dead
```

## Working with data in step definitions

How does SpecFlow convert text to .Net code?
This conversion includes:

- Step argument conversion
- Built-in automatic / custom transforms
- Automatic enum conversion
- Weakly-typed table data to strongly-typed table data
- There is also a nuget package to convert SpecFlow
table data to dynamic C# objects
- Custom data transforms, e.g., transforming
"3 days ago" --> DateTime
- Automatically applying custom transforms
- Passing data between step definitions (thread-safe
and non-thread-safe ways, and strongly-and weakly-typed versions)

### Step argument conversion

There is an order of precedence when conversion happens:

- **No conversion**: `object` or `string` data types
- **Custom step arguments**: custom transform data types
- **Standard (inbuilt) conversion**: first SpecFlow tries `Convert.ChangeType()`
then, `Text --> enum value`, or if parameter is a GUID then `Text --> GUID`

#### Automatic enum conversion

Suppose we have an enum

```csharp
public enum CharacterClass
{
    None,
    Healer,
    Warrior
}
```

Let's add a new scenario,

``` Gherkin
Feature: PlayerCharacter
    In order to play the game
    As a human player
    I want my character attributes to be correctly represented

Background:
    Given I'm a new player
    
Scenario Outline: Health reduction
    When I take <damage> damage
    Then my health should now be <remainingHealth>
    Examples:
    | damage | remainingHealth |
    | 0      | 100             |
    | 40     | 60              |
    
Scenario: Taking too much damage results in player death
    When I take 100 damage
    Then I should be dead

Scenario: Healers restore all health
    Given My character class is set to Healer
    When I take 40 damage
        And Cast a healing spell
    Then My health should now be 100
```

Let's look at the C# code:

```csharp
[Given(@"My character class is set to (.*)"]
public void GivenMyCharacterClassIsSetToHealer(CharacterClass characterClass)
{
    _player.CharacterClass = characterClass;
}

[When(@"Cast a healing spell"]
public void WhenCastAHealingSpell()
{
    _player.CastHealingSpell();
}
```

SpecFlow will automatically convert the string value to the correct enum value.

#### Strongly-Typed step table data

We will add the `TechTalk.SpecFlow.Assist` nuget package,
which contains some useful extension methods.

``` Gherkin
Feature: PlayerCharacter
    In order to play the game
    As a human player
    I want my character attributes to be correctly represented

Background:
    Given I'm a new player
    
Scenario Outline: Health reduction
    When I take <damage> damage
    Then my health should now be <remainingHealth>
    Examples:
    | damage | remainingHealth |
    | 0      | 100             |
    | 40     | 60              |
    
Scenario: Taking too much damage results in player death
    When I take 100 damage
    Then I should be dead

Scenario: Healers restore all health
    Given My character class is set to Healer
    When I take 40 damage
        And Cast a healing spell
    Then My health should now be 100

Scenario: Elf race characters get additional 20 damage resistance
    using data table
        And I have the following attributes
        | attribute  | value |
        | race       | Elf   |
        | Resistance | 10    |
    When I take 40 damage
    Then My health should now be 90
```

Let's look at the C# code for accessing the table data
in a weakly-typed manner:

```csharp
[Given(@"I have the following attributes")]
public void GivenIHaveTheFollowingAttributes(Table table)
{
    var race = table.Rows.First(row => row["attribute"] == "Race"["value"];
    var resistance = table.Rows.First(row => row["attribute"] == "Resistance"["value"];
    _player.Race = race;
    _player.DamageResistance = int.Parse(resistance);
}
```

Now, let's write code to access the data in a strongly-typed manner.
Let' make a `PlayerAttributes` class.

```csharp
public class PlayerAttributes
{
    public string Race { get; set; }
    
    public int Resistance { get; set; }
}
```

Then where our `Given` step is add a new `using` for `TechTalk.SpecFlow.Assist`

```csharp
[Given(@"I have the following attributes")]
public void GivenIHaveTheFollowingAttributes(Table table)
{
    var attributes = table.CreateInstance<PlayerAttributes>();
    _player.Race = attributes.Race;
    _player.DamageResistance = attributes.Resistance;
}
```

SpecFlow will automatically convert the string value to the correct enum value.

### Custom Data Conversions

Suppose we have the scenario:

```Gherkin
Scenario: Reading a restore health scroll when over tired has no effect
    Given I last slept 3 days ago
    When I take 40 damage
        And I read a restore health scroll
    Then My health should now be 60
```

We are going to create custom converters for the date.
We create the code for the `Given` step.

```csharp
[Given(@"I last slept (.* days ago)")]
public void GivenILastDaysAgo(DateTime lastSlept)
{
    _player.LastSleepTime = lastSleep;
}
```

We then create an additional class containing our converters.
(The class will be in th same namespace as the step methods).

```csharp
[Binding]
class CustomConversions
{
    [StepArgumentTransformation(@"(\d+) days ago")]
    public DateTime DaysAgoTransformation(int daysAgo)
    {
        return DateTime.Now.Subtract(TimeSpan.FromDays(daysAgo));
    }
}
```

Because of the attributes this custom method will be executed when a step is
executed with parameters matching the regex of this method.

### Automatically applying custom transforms

Add new scenario

```Gherkin
Scenario: Weapons are worth money
    Given I have the following weapons
    | name  | value |
    | Sword | 50    |
    | Pick  | 40    | 
    | Knife | 10    |
    Then My weapons should be worth 100
```

Now we create the step definitions

```csharp
[Given(@"I have the following weapons")]
public void GivenIHaveTheFollowingWeapons(Table table)
{

}

[Then(@"My weapons should be worth (.*)"]
public void ThenMyWeaponsShouldBeWorth(int value)
{
    Assert.Equal(value, _player.WeaponsValue);
}
```

Now, with the power of transforms, we can change the dynamic,
ambiguous `Table` to something more clear and concrete,
like `IEnumerable<Weapon>`:

```csharp
[Given(@"I have the following weapons")]
public void GivenIHaveTheFollowingWeapons(IEnumerable<Weapon> weapons)
{
    _player.Weapons.AddRange(weapons);
}
```

Now, in our `CustomConversions` class, we add a new method
(we are working with a `Table` so remember to add the
`TechTalk.SpecFlow.Assist` namespace):

```csharp
[StepArgumentTransformation]
public IEnumerable<Weapon> WeaponsTransformation(Table table)
{
    return table.CreateSet<Weapon>();
}
```

Notice that we don't supply a regex to the `[StepArgumentTransformation]`.
This means that this conversion will get called every time we have a Step
whose arguments' type matches the return type of this method.

### Passing data between step definitions

There are different ways of passing data between step definitions:

- **Step class fields or properties**: This becomes a problem when you have
multi-class bindings, and any solutions to this will not be thread/parallel safe.
- **SpecFlow provided context objects**: This way is convenient, and thread-safe,
however, we have to work with weakly-typed objects.
- **Custom context objects injection**: Not as convenient as previous method,
because we have to write some additional classes, however,
this method is strongly-typed.

#### Step class fields or properties

We can share data between step definitions by writing to the `ScenarioContext`.
This is just dictionary key on the name of the parameters.

```csharp
[Given(@"I have an amulet with a power (.*)")]
public void GivenIHaveAnAmuletWithAPowerOf(int power)
{
    ScenarioContext.Current["power"] = power;
}

[Then(@"The amulet power should not be reduced")]
public void ThenAmuletPowerShouldNotBeReduced()
{
    int expectedPower = (int)ScenarioContext.Current["power"];
}
```

As you can see this neither convenient not thread-safe.
