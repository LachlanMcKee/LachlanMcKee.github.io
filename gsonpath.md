---
layout: post
title: Gson Path
date: August 27th 2016
author: Lachlan McKee
permalink: /gsonpath/
---

[Github](https://github.com/LachlanMcKee/gsonpath) \| [Download](#download) \| [License](#license) \| [Javadoc](javadoc)

# Introduction
Reading and writing JSON is common place when working with REST APIs. One of the most popular Java libraries used for this purpose is [Gson](https://github.com/google/gson), which is a library written and maintained by Google.

Gson Path is a Java annotation processor which generates Gson Type Adapters at compile time to avoid costly reflection at runtime. It also hosts a multitude of other features which will be described within this document.

# Usage

Getting started using Gson Path is straight forward. If you have not already done so, visit the [download](#download) section and add the dependencies to your build.

As Gson Path is an annotation processor, the majority of the heavy lifting is done during compilation. Every class annotated with the `@AutoGsonAdapter` annotation will generate a Type Adapter. The annotation exposes a rich set of configurations, and there are many examples their usage discussed in the [features](#features) section.

To add these generated classes into your project, the Gson Path Type Adapter Factory must be added to the Gson instance, as shown below:

```java
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.registerTypeAdapterFactory(GsonPath.createTypeAdapterFactory());
Gson gson = gsonBuilder.create();
```

This factory maps all of the generated Type Adapters to their respective POJO within Gson without a developer needing to reference the classes directly.

# Features

Aside from generating typical Gson Type Adapters at compile time, there are many other features available within the library which expand the Type Adapters functionality to further reduce boilerplate code.

1. [Basic Json Path support](#basic-json-path-support)
2. [Client side validation](#client-side-validation)
3. [Generate POJOs using interfaces](#generate-pojos-using-interfaces)
4. [Json Path Substitutions](#json-path-substitutions)

## Basic Json Path support

Gson Type Adapters handle marshalling and unmarshalling JSON content into Java POJOs out of the box. Unfortunately the library requires a one-to-one relationship between a POJO and a JSON object. This means that the more complex a JSON document is, the more POJOs a developer must create.

Gson Path adds basic [Json Path](http://goessner.net/articles/JsonPath/) support to solve this problem. Json Path is a query language similar to [XPath](https://en.wikipedia.org/wiki/XPath), which allows you to reference elements within a document through shorthand notation (e.g. `parent.child`).

This shorthand notation is extremely useful as it provides access to a particular subsection of a JSON document. Gson Path takes advantage of this Json Path 'dot notation' query to allow developers to reference specific fields within a JSON tree from the standard Gson `SerializedName` annotation (this works for Type Adapter reading and writing).

For example, given the following JSON:

```json
{
    "person": {
        "names": {
            "first": "Lachlan",
            "last": "McKee"
        }
    }
}
```

We can deserialize this JSON using a single Java class several ways. The following classes demonstrate the annotations required to create a Type Adapter which can read the nested content.

### POJO Example 1

The Json Path dot notation is added to Gson `@SerializedName` annotation value.

```java
public class PersonModel {
    @SerializedName("person.names.first")
    String firstName;

    @SerializedName("person.names.last")
    String lastName;
}
```

### POJO Example 2
The `rootField` property of the `@AutoGsonAdapter` annotation is used to avoid repetition.

```java
@AutoGsonAdapter(rootField = "person.names")
public class PersonModel {
    @SerializedName("first")
    String firstName;

    @SerializedName("last")
    String lastName;
}
```

### POJO Example 3
When the name of the field matches the name of the JSON key, the `@SerializedName` annotation can be omitted.

```java
@AutoGsonAdapter(rootField = "person.names")
public class PersonModel {
    String first;
    String last;
}
```

### Generated Type Adapter
All of the previous examples will generate the following Gson TypeAdapter:

```java
public final class PersonModel_GsonTypeAdapter extends TypeAdapter<PersonModel> {
    private final Gson mGson;

    public PersonModel_GsonTypeAdapter(Gson gson) {
        this.mGson = gson;
    }

    @Override
    public PersonModel read(JsonReader in) throws IOException {
        // Ensure the object is not null.
        if (!isValidValue(in)) {
            return null;
        }
        
        PersonModel result = new PersonModel();
        int jsonFieldCounter0 = 0;
        in.beginObject();
        
        while (in.hasNext()) {
            if (jsonFieldCounter0 == 1) {
                in.skipValue();
                continue;
            }
            
            switch (in.nextName()) {
                case "person":
                    jsonFieldCounter0++;
                    
                    // Ensure the object is not null.
                    if (!isValidValue(in)) {
                        break;
                    }
                    
                    int jsonFieldCounter1 = 0;
                    in.beginObject();
                    
                    while (in.hasNext()) {
                        if (jsonFieldCounter1 == 1) {
                            in.skipValue();
                            continue;
                        }
                        
                        switch (in.nextName()) {
                            case "names":
                                jsonFieldCounter1++;
                                
                                // Ensure the object is not null.
                                if (!isValidValue(in)) {
                                    break;
                                }
                                
                                int jsonFieldCounter2 = 0;
                                in.beginObject();
                                
                                while (in.hasNext()) {
                                    if (jsonFieldCounter2 == 2) {
                                        in.skipValue();
                                        continue;
                                    }
                                    
                                    switch (in.nextName()) {
                                        case "first":
                                            jsonFieldCounter2++;
                                            
                                            String safeValue0 = getStringSafely(in);
                                            if (safeValue0 != null) {
                                                result.firstName = safeValue0;
                                            }
                                            break;
                                            
                                        case "last":
                                            jsonFieldCounter2++;
                                            
                                            String safeValue1 = getStringSafely(in);
                                            if (safeValue1 != null) {
                                                result.lastName = safeValue1;
                                            }
                                            break;
                                            
                                        default:
                                            in.skipValue();
                                            break;
                                    }
                                }
                                
                                in.endObject();
                                break;
                                
                            default:
                                in.skipValue();
                                break;
                        }
                    }
                    
                    in.endObject();
                    break;
                    
                default:
                    in.skipValue();
                    break;
            }
        }
        
        in.endObject();
        return result;
    }

    @Override
    public void write(JsonWriter out, PersonModel value) throws IOException {
        if (value == null) {
            out.nullValue();
            return;
        }
    
        // Begin
        out.beginObject();
    
        // Begin person
        out.name("person");
        out.beginObject();
    
        // Begin person.names
        out.name("names");
        out.beginObject();
        String obj0 = value.first;
        if (obj0 != null) {
            out.name("first");
            out.value(obj0);
        }
    
        String obj1 = value.last;
        if (obj1 != null) {
            out.name("last");
            out.value(obj1);
        }
    
        // End person.names
        out.endObject();
        // End person
        out.endObject();
        // End 
        out.endObject();
    }
}
```

[Back to top](#introduction)

## Client side validation

Developers are able to optionally turn on a degree of client side validation (in the form of required fields) by adding `@Nullable` and `@NonNull` annotations to their POJOs.

The `@AutoGsonAdapter` annotation has a property called `fieldValidationType` which specifies if and how the generated Type Adapter will validate the JSON. There are three different possible validation types:

1. `NO_VALIDATION` - No fields are validated, and the Gson parser should never raise exceptions for missing content.
2. `VALIDATE_EXPLICIT_NON_NULL` - Any Objects marked with `@NonNull` (or similar annotation), or primitives (since primitives cannot be null) will raise an exception if the value does not exist within the JSON.
3. `VALIDATE_ALL_EXCEPT_NULLABLE` - All fields will be treated as `@NonNull` (even when they are not annotated as such), and will raise an exception when if the value is not found. The only exceptions to this rule is when a field is annotation with `@Nullable` (except for primitives, since they cannot be null).

### Validate explicit non null - POJO

```java
@AutoGsonAdapter(fieldValidationType = GsonFieldValidationType.VALIDATE_EXPLICIT_NON_NULL)
public class TestValidateExplicitNonNull {
    @NonNull
    public Integer mandatory1;
    public int mandatory2;

    public Integer optional1;
}
```

### Validate explicit non null - Generated Type Adapter

```java
public final class TestValidateExplicitNonNull_GsonTypeAdapter extends TypeAdapter<TestValidateExplicitNonNull> {
    private static final int MANDATORY_INDEX_MANDATORY1 = 0;
    private static final int MANDATORY_INDEX_MANDATORY2 = 1;
    private static final int MANDATORY_FIELDS_SIZE = 2;

    private final Gson mGson;

    public TestValidateExplicitNonNull_GsonTypeAdapter(Gson gson) {
        this.mGson = gson;
    }

    @Override
    public TestValidateExplicitNonNull read(JsonReader in) throws IOException {
        // Ensure the object is not null.
        if (!isValidValue(in)) {
            return null;
        }
        
        TestValidateExplicitNonNull result = new TestValidateExplicitNonNull();
        boolean[] mandatoryFieldsCheckList = new boolean[MANDATORY_FIELDS_SIZE];

        int jsonFieldCounter0 = 0;
        in.beginObject();

        while (in.hasNext()) {
            if (jsonFieldCounter0 == 4) {
                in.skipValue();
                continue;
            }

            switch (in.nextName()) {
                case "mandatory1":
                    jsonFieldCounter0++;

                    Integer value_mandatory1 = getIntegerSafely(in);
                    if (value_mandatory1 != null) {
                        result.mandatory1 = value_mandatory1;
                        mandatoryFieldsCheckList[MANDATORY_INDEX_MANDATORY1] = true;
                    } else {
                        throw new gsonpath.JsonFieldMissingException("Mandatory JSON element 'mandatory1' was null for class 'adapter.auto.field_policy.validate_explicit_non_null.TestValidateExplicitNonNull'");
                    }
                    break;

                case "mandatory2":
                    jsonFieldCounter0++;

                    Integer value_mandatory2 = getIntegerSafely(in);
                    if (value_mandatory2 != null) {
                        result.mandatory2 = value_mandatory2;
                        mandatoryFieldsCheckList[MANDATORY_INDEX_mandatory2] = true;
                    } else {
                        throw new gsonpath.JsonFieldMissingException("Mandatory JSON element 'mandatory2' was null for class 'adapter.auto.field_policy.validate_explicit_non_null.TestValidateExplicitNonNull'");
                    }
                    break;

                case "optional1":
                    jsonFieldCounter0++;

                    Integer value_optional1 = getIntegerSafely(in);
                    if (value_optional1 != null) {
                        result.optional1 = value_optional1;
                    }
                    break;

                default:
                    in.skipValue();
                    break;
            }
        }

        in.endObject();

        // Mandatory object validation
        for (int mandatoryFieldIndex = 0; mandatoryFieldIndex < MANDATORY_FIELDS_SIZE; mandatoryFieldIndex++) {

            // Check if a mandatory value is missing.
            if (!mandatoryFieldsCheckList[mandatoryFieldIndex]) {

                // Find the field name of the missing json value.
                String fieldName = null;
                switch (mandatoryFieldIndex) {
                    case MANDATORY_INDEX_MANDATORY1:
                        fieldName = "mandatory1";
                        break;

                    case MANDATORY_INDEX_MANDATORY2:
                        fieldName = "mandatory2";
                        break;
                }
                throw new gsonpath.JsonFieldMissingException("Mandatory JSON element '" + fieldName + "' was not found for class 'adapter.auto.field_policy.validate_explicit_non_null.TestValidateExplicitNonNull'");
            }
        }

        return result;
    }

    @Override
    public void write(JsonWriter out, TestValidateExplicitNonNull value) throws IOException {
        // Omitted for this example to reduce page length.
    }
}
```

### Validate all except nullable - POJO

```java
@AutoGsonAdapter(fieldValidationType = GsonFieldValidationType.VALIDATE_ALL_EXCEPT_NULLABLE)
public class TestValidateAllExceptNullable {
    public Integer mandatory1;
    public Integer mandatory2;

    @Nullable
    public Integer optional1;
}
```

### Validate all except nullable - Generated Type Adapter

```java
public final class TestValidateAllExceptNullable_GsonTypeAdapter extends TypeAdapter<TestValidateAllExceptNullable> {
    private static final int MANDATORY_INDEX_MANDATORY1 = 0;
    private static final int MANDATORY_FIELDS_SIZE = 1;

    private final Gson mGson;

    public TestValidateAllExceptNullable_GsonTypeAdapter(Gson gson) {
        this.mGson = gson;
    }

    @Override
    public TestValidateAllExceptNullable read(JsonReader in) throws IOException {
        // Ensure the object is not null.
        if (!isValidValue(in)) {
            return null;
        }
		
        TestValidateAllExceptNullable result = new TestValidateAllExceptNullable();
        boolean[] mandatoryFieldsCheckList = new boolean[MANDATORY_FIELDS_SIZE];

        int jsonFieldCounter0 = 0;
        in.beginObject();

        while (in.hasNext()) {
            if (jsonFieldCounter0 == 3) {
                in.skipValue();
                continue;
            }

            switch (in.nextName()) {
                case "mandatory1":
                    jsonFieldCounter0++;

                    Integer value_mandatory1 = getIntegerSafely(in);
                    if (value_mandatory1 != null) {
                        result.mandatory1 = value_mandatory1;
                        mandatoryFieldsCheckList[MANDATORY_INDEX_MANDATORY1] = true;
                    } else {
                        throw new gsonpath.JsonFieldMissingException("Mandatory JSON element 'mandatory1' was null for class 'adapter.auto.field_policy.validate_all_except_nullable.TestValidateAllExceptNullable'");
                    }
                    break;

                case "optional1":
                    jsonFieldCounter0++;

                    Integer value_optional1 = getIntegerSafely(in);
                    if (value_optional1 != null) {
                        result.optional1 = value_optional1;
                    }
                    break;

                default:
                    in.skipValue();
                    break;
            }
        }

        in.endObject();

        // Mandatory object validation
        for (int mandatoryFieldIndex = 0; mandatoryFieldIndex < MANDATORY_FIELDS_SIZE; mandatoryFieldIndex++) {

            // Check if a mandatory value is missing.
            if (!mandatoryFieldsCheckList[mandatoryFieldIndex]) {

                // Find the field name of the missing json value.
                String fieldName = null;
                switch (mandatoryFieldIndex) {
                    case MANDATORY_INDEX_MANDATORY1:
                        fieldName = "mandatory1";
                        break;
                }
                throw new gsonpath.JsonFieldMissingException("Mandatory JSON element '" + fieldName + "' was not found for class 'adapter.auto.field_policy.validate_all_except_nullable.TestValidateAllExceptNullable'");
            }
        }

        return result;
    }

    @Override
    public void write(JsonWriter out, TestValidateAllExceptNullable value) throws IOException {
        // Omitted for this example to reduce page length.
    }
}
```

[Back to top](#introduction)

## Generate POJOs using interfaces

Gson Path supports annotating interfaces with the `@AutoGsonAdapter` annotation.

At compile time an immutable POJO is generated, and is used as the concrete implementation of the interface when the Type Adapter deserializes the JSON.

### Differences to standard POJO

There are a few key differences between standard classes using the `AutoGsonAdapter` annotation, and an interface. These are as follows:

* The `GsonFieldValidationType` property (mentioned in the [Client side validation](#client-side-validation) section) can only ever be `VALIDATE_EXPLICIT_NON_NULL` or `VALIDATE_ALL_EXCEPT_NULLABLE`. 
   * This API design decision was made since you are unable to set a default value for primitive return types. 
   * If the `GsonFieldValidationType` is not one of the previously mentioned values, it will be forced to `VALIDATE_EXPLICIT_NON_NULL`.
   
* When designing your interface:
   * Be sure to always specify a return type for methods.
   * Do not add any parameters to your method as this method should acts as a 'getter'.
   * The library creates variable names by stripping the characters until it reaches the first uppercase letter. Be sure to begin the name of the method with a 'get' or 'is' or whatever POJO standard you wish to ensure you don't encounter name collisions. 
   * Be aware that although the generated object POJO itself is immutable, the objects contained within the class may not be. 
 
### Sample

The following interface:

```java
@AutoGsonAdapter
public interface InterfaceExample {
    Integer getObjectExample();
    int getPrimitiveExample();
}
```

Will generate the following POJO:

```java
public final class InterfaceExample_GsonPathModel implements InterfaceExample {
    private final Integer objectExample;
    private final int primitiveExample;
    
    public InterfaceExample_GsonPathModel(Integer objectExample, int primitiveExample) {
        this.objectExample = objectExample;
        this.primitiveExample = primitiveExample;
    }
    
    @Override
    public Integer getObjectExample() {
        return objectExample;
    }
    
    @Override
    public int getPrimitiveExample() {
        return primitiveExample;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        
        InterfaceExample_GsonPathModel that = (InterfaceExample_GsonPathModel) o;
        
        if ((objectExample == null || !objectExample.equals(that.objectExample))) return false;
        if (primitiveExample != that.primitiveExample) return false;
        
        return true;
    }
    
    @Override
    public int hashCode() {
        int result = objectExample != null ? objectExample.hashCode() : 0;
        result = 31 * result + (primitiveExample);
        return result;
    }
}
```

Which will in turn generate the following `TypeAdapter`

```java
public final class InterfaceExample_GsonTypeAdapter extends TypeAdapter<InterfaceExample> {
    private static final int MANDATORY_INDEX_PRIMITIVEEXAMPLE = 0;
    private static final int MANDATORY_FIELDS_SIZE = 1;
    
    private final Gson mGson;
    
    public InterfaceExample_GsonTypeAdapter(Gson gson) {
        this.mGson = gson;
    }
    
    @Override
    public InterfaceExample read(JsonReader in) throws IOException {
        // Ensure the object is not null.
        if (!isValidValue(in)) {
            return null;
        }
        
        java.lang.Integer value_objectExample = null;
        int value_primitiveExample = 0;
        
        boolean[] mandatoryFieldsCheckList = new boolean[MANDATORY_FIELDS_SIZE];
        
        int jsonFieldCounter0 = 0;
        in.beginObject();
        
        while (in.hasNext()) {
            if (jsonFieldCounter0 == 2) {
                in.skipValue();
                continue;
            }
            
            switch (in.nextName()) {
                case "objectExample":
                    jsonFieldCounter0++;
                    
                    value_objectExample = getIntegerSafely(in);
                    break;
                    
                case "primitiveExample":
                    jsonFieldCounter0++;
                    
                    Integer value_primitiveExample_safe = getIntegerSafely(in);
                    if (value_primitiveExample_safe != null) {
                        value_primitiveExample = value_primitiveExample_safe;
                        mandatoryFieldsCheckList[MANDATORY_INDEX_PRIMITIVEEXAMPLE] = true;
                        
                    } else {
                        throw new gsonpath.JsonFieldMissingException("Mandatory JSON element 'primitiveExample' was null for class 'adapter.auto.interface_example.primitive.InterfaceExample_GsonPathModel'");
                    }
                    break;
                
                default:
                    in.skipValue();
                    break;
            }
        }
        
        in.endObject();
        
        // Mandatory object validation
        for (int mandatoryFieldIndex = 0; mandatoryFieldIndex < MANDATORY_FIELDS_SIZE; mandatoryFieldIndex++) {
            
            // Check if a mandatory value is missing.
            if (!mandatoryFieldsCheckList[mandatoryFieldIndex]) {
                
                // Find the field name of the missing json value.
                String fieldName = null;
                switch (mandatoryFieldIndex) {
                    case MANDATORY_INDEX_PRIMITIVEEXAMPLE:
                        fieldName = "primitiveExample";
                        break;
                        
                }
                throw new gsonpath.JsonFieldMissingException("Mandatory JSON element '" + fieldName + "' was not found for class 'adapter.auto.interface_example.primitive.InterfaceExample_GsonPathModel'");
            }
        }
        
        return new InterfaceExample_GsonPathModel(
                value_objectExample,
                value_primitiveExample
        );
    }
    
    @Override
    public void write(JsonWriter out, InterfaceExample value) throws IOException {
    }
}
```

[Back to top](#introduction)

## Json Path Substitutions

Json Path Substitutions build on top of the expressive Json Path 'dot notation' by adding placeholder substitution within the `@SerializedName` annotation field name.

This is useful in situations where an API returns nearly identical JSON, however the keys may differ. The aim of this feature is to reuse as much code as possible, and avoid creating two completely different models, or using inheritance in conjunction with abstract methods.

For example, given the following two similar json responses:

```json
{
  "staff": {
    "name": "John Smith",
    "age": 50
  },
  "address": "123 Fake St"
}
```
```json
{
  "customer": {
    "name": "Jack Torrance",
    "age": 29
  },
  "address": "Overlook Hotel"
}
```

The following base model can be created to contain all of the fields:

```java
public class ContactDetailsBase {
    @SerializedName("{PERSON_DETAILS_SUB}.")
    public String name;
    
    @SerializedName("{PERSON_DETAILS_SUB}.")
    public int age;
    
    public String address;
}
```

And the two implementing classes would be as follows:

```java
@AutoGsonAdapter(substitutions = {
        @PathSubstitution(original = "PERSON_DETAILS_SUB", replacement = "staff")
})
public class StaffContactDetails extends ContactDetailsBase {
}
```
```java
@AutoGsonAdapter(substitutions = {
        @PathSubstitution(original = "PERSON_DETAILS_SUB", replacement = "customer")
})
public class CustomerContactDetails extends ContactDetailsBase {
}
```

When the Type Adapter is generated it will inspect every field annotated with `@SerializedName` and execute a string replacement which searches for the 'original' value wrapped with `{}` and replaces it with the 'replacement' value.

[Back to top](#introduction)

# Download

### Gradle dependencies

```gradle
compile 'net.lachlanmckee:gsonpath:1.6.0'
apt 'net.lachlanmckee:gsonpath-compiler:1.6.0'
```

# License

```license
The MIT License (MIT)
Copyright (c) 2016 Lachlan McKee

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

[Back to top](#introduction)