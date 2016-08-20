---
layout: page
title: Gson Path
permalink: /gsonpath/
---

Last updated: August 21st 2016

# Introduction
Dealing with JSON is a common occurance these days when dealing with REST APIs. One of the most popular libraries for reading and writing JSON is [Gson](https://github.com/google/gson), a library written and maintained by Google.

Gson Path at its core is an annotation processor which generates Gson Type Adapters at compile time to avoid costly reflection at runtime.

# Features

Aside from generating typical Gson Type Adapters at compile time, there are many other features available within the library which create extremely powerful Type Adapters which allow you to further reduce boilerplate code.

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

# Download

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