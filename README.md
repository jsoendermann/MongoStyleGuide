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
  1. [Todo](#todo)


## General

- [1.1](#general--mix) **Mixed types**: Don't mix values of different types in one column. Restrict yourself to one type per column

    > Why? This way, you don't have to typecheck or cast when you consume the data.

    | :x:           |:white_check_mark: |
    | :------------ | :---------------- |
    | 1             | 1                 |
    | "2"           | 2                 |
    | { value: 3 }  | 3                 |

- [1.2](#general--object-schema) **Object columns**: If you have a column that contains objects, make sure all objects share the same format

    | :x:       | :white_check_mark:     |
    | :---------------- | :------------- |
    | { value: 42 }    | { value: 42 }   |
    | { value: "foo" } | { value: 43 }   |
    | { }              | { value: null } |

- [1.3](#general--falsiness) **Falsiness**: Don't use `""` or `0` for their falsiness

    > Why? It's very easy to write buggy code otherwise which is also why you should not coerce to booleans in your if statements.

    | :x:       | :white_check_mark:   |
    | :---------------- | :------------- |
    | "foo"            | "foo"         |
    | "bar"            | "bar"         |
    | ""               | null          |


## Enumerations

An enumeration is a type that allows a limited number of values, e.g. a userType field that can contain the values `"DOCTOR"`, `"NURSE"`, `"PATIENT"` and `"ADMINISTRATOR"`

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

- [2.2](#enumerations--explicit) **Explicit states**: Don't use `null`, `undefined`, or any value except upper case string constants in your enums. This includes initial, undecided or unknown states

    > Why? This makes the meaning explicit. If you have an assessmentState field that's set to `null`, you can't tell whether that means "no assessment necessary", "not applicable", "patient didn't show up", "undecided" or any other possible state

    | :x: assessmentState | :white_check_mark: assessmentState |
    | :------------------ | :--------------------------------- |
    | null                | "NOT_REQUIRED"                     |
    | *missing*           | "NOT_APPLICABLE"                   |


## Booleans

- [3.1](#booleans--booleans-and-enums) **Booleans and enums**: Don't model things as booleans that have no natural correspondence to true/false, even if they only have two possible states. Use enums instead

    > Why?
    > 1. Booleans can not be extended beyond two possible values but the field might have to change later to include additional values, e.g. a boolean field "hasArrived" could not be extended later to include the possibility of cancelled appointments
    > 1. It makes meaning explicit. If your gender field contains the value `true`, you can't tell if that means male or female

    | :x: gender              | :white_check_mark: gender |
    | :---------------- | :----------------- |
    | false             | "MALE"              |
    | true             | "FEMALE"           |

- [3.2](#booleans--prefix) **Prefix**: Model things as booleans that are prefixed with verbs such as "is..." or "has..." (e.g. "isDoctor", "didAnswerPhoneCall" or "hasDiabetes")

- [3.2](#booleans--orthogonal) **Orthogonality**: If you have several mutually exclusive boolean fields in your collection, merge them into an enum

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

- [3.1](#dates--iso-strings) **ISO strings**: Make sure you never save dates as ISO strings like `"2017-04-14T06:41:21.616Z"`

    | :x:              | :white_check_mark: |
    | :---------------- | :----------------- |
    | "2017-04-14T06:41:21.616Z"             | ISODate("2017-04-14T06:41:21.616Z")            |

    
- [3.2](#dates--day-strings) **Day strings**: Don't use Date when all you are concerned with is a timezone independent day value. Instead, use strings of the form "YYYY-MM-DD"

    | :x: dateOfBirth             | :white_check_mark: dateOfBirth |
    | :---------------- | :----------------- |
    | ISODate("1989-10-03T06:41:21.616Z")             | '1989-10-03'            |

## Null and undefined

- [4.1](#null-and-undefined--no-overloading) **No overloading**: Don't overload the meaning of `null` and `undefined` to mean anything other than "missing value"

    > Why? It breaks expectations and makes it impossible to interpret the data by looking at it.

- [4.2](#null-and-undefined--sensible-default) **Default**: Don't use null or undefined when there is a sensible default value like `0`, `""` or `[]`

    > Why? It keeps your types pure

    | :x: notes             | :white_check_mark: notes |
    | :---------------- | :----------------- |
    | "Some interesting observation"             | "Some interesting observation"           |
    | null             | ""           |

    | :x: comments             | :white_check_mark: comments |
    | :---------------- | :----------------- |
    | [ "First", "Great post!" ]             | [ "First", "Great post!" ]    |
    | null             | []           |

- [4.3](#null-and-undefined--dont-mix) **Don't mix the two**: Don't mix `null` and `undefined` in the same column

    > Why? It makes it hard to understand what the two values mean

## Other types

- [5.1](#other-types--numbers-as-strings) **Numbers as strings**: Don't save numbers as strings except when saving data like phone numbers or ID numbers for which leading zeroes are significant

    > Why? It means you don't have to cast when performing arithmetic

    | :x: height             | :white_check_mark: height |
    | :---------------- | :----------------- |
    | "178"             | 178           |
    | "182"             | 182           |

    | :x: phoneNumber             | :white_check_mark: phoneNumber |
    | :---------------- | :----------------- |
    | 20123123            |  "020123123"   |

## Names

- [6.1](#names--abbreviations) **Abbreviations**: Don't use abbreviations except for domain specific language. When you do abbreviate, capitalize

    > Why? It makes your data harder to read

    - :x: `apTime`
    - :white_check_mark: `appointmentTime`
    - :x: `ankleBrachialPressureIndexRight`
    - :white_check_mark: `ABIRight`
    - :x: `healthInformationSystemNumber`
    - :white_check_mark: `HISNumber`


- [6.2](#names--case) **Case**: Use camelCase over snake_case

    > Why? It's what we use in JavaScript

## Object modelling

- [7.1](#object-modelling--growth) **Growth**: Don't let your objects keep growing. Prune and merge properties into nested objects where appropriate (e.g. by combining "footAssessmentState", "eyeAssessmentState" and "nutritionAssessmentState" into a nested "assessmentStates" object with properties "foot", "eye" and "nutrition")

- [7.1](#object-modelling--excessive-nesting) **Nesting**: Don't excessively nest objects. Do consider breaking up your data if you find yourself needing deeply nested objects

## Todo

Prefer null over undefined/missing?
Normalization/denormalization
ObjectIds