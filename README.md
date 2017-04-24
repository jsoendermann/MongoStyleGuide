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

<a name="general"></a><a name="1.1"></a>

- [1.1](#general--mix) **Mixed types**: Don't mix values of different types in one column.

| <span style="color: red">Bad column</style> | <span style="color: green">Good column</span> |
| ---------- | ----------- |
| 1          | 1           |
| "2"        | 2           |
| { }        | 0           |

- Don't mix types. **Do be conservative in what you write liberal in what you accept**
- Don't use the empty string or 0 for their falsiness. **Do make non-casting, explicit comparisons**

## Enums

- Don't use numbers, booleans or **any other types** to model **enumerations**. Do model them as uppercase string constants, e.g. "WAITING", "IN_PROGRESS" and "COMPLETED". 
- Don't use null, undefined **or any other value or type** in your enum columns. Do give every state an **explicit enum value**, including **initial, undecided or unknown**

## Booleans

- Don't model things as booleans that have no natural correspondence to true/false, even if they only have two possible states (e.g. a gender field that only contains "MALE" and "FEMALE". Also keep in mind that fields might have to be extended later to include additional values, a boolean field "hasArrived" could not be extended later to include the possibility of cancelled appointments).
- Do model things as booleans that are prefixed with verbs such as "is..." or "has..." (e.g. "isDoctor", "didAnswerPhoneCall" or "hasDiabetes")
- **Don't include multiple mutually exclusive boolean keys in your collection. Do merge them into an enum**

## Dates

- Don't save dates as serialized ISO strings like "2017-04-14T06:41:21.616Z". Do save them as Date objects
- **Don't use Date when all you are concerned with is a timezone independent day value**. Do use strings of the form "YYYY-MM-DD" in those cases

## Null and undefined

- Don't use null or undefined when there is a sensible default (e.g. when you have a "notes" field, the initial value should be the empty string, not null). Do prefer null over undefined or missing values

## Other types

- Don't save numbers as strings. Do make an exception for data like phone numbers or shenfenzheng ids, for which leading zeroes are significant 
- Don't mix ObjectIds with any other types like string. Do use ObjectIds for _id when there is no obvious unique field suitable to be used as _id

## Names

- Don't use snake_case. Do use camelCase
- Don't use abbreviations except for widely used, domain specific language. Do learn the lingo of your field

## Object modelling

- **Don't let your objects keep growing. Do prune them and make judicious use of nested objects to group properties that belong together** (e.g. combining "footAssessmentState", "eyeAssessmentState" and "nutritionAssessmentState" into an "assessmentStates" nested object with properties "foot", "eye" and "nutrition"
- **Don't excessively nest objects. Do consider breaking up your data if you find yourself needing deeply nested objects**