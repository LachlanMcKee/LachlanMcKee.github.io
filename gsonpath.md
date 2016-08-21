---
layout: page
title: Gson Path
permalink: /gsonpath/
---

Last updated: August 21st 2016

* [Download](#download)
* [License](#license)

# Introduction
Reading and writing JSON is a common practice when working with REST APIs. One of the most popular Java libraries for reading and writing JSON is [Gson](https://github.com/google/gson), a library written and maintained by Google.

Gson Path at its core is a Java annotation processor which generates Gson Type Adapters at compile time to avoid costly reflection at runtime. It also hosts a multitude of other features which will be described within this document.

# Usage

Getting started using Gson Path is extremely simple. If you have not already done so, go to the [download](#download) section and add the dependency to your build.

Since Gson Path is an annotation processor, the majority of the work is done for you during compilation. Every class annotated with the `@AutoGsonAdapter` annotation will prompt the library to generate a Type Adapter. There are many examples of the annotation usage described in the [features](#features) section.

To make use of these generated classes, you must add the Gson Path Type Adapter Factory:

```java
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.registerTypeAdapterFactory(GsonPath.createTypeAdapterFactory());
Gson gson = gsonBuilder.create();
```

This Type Adapter factory adds all of the generated Type Adapters into Gson without a developer needing to reference the classes directly.

# Features

Aside from generating typical Gson Type Adapters at compile time, there are many other features available within the library which create extremely powerful Type Adapters which allow you to further reduce boilerplate code.

1. [Basic Json Path support](#basic-json-path-support)
2. [Client side validation](#client-side-validation)
3. [Generate POJOs using interfaces](#generate-pojos-using-interfaces)
4. [Json Path Substitutions](#json-path-substitutions)

## Basic Json Path support

Gson supports marshalling and unmarshalling JSON content into Java POJOs. Unfortunately the library requires a one-to-one relationship between a POJO and a JSON object. This means that the more complex a JSON document is, the more POJOs a developer must create.

Gson Path adds basic [Json Path](http://goessner.net/articles/JsonPath/) support to solve this problem. Json Path is a query language similar to [XPath](https://en.wikipedia.org/wiki/XPath), which allows you to reference elements within a document through shorthand notation (e.g. `parent.child`).

This shorthand notation is extremely useful to avoid creating numerous Java POJOs to access a particular subsection of a JSON document. Gson Path takes advantage of this Json Path 'dot notation' concept to give developers the ability to reference specific fields within a JSON tree from the standard Gson `SerializedName` annotation (this works for Type Adapter reading and writing).

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

We can deserialize the JSON with a single Java class multiple ways. The following classes demonstrate the annotations required to create a type adapter which can correctly read the nested content.

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
The `rootField` property of the `@AutoGsonAdapter` annotation is useful to avoid repetition.

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
If the name of the field matches the name of the JSON key, the `@SerializedName` annotation can be omitted.

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

## Client side validation

By adding `@Nullable` and `@NonNull` annotations to your POJOs you are able to optionally turn on a degree of client side validation in the form of required fields.

The `@AutoGsonAdapter` has a property called `fieldValidationType` which specifies if and how the generated Type Adapter will validate the JSON. There are three different possible validation types:

1. `NO_VALIDATION` - No fields are validated, and the Gson parser should never raise exceptions for missing content.
2. `VALIDATE_EXPLICIT_NON_NULL` - Any Objects marked with `@NonNull` (or similar annotation), or primitives (since primitives cannot be null) should fail if the value does not exist within the JSON.
3. `VALIDATE_ALL_EXCEPT_NULLABLE` - All fields will be treated as `@NonNull` (even when they are not annotated as such), and should fail when value is not found. The only exceptions to this rule is when a field is annotation with `@Nullable` (except for primitives).

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

## Generate POJOs using interfaces

## Json Path Substitutions

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