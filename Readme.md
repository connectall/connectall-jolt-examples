# ConnectALL Jolt Examples

[Jolt](https://github.com/bazaarvoice/jolt) is a JSON transformation library from [Bazaarvoice](https://github.com/bazaarvoice) written in Java. ConnectALL uses this beautiful library to power its Universal adapters, helping the customers develop integrations to hundreds and thousands of applications by transforming the REST requests and responses to the ConnectALL format.

## Table of contents
* [Overview](#Overview)
* [Structure](#Jolt%20Structure)
* [Operations](#Jolt%20Operations)  
 * [shift](#shift)
 * [default](#default)
 * [remove](#remove)
 * [modify-overwrite-beta](#modify-overwrite-beta)
 * [modify-default-beta](#modify-default-beta)
* [Functions](#Jolt%20Functions)
  * [List](#List%20Functions)
  * [String](#String%20Functions)
  * [Type casting](#Type%20casting%20Funcations)

## Overview

ConnectALL universal adapter provides JOLT, Groovy language options for converting a JSON document to and from its API structures. Groovy language provides a more sophisticated option for developers, while JOLT is aimed at users who may not having the programming background. The easy to use syntax of JOLT will help anyone with a basic knowledge of REST api and JSON structures to create new integrations using ConnectALL in a matter of hours.

## JOLT Structure

JOLT specification contains an Array of Operations that are used to transform the input. Each operation is a JSON object with 2 attributes `operation` contains JOLT operation to be performed, and `spec` contains a JSON object with instructions to use for transforming the content.  

Example:
```
[
  {
    "operation": "shift",
    "spec": {

    }
  },
  {
    "operation": "default",
    "spec": {

    }
  }
]
```

## JOLT Operations

JOLT operation library is continually growing for handling a variety of scenarios, defined under are a list of operations that are frequently used in writing the specifications by ConnectALL Universal adapter.

### shift
Shift operation specifies where "data" from the input JSON should be placed in the output JSON. This is the most widely used operation in the JOLT specification and is the basis for all transformations.

> Note: Only one Shift operation is allowed in the JOLT specification


_Examples defined under cover the basic fundementals, do visit [JOLT demo app](https://jolt-demo.appspot.com/#inception) for many uses of the Shift operation_

Shift operations uses several wildcard characters, the examples below cover a couple of them.

|Wildcard|Meaning|
|---|---|
|`*`| predominantly used in LHS, means for all attributes in a JSON object or for all elements in a JSON array|
|`&`| predominantly used in RHS, means use the same value as the attribute key captured by `*`|
|`@`/`@(3,attr1)`| can be used in LHS or RHS, means replace with the value of the attribute in the Input. When used in complex form the operation will go up 3 levels and capture the value of the `attr1` attribute|
|`#`| used in the LHS keys, to set a default value |
|`$`/`$(0)`| used in the LHS to specify copy the attribute name as value in Output. Index in the paranthesis will aid in navigating to upper level|



**Example 1**

In this example we are promoting the fields under "fields" attribute to the root object

| Type | Spec |
| ----- | --- |
|Input | ```{ "fields": {"Version": "0.0.4","ProjectId": "Projects-2","ChannelId":"Channels-2","ReleaseNotes": "0.0.4 release for test","IgnoreChannelRules": true  }}``` |
|Spec|```[{"operation":"shift","spec":{"fields":{"Version":"Version","ProjectId":"ProjectId","ChannelId":"ChannelId","ReleaseNotes":"ReleaseNotes"}}}]```|
|Output| ```{"Version":"0.0.4","ProjectId":"Projects-2","ChannelId":"Channels-2","ReleaseNotes":"0.0.4 release for test"}```|

**Example 2**

In this example we will try obtaining the same result, using the awesome regular expressions of the `Shift` operation. In this example wildcard character `*` is used in LHS meant as `for any field under the "fields" JSON object`, and the wildcard character `&` used in RHS meant as `use the same attribute name`.  

| Type | Spec |
| ----- | --- |
|Input | ```{ "fields": {"Version": "0.0.4","ProjectId": "Projects-2","ChannelId":"Channels-2","ReleaseNotes": "0.0.4 release for test","IgnoreChannelRules": true  }}``` |
|Spec|```[{"operation":"shift","spec":{"fields":{"*":"&"}}}]```|
|Output| ```{"Version":"0.0.4","ProjectId":"Projects-2","ChannelId":"Channels-2","ReleaseNotes":"0.0.4 release for test"}```|

**Example 3**

In this example we will try extend the use of wildcard characters from the above example to convert the name of `Version` as `Affect-Version` and nesting the rest of attributes under field under `Details`

| Type | Spec |
| ----- | --- |
|Input | ```{ "fields": {"Version": "0.0.4","ProjectId": "Projects-2","ChannelId":"Channels-2","ReleaseNotes": "0.0.4 release for test","IgnoreChannelRules": true  }}``` |
|Spec|```[{"operation":"shift","spec":{"fields":{"Version":"Affect-&","*":"Details.&"}}}]```|
|Output| ```{"Affect-Version":"0.0.4","Details":{"ProjectId":"Projects-2","ChannelId":"Channels-2","ReleaseNotes":"0.0.4 release for test","IgnoreChannelRules":true}}```|


### default

When you want to add additional data that is not in INPUT, you can use this operation.

> Note: The input for this operation is the output of the previous operation so the specification should use the transformed keys from the preceding `shift` operation

**Example 1**

In this example we will add `"type":"bug"` in addition to moving the attributes under `fields` to `data` under the newly created `request` object.

| Type | Spec |
| ----- | --- |
|Input | ```{ "fields": {"Version": "0.0.4","ProjectId": "Projects-2","ChannelId":"Channels-2","ReleaseNotes": "0.0.4 release for test","IgnoreChannelRules": true  }}``` |
|Spec|```[{"operation":"shift","spec":{"fields":"request.data"}},{"operation":"default","spec":{"request":{"type":"bug"}}}]```|
|Output| ```{"request":{"data":{"Version":"0.0.4","ProjectId":"Projects-2","ChannelId":"Channels-2","ReleaseNotes":"0.0.4 release for test","IgnoreChannelRules":true},"type":"bug"}}```|

### remove

As the name suggests `remove` operation will remove any of the attributes you do not need in the OUTPUT

**Example 1**

In this example we will remove `_links` object from the output.

| Type | Spec |
| ----- | --- |
|Input | ```{"fields":{"Version":"0.0.4","ProjectId":"Projects-2","ChannelId":"Channels-2","ReleaseNotes":"0.0.4 release for test","IgnoreChannelRules":true,"_links":{"_self":"/bug/1","_parent":"/epic/12"}}}``` |
|Spec|```[{"operation":"shift","spec":{"fields":"request.data"}},{"operation":"remove","spec":{"request":{"data":{"_links":""}}}}]```|
|Output| ```{"request":{"data":{"Version":"0.0.4","ProjectId":"Projects-2","ChannelId":"Channels-2","ReleaseNotes":"0.0.4 release for test","IgnoreChannelRules":true}}}```|

## modify-overwrite-beta

This operation uses several of the [JOLT functions](#JOLT%20functions) defined under to transform the values in the output. This operation is mainly used for altering the value in the output.

> Note: You can chain multiple modify-overwrite-beta operations in the JOLT specification

**Example 1**

In this example we will copy the array of "data" into "result" and format the output value to have only the firstElement of the Array

| Type | Spec |
| ----- | --- |
|Input | ```{"data":["1","2","3"]}``` |
|Spec|```[{"operation":"shift","spec":{"data":"result"}},{"operation":"modify-overwrite-beta","spec":{"result":"=firstElement(@(1,result))"}}]```|
|Output| ```{"result":"1"}```|

**Example 1**

In this we will use the the `lastElement` instead of `fistElement` from the above example and also convert that into an Integer value

| Type | Spec |
| ----- | --- |
|Input | ```{"data":["1","2","3"]}``` |
|Spec|```[{"operation":"shift","spec":{"data":"result"}},{"operation":"modify-overwrite-beta","spec":{"result":"=lastElement(@(1,result))"}},{"operation":"modify-overwrite-beta","spec":{"result":"=toInteger"}}]```|
|Output| ```{"result":3}```|

## modify-default-beta

This operation is used for checking if an attribute exists or not and when not present will use this transformation to add it to the OUTPUT.

> Refer Gitlab get-modified-record-response-jolt.json for the behavior

## JOLT functions

JOLT specifications provides a wide library of functions for the most common operations to be performed in the transformations. These funtions are classifed as `List`, `String` and `Type casting` functions.

These functions are typically used in the RHS of of `modify-overwrite-beta` or `modify-default-beta` operations to transform the values in the OUTPUT

### List functions

List functions are used on top of Array objects during the transformations.

#### size

`=size(@(1,element))` will give the size of the element array, which is one level above the current attribute

#### sort

`=sort(@(1,element))` will sort the contents of element array, which is one level above the current attribute

#### firstElement

`=firstElement(@(1,element))` will return the firstElement of the element array, which is one level above the current attribute

#### lastElement

`=lastElement(@(1,element))` will return the lastElement of the element array, which is one level above the current attribute

#### elementAt

`=elementAt(2,@(1,element))` will return the 2nd element of the element array, which is one level above the current attribute

### String functions

String functions are used for typical String manipulations

#### concat

// TODO

#### replace

// TODO

#### trim

// TODO

### Type casting functions

#### toString

`=toString` will convert the value as String

#### toInteger

`=toInteger` will convert the value as Integer

#### toBoolean

`=toBoolean` will convert the value as Boolean `true`/`false`
