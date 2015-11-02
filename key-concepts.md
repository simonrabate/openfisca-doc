# Key concepts

This section presents the key concepts required to have a good understanding of OpenFisca,
without being too technical.
The next sections of the documentation are more specific.

We use the French legislation to illustrate these concepts as it is the only actively maintained country for now.
French names are kept as is.

## Tax and benefit system

The tax and benefit system is the higher-level object in OpenFisca.
Its goal is to model the legislation of a country.

Basically a tax and benefit system contains simulation variables (source code) and legislation parameters (data).

The OpenFisca core engine is able to simulate any country legislation once it is (partially) represented as source code.

## Periods and instants

OpenFisca manipulates time via periods and instants.

The atomic unit is a day, so instants are day dates.

Periods are defined using an ad-hoc strings format.
Internally, they are stored as a start instant, a unit (month, year) and a quantity of units.

For example:

- `"2015"` is a year, `"2015-01"` is a month, `"2015-06:3"` are the 3 months
june, july and august of the year 2015. They are all periods.
- `"2015-02-15"` is an instant.

Functions exist to transform periods or turn them into an instant, which are documented later.

## Variables

Variables can be calculated or input.

Input variables are normal variables but don't have any formula defined.
Their value is given by the caller who runs the simulation.

Calculated variable have a formula which can be bypassed if an actual value is given.
Calculated variables have dependencies: variables involved in the formula.
The dependencies are either calculated, either looked in input variables. If not found, default values are used
(most of the time `0`).

Variables values (input or calculated) are associated to a period.

Input variables are defined given a period. For example you give a salary amount for a period of 6 months.

When a variable is called, a period is given.
For example you want to know the amount of a tax for a specific year.

But sometimes the asked period isn't calculable so the core engine returns the value for another period.
There are some strategies to deal with these periods mismatch, which are documented later.

The [legislation explorer](http://legislation.openfisca.fr/) presents each variable under its own URL.
For example [`irpp`](http://legislation.openfisca.fr/variables/irpp).

## Parameters

Parameters are data, variables are algorithms.

Each time a part of the legislation is data, it should be stored in a parameters file
instead of creating a variable which returns a static value.

Parameters are read by variables formulas.

Legislation parameters are stored in an XML file:
[`param.xml`](https://github.com/openfisca/openfisca-france/blob/master/openfisca_france/param/param.xml).

They can be simple values or scales and are stored by date to collect the past values.
Even with simple values, there can be many values for the same parameter because of the multiple dates.

For instance, the calculation of the `irpp` variable will involve the `ir.recouvrement.min` parameter, also present in the [legislation explorer](http://legislation.openfisca.fr/parameters/ir.recouvrement.min).

There are functions to extract a subset of the whole legislation at a specific instant,
which is necessary when writing the formula of a variable.

The [legislation explorer](http://legislation.openfisca.fr/) presents each parameter under its own URL.
For example [a simple value](http://legislation.openfisca.fr/parameters/prelsoc.rsa)
or [a scale](http://legislation.openfisca.fr/parameters/ir.bareme)

## Simulation

A simulation is basically a cache of previously computed results.

To calculate any variable you need to create a `Simulation` from the `TaxBenefitSystem`.

It's possible to run many independent simulations using the same `TaxBenefitSystem`.

## Persons, entities and roles

OpenFisca models the real world legislation.
Each tax and benefit concerns either individual persons or entities.

Entities are groups of persons like a family, a household or a company.
The legislation defines many entities and specifies which tax and benefit applies to which entity.

The entities definitions are closely related to a country, therefore they are defined in a Python package
independent from the core engine (ie OpenFisca-Core / OpenFisca-France).

In France the legislation defines these entities: `"familles"`, `"foyers_fiscaux"` and `"menages"`.

Each person related to an entity has a role. For `"familles"` the roles are "`parents`" and `"enfants"` (children).

You can define as many entities as you want and dispatch persons into them.

## Test cases or data

OpenFisca can take test cases or data (surveys with aggregated data or real population data)
as input when calculating variables.

A test case describes persons and entities with their input variables
whereas data contains potentially a huge quantity of persons and entities.

Test cases can be expressed in Python or in JSON when using the Web API (see specific sections of the documentation).

Here is a test case sample in JSON for a single person with 3 children:

```json
{
  "familles": [
    {
      "parents": ["parent1"],
      "enfants": ["enfant1", "enfant2", "enfant3"]
    }
  ],
  "foyers_fiscaux": [
    {
      "declarants": ["parent1"],
      "personnes_a_charge": ["enfant1", "enfant2", "enfant3"]
    }
  ],
  "individus": [
    {
      "id": "parent1",
      "birth": "1980-01-01",
      "salaire_de_base": 15000
    },
    {
      "id": "enfant1",
      "birth": "2005-01-01"
    },
    {
      "id": "enfant2",
      "birth": "2003-01-01"
    },
    {
      "id": "enfant3",
      "birth": "1997-01-01"
    }
  ],
  "menages": [
    {
      "personne_de_reference": "parent1",
      "enfants": ["enfant1", "enfant2", "enfant3"],
      "loyer": 1500
    }
  ]
}
```

Notice the input variables associated to the `"individus"` (`"birth"` and `"salaire_de_base"`)
and to the entity `"menages"` (`"loyer"`).

This is quite verbose but there are shortcuts to generate a test case in common situations.

Using data as input is not documented yet. Please consult this repository:
https://github.com/openfisca/openfisca-france-data

## Scenarios

To support both test cases and data, and for performance reasons,
OpenFisca is developed using vector computing via the
[NumPy](http://www.numpy.org/) Python package.

Whatever the input is, a test case or data, OpenFisca will transform it to vectors internally.

Scenarios take a test cases as input and convert them into vectors.

They even offer a way to simplify the declaration of test cases when it contains only a single entity of each type.

## Thinking in vectors

Because OpenFisca needs to accept test cases or data as input, it uses vector computing.
So we need to reason always with vectors instead of values.

Let's dive into OpenFisca's internals sightly.

Each variable of the tax and benefit legislation is represented by a vector.
The size of a vector is equal to the number of entities the variable is defined on.

This applies to all variables, whether or not calculated.

For example let's take the input variable [`"age"`](http://legislation.openfisca.fr/variables/age)
which is defined on `"individus"`. Say there are 3 persons defined, the vector contains 3 ages: `[30, 25, 15]`.

Now let's take the computed tax variable [`"irpp"`](http://legislation.openfisca.fr/variables/irpp)
which is defined on `"foyers_fiscaux"`. Say there is 1 `"foyer_fiscal"` defined, containing the 3 persons above,
the vector contains 1 value (here is a dummy value): `[999]`.

Test cases are just a syntactic sugar to generate vectors. This is a completely optional layer.

Entities and roles are encoded using special variables `"id*"` and `"qui*"` whose vectors contain respectively
unique identifiers of an entity and integers corresponding to a role.

For example, for `"familles"` we have
[`"idfam"`](http://legislation.openfisca.fr/variables/idfam) and
[`"quifam"`](http://legislation.openfisca.fr/variables/quifam).
For `"quifam"` the role `0` is "chef" (head of family), the role `1` is "part" (partner).

With `"id*"` and `"qui*"` variables, entities are modeled so the core engine knows which person is into which entity.

For example the following test case defined for the year `2015`:

```json
{
  "familles": [
    {
      "parents": ["parent1"],
      "enfants": ["enfant1", "enfant2", "enfant3"]
    }
  ],
  "foyers_fiscaux": [
    {
      "declarants": ["parent1"],
      "personnes_a_charge": ["enfant1", "enfant2", "enfant3"]
    }
  ],
  "individus": [
    {
      "id": "parent1",
      "birth": "1980-01-01",
      "salaire_de_base": 15000
    },
    {
      "id": "enfant1",
      "birth": "2005-01-01"
    },
    {
      "id": "enfant2",
      "birth": "2003-01-01"
    },
    {
      "id": "enfant3",
      "birth": "1997-01-01"
    }
  ],
  "menages": [
    {
      "personne_de_reference": "parent1",
      "enfants": ["enfant1", "enfant2", "enfant3"],
      "loyer": 1500
    }
  ]
}
```

can be expressed with these vectors:

```json
{
  "activite": [ 4, 2, 2, 4 ],
  "birth": [ "1980-01-01", "2005-01-01", "2003-01-01", "1997-01-01" ],
  "idfam": [ 0, 0, 0, 0 ],
  "idfoy": [ 0, 0, 0, 0 ],
  "idmen": [ 0, 0, 0, 0 ],
  "quifam": [ 0, 2, 3, 4 ],
  "quifoy": [ 0, 2, 3, 4 ],
  "quimen": [ 0, 2, 3, 4 ],
  "salaire_de_base": [ 15000.0, 0.0, 0.0, 0.0 ],
  "loyer": [ 1500.0 ]
}
```

Notice the different sizes of the vectors, and the disappearance of the person identifiers
(`"parent1"`, `"enfant1"`, etc.).

Then OpenFisca core engine can start calculating the asked variables, using the given input variables as any
calculated variable result.

## Extensions
An extension represents a modified version of the tax and benefit legislation.

For example it can be used to add, remove or modify a variable, or a legislation parameter.

The tax and benefit system of the country knows about the laws that are already adopted, were existing in the past,
or will exist in a near future. In contrast, the extensions are used for political reforms or
propositions that people do but are not officially voted.

Extensions do not modify the `TaxBenefitSystem` itself, they create a shallow copy and modify only what changed.

Extensions are loaded given a base `TaxBenefitSystem` and return an extended one.

As a consequence extensions can be composed (`ext2(ext1(tax_benefit_system))`).

Extensions can be published in their own `git` repository.

How to write an extension is documented [in this section](./openfisca-in-python/extensions.md).

## Writing some legislation

From the point of view of someone (developer, economist, etc.) who wants to implement a part of the legislation,
for example a new benefit, here are some key steps:

- understand the part of the legislation you want to implement
- identify the variable dependencies using the [legislation explorer](http://legislation.openfisca.fr/)
- identify the new variables you need to implement
- write the new variables with their formulas
- store the new parameters
- if you implement a part of the official legislation, your code should go in OpenFisca-France,
  but if you implement a new idea or a future reform, your code should go in an extension.