# MongoDB Style Guide

## Table of Contents

  1. [General](#general)
  1. [Enumerations](#enumerations)
  1. [Booleans](#booleans)
  1. [Dates](#dates)
  1. [Null and undefined](#null-and-undefined)
  1. [Other types](#other-types)
  1. [Names](#names)
  1. [Object modelling](#object-modelling)


## General

<a name="general--no-surprises"></a>
- [1.1](#general--no-surprises) **Avoid surprises**: The goal you should be striving for when creating a new schema is to [be as least surprising as you can](https://en.m.wikipedia.org/wiki/Principle_of_least_astonishment). The rules in this style guide are written with this goal in mind. If there is a trade off between ease of creating data and ease of consuming it, choose the latter.

    > Why? Under normal circumstances, code that reads your data will be much more common than code that writes it. In addition, code that consumes your schema is more likely to be written by other people.

<a name="general--mix"></a>
- [1.2](#general--mix) **Mixed types**: Don't mix values of different types in one column. Restrict yourself to one type per column

    > Why? So you don't have to typecheck or cast when you consume the data.

    | :x:           |:white_check_mark: |
    | :------------ | :---------------- |
    | 1             | 1                 |
    | "2"           | 2                 |
    | { value: 3 }  | 3                 |

<a name="general--object-schema"></a>
- [1.3](#general--object-schema) **Object columns**: If you have a column or array that contains objects, make sure all objects share the same schema

    | :x:       | :white_check_mark:     |
    | :---------------- | :------------- |
    | { value: 42 }    | { value: 42 }   |
    | { otherValue: "foo" } | { value: 43 }   |
    | { }              | { value: null } |

<a name="general--falsiness"></a>
- [1.4](#general--falsiness) **Falsiness**: Don't use `""` or `0` for their falsiness

    > Why? It's very easy to write buggy code otherwise

    | :x: height      | :white_check_mark: height  |
    | :---------------- | :------------- |
    | 182            | 182         |
    | 167            | 167         |
    | 0               | null          |


## Enumerations

An enumeration is a type that allows a limited number of values, e.g. a userType field that can contain the values `"DOCTOR"`, `"NURSE"`, `"PATIENT"` and `"ADMINISTRATOR"`

<a name="enumerations--modelling"></a>
- [2.1](#enumerations--modelling) **Modelling**: Always model your enums as uppercase string constants, e.g. `"WAITING"`, `"IN_PROGRESS"` and `"COMPLETED"`

    > Why?
    > 1. Favoring strings over booleans or numbers means your data is self-documenting. If your gender field contains the value `1`, you don't know if that means male, female or something else
    > 1. Using uppercase strings makes it easy to see at a glance whether something is an enum or a user-facing value

    | :x: gender              | :white_check_mark: gender |
    | :---------------- | :----------------- |
    | 0             | "MALE"              |
    | 1             | "FEMALE"           |

    | :x: gender              | :white_check_mark: gender |
    | :---------------- | :----------------- |
    | "男"             | "MALE"              |
    | "女"             | "FEMALE"           |

<a name="enumerations--explicit"></a>
- [2.2](#enumerations--explicit) **Explicit states**: Don't use `null`, `undefined`, or any value except upper case string constants in your enums. This includes initial, undecided or unknown states

    > Why? This makes the meaning explicit. If you have an assessmentState field that's set to `null`, you can't tell whether that means "no assessment necessary", "not applicable", "patient didn't show up", "undecided" or any other possible state

    | :x: assessmentState | :white_check_mark: assessmentState |
    | :------------------ | :--------------------------------- |
    | null                | "NOT_REQUIRED"                     |
    | *missing*           | "NOT_APPLICABLE"                   |


## Booleans

<a name="booleans--booleans-and-enums"></a>
- [3.1](#booleans--booleans-and-enums) **Booleans and enums**: Don't model things as booleans that have no natural correspondence to true/false, even if they only have two possible states. Use enums instead

    > Why?
    > 1. Booleans can not be extended beyond two possible values but the field might have to change later to include additional values, e.g. a boolean field "hasArrived" could not be extended later to include the possibility of cancelled appointments
    > 1. It makes meaning explicit. If your gender field contains the value `true`, you can't tell if that means male or female

    | :x: gender              | :white_check_mark: gender |
    | :---------------- | :----------------- |
    | false             | "MALE"              |
    | true             | "FEMALE"           |

<a name="booleans--prefix"></a>
- [3.2](#booleans--prefix) **Prefix**: Model things as booleans that are prefixed with verbs such as "is..." or "has..." (e.g. "isDoctor", "didAnswerPhoneCall" or "hasDiabetes")

<a name="booleans--orthogonal"></a>
- [3.3](#booleans--orthogonal) **Orthogonality**: If you have several mutually exclusive boolean fields in your collection, merge them into an enum

    > Why? So that it's impossible to save invalid data in your db like a car that's green and red simultaneously

    :x:

    | isRed              | isBlue | isGreen |
    | :---------------- | :----------------- | :----------------- |
    | false             | true              |  false              |
    | true             | false         | false         |

    :white_check_mark:

    | color |
    | :---------------- |
    | "BLUE"              |
    | "RED"         |

## Dates

<a name="dates--iso-strings"></a>
- [4.1](#dates--iso-strings) **ISO strings**: Make sure you never save dates as ISO strings like `"2017-04-14T06:41:21.616Z"`

    | :x:              | :white_check_mark: |
    | :---------------- | :----------------- |
    | "2017-04-14T06:41:21.616Z"             | ISODate("2017-04-14T06:41:21.616Z")            |

    
<a name="dates--day-strings"></a>
- [4.2](#dates--day-strings) **Day strings**: Don't use Date when all you are concerned with is a timezone independent day value. Instead, use strings of the form "YYYY-MM-DD"

    > Why? It makes it very easy to check whether a day falls on a certain day and it keeps your database clean

    | :x: dateOfBirth             | :white_check_mark: dateOfBirth |
    | :---------------- | :----------------- |
    | ISODate("1989-10-03T06:41:21.616Z")             | "1989-10-03"            |

## Null and undefined

<a name="null-and-undefined--no-overloading"></a>
- [5.1](#null-and-undefined--no-overloading) **No overloading**: Don't overload the meaning of `null` and `undefined` to mean anything other than "missing value"

    > Why? It breaks expectations and makes it impossible to interpret the data by looking at it.

<a name="null-and-undefined--sensible-default"></a>
- [5.2](#null-and-undefined--sensible-default) **Default**: Don't use null or undefined when there is a sensible default value like `0`, `""` or `[]`

    > Why? It keeps your types pure

    | :x: notes             | :white_check_mark: notes |
    | :---------------- | :----------------- |
    | "Some interesting observation"             | "Some interesting observation"           |
    | null             | ""           |

    | :x: comments             | :white_check_mark: comments |
    | :---------------- | :----------------- |
    | [ "First", "Great post!" ]             | [ "First", "Great post!" ]    |
    | null             | []           |

<a name="null-and-undefined--primitive-types"></a>
- [5.3](#null-and-undefined--primitive-types) **Primitive types**: In columns that contain primitive types (booleans, numbers or strings) or Date objects, use `null` to express absent values.

    | :x:            | :white_check_mark: |
    | :---------------- | :----------------- |
    | 1             | 1   |
    | 2             | 2          |
    | *missing*             | null           |

<a name="null-and-undefined--objects-and-arrays"></a>
- [5.4](#null-and-undefined--objects-and-arrays) **Complex types**: To express the absence of a value in columns that contain objects or arrays, add another column that controls whether your array or object is present. If it isn't, it should be undefined.

    :x:

    | productType              | chapterTitles |
    | :---------------- | :----------------- |
    | 'BOOK'             | ['1 Get Started', '2 The End'] |
    | 'CAR'             | []         |
    | 'CAR'             | null         |

    :white_check_mark:

    | productType              | chapterTitles |
    | :---------------- | :----------------- |
    | 'BOOK'             | ['1 Get Started', '2 The End'] |
    | 'CAR'             | *missing*        |
    | 'CAR'             | *missing*         |

<a name="null-and-undefined--dont-mix"></a>
- [5.5](#null-and-undefined--dont-mix) **Don't mix the two**: Don't mix `null` and `undefined` in the same column

    > Why? It makes it hard to understand what the two values are supposed to represent

    | :x: height            | :white_check_mark: height |
    | :---------------- | :----------------- |
    | 178             | 178   |
    | null             | null          |
    | *missing*             | null           |

## Other types

<a name="other-types--numbers-as-strings"></a>
- [6.1](#other-types--numbers-as-strings) **Numbers as strings**: Don't save numbers as strings except when saving data like phone numbers or ID numbers for which leading zeroes are significant

    > Why? It means you don't have to cast when performing arithmetic

    | :x: height             | :white_check_mark: height |
    | :---------------- | :----------------- |
    | "178"             | 178           |
    | "182"             | 182           |

    | :x: phoneNumber             | :white_check_mark: phoneNumber |
    | :---------------- | :----------------- |
    | 20123123            |  "020123123"   |

<a name="other-types--sets"></a>
- [6.2](#other-types--sets) **Sets**: Model sets as arrays containing uppercase string constants. Use JavaScript's `Set` class where appropriate.

    > Why? You can look at sets as multi-valued enumerations

    | :x:              | :white_check_mark:  |
    | :---------------- | :----------------- |
    | ['海豚', '鸽子', '蜜蜂']             | ['DOLPHIN', 'PIGEON', 'BEE']          |
    | { DOLPHIN: true, PIGEON: false, BEE: false }             | ['DOLPHIN']           |

## Names

<a name="names--abbreviations"></a>
- [7.1](#names--abbreviations) **Abbreviations**: Don't use abbreviations except for domain specific language. When you do abbreviate, capitalize

    > Why? It makes your data harder to read

    - :x: `apTime`
    - :white_check_mark: `appointmentTime`
    - :x: `ankleBrachialPressureIndexRight`
    - :white_check_mark: `ABIRight`
    - :x: `healthInformationSystemNumber`
    - :white_check_mark: `HISNumber`


<a name="names--key-names"></a>
- [7.2](#names--key-names) **Case**: Use camelCase over snake_case for key names

    > Why? It's what we use in JavaScript which means we don't have to mix the two in our code

<a name="names--collection-names"></a>
- [7.3](#names--collection-names) **Collection names**: Collection names should be pluralized and in camelCase. Use dots when there is a relationship between collections (e.g. `user` and `user.appointments`)

## Object modelling

<a name="object-modelling--growth"></a>
- [8.1](#object-modelling--growth) **Growth**: Don't let your objects keep growing. Prune and merge properties into nested objects where appropriate (e.g. by combining "footAssessmentState", "eyeAssessmentState" and "nutritionAssessmentState" into a nested "assessmentStates" object with properties "foot", "eye" and "nutrition")

<a name="object-modelling--excessive-nesting"></a>
- [8.2](#object-modelling--excessive-nesting) **Nesting**: Don't excessively nest objects. Do consider breaking up your data if you find yourself needing deeply nested objects

## Todo

- Normalization/denormalization
- ObjectIds
- Migrations