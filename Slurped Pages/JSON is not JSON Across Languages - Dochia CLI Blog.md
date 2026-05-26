---
link: https://blog.dochia.dev/blog/json-isnt-json/
site: Dochia CLI Blog
excerpt: JSON (JavaScript Object Notation) was supposed to be the universal data interchange format that would solve the compatibility nightmares of XML. What looks like astraightforward data format becomes a minefield of subtle incompatibilities, edge cases, and implementation quirks that can break your applications in unexpected ways.
twitter: https://twitter.com/@dochiadev
slurped: 2026-05-10T16:54
title: JSON is not JSON Across Languages | Dochia CLI Blog
tags:
  - json
---

## Introduction: These Aren’t the JSONs You’re Looking For

JSON (JavaScript Object Notation) was **designed as a simple, lightweight, and human-readable** data interchange format, often positioned as a more accessible alternative to XML. It has become the de facto standard for web APIs and system integration. However, while the specification itself is straightforward, **different programming languages and libraries can interpret certain aspects of JSON differently**. What appears to be a uniform format can, in practice, lead to **subtle inconsistencies, edge cases, and implementation details** that developers need to be aware of when working across diverse platforms.

It turns out JSON isn’t **quite as universal in practice** as the spec would suggest. Who knew?

## The Promise vs. Reality: The JSON Specification Strikes Back

The JSON specification (RFC 7159/8259) is simple: objects, arrays, strings, numbers, booleans, and null. Six data types. Clean syntax. No namespace nightmares. No schema validation headaches. Just **pure data representation**.

This **simplicity was supposed to be JSON’s superpower**. Unlike XML with its verbose tags and complex parsing rules, JSON promised to be the data format that just worked everywhere. Write once, parse anywhere. But there is always a downside with simple things: **they tend to leave room for interpretation**. And when different languages and libraries start interpreting things differently, your “universal” data format becomes a bit less universal.

Consider this perfectly valid JSON:

```
{
  "id": 9007199254740993,
  "timestamp": "2023-12-25T10:30:00Z", 
  "temperature": 23.1,
  "readings": [null, "test"],
  "metadata": {
    "sensor": "室温",
    "location": "café"
  }
}
```

Your frontend parses this without complaint. Your backend processes it successfully. Your data pipeline ingests it just fine. But this **harmony is an illusion**. That **integer just lost precision** in JavaScript. The **timestamp means different things** to different parsers, UTC here, local time there, plain string somewhere else. And those Unicode characters in your metadata? Depends on the **normalization strategy**.

What appeared simple at first glance reveals a host of **subtle interoperability pitfalls**. The **JSON specification didn’t break**, it just **left certain details open**. Different **languages and libraries filled in those blanks differently**, and suddenly we’ve got a new Tower of Babel: everyone speaks JSON, but not everyone means the same thing.

## The Number Nightmare: Do Androids Dream of Electric Integers?

### Integer Precision Hell: HAL 9000’s Arithmetic Error

JavaScript represents all numbers as 64-bit floating-point values. This means integers larger than `Number.MAX_SAFE_INTEGER` (2^53 - 1 = 9007199254740991) lose precision:

```
// JavaScript
console.log(JSON.parse('{"id": 9007199254740992}').id === 9007199254740992); // true (this is MAX_SAFE_INTEGER + 1, but still representable)
console.log(JSON.parse('{"id": 9007199254740992}').id);                      // 9007199254740992

// The precision loss happens at larger values
console.log(JSON.parse('{"id": 9007199254740993}').id === 9007199254740993); // true because they bot get downscaled to 9007199254740992!
console.log(JSON.parse('{"id": 9007199254740993}').id);                      // 9007199254740992 (wrong!)

// At the limit, adding 1 doesn't change the value
const parsed = JSON.parse('{"id": 9007199254740992}').id;
console.log(parsed + 1 === parsed);     // true because they both get downscaled to 9007199254740992
console.log(parsed + 2);                // 9007199254740994 (works but skips odd numbers above MAX_SAFE_INTEGER)
```

But in Python, those same JSON values parse perfectly:

```
# Python
import json
data = json.loads('{"id": 9007199254740993}')
print(data['id'] == 9007199254740993)  # True
print(data['id'])                      # 9007199254740993
```

And in Go, it depends on what type you unmarshal into:

```
// Go - loses precision with interface{}
package main
import (
    "encoding/json"
    "fmt"
)

func main() {
    var data map[string]interface{}
    json.Unmarshal([]byte(`{"id": 9007199254740993}`), &data)
    fmt.Printf("%.0f\n", data["id"].(float64)) // 9007199254740992 - precision lost!
    
    // But correct with explicit int64
    var typed struct {
        ID int64 `json:"id"`
    }
    json.Unmarshal([]byte(`{"id": 9007199254740993}`), &typed)
    fmt.Println(typed.ID) // 9007199254740993 - correct!
}
```

In Java with Jackson:

```
// Java with Jackson - precision is generally preserved

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.JsonNode;

import java.util.Map;
import java.util.HashMap;

public class JsonPrecisionDemo {
    static class Data {
        public long id;
    }

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        // Jackson handles long integers correctly within Java's long range
        JsonNode node = mapper.readTree("{\"id\": 9007199254740993}");
        System.out.println(node.get("id").longValue()); // 9007199254740993 (correct)

        // Map also preserves precision for this value
        Map<String, Object> map = mapper.readValue("{\"id\": 9007199254740993}", HashMap.class);
        System.out.println(map.get("id"));              // 9007199254740993 (correct)

        // Even very large numbers within long range are handled correctly
        Map<String, Object> bigMap = mapper.readValue("{\"id\": 9223372036854775807}", HashMap.class);
        System.out.println(bigMap.get("id"));           // 9223372036854775807 (Long.MAX_VALUE)

        // Using specific type also works
        Data data = mapper.readValue("{\"id\": 9007199254740993}", Data.class);
        System.out.println(data.id);                    // 9007199254740993

        // Jackson only has issues with numbers beyond Java's native ranges
        // For numbers larger than Long.MAX_VALUE, you'd need BigInteger handling
    }
}
```

This inconsistency has **real-world consequences**. Database IDs, timestamps, and financial calculations can all be silently corrupted when crossing language boundaries.

### Decimal Precision Variations: The Floating Point Mentat Problem

While this is fundamentally an IEEE 754 floating-point issue rather than a JSON parsing inconsistency, it’s critical for developers to understand when working with **financial data, measurements, or any calculations** requiring exact decimal precision:

```
{
  "price": 0.1
}
```

```
// JavaScript
console.log(JSON.parse('{"price": 0.1}').price === 0.1);  // true
console.log(JSON.parse('{"price": 0.1}').price);          // 0.1

// The floating-point precision issue becomes apparent with arithmetic:
const price = JSON.parse('{"price": 0.1}').price;
console.log(price + 0.2);                                 // 0.30000000000000004
```

```
# Python (default json module)
import json
data = json.loads('{"price": 0.1}')
print(data['price'])                    # 0.1
print(data['price'] + 0.2)              # 0.30000000000000004

# Python with Decimal for precision
import json
from decimal import Decimal
data = json.loads('{"price": 0.1}', parse_float=Decimal)
print(data['price'])                    # 0.1
print(type(data['price']))              # <class 'decimal.Decimal'>
print(data['price'] + Decimal('0.2'))   # 0.3
```

```
// Java with Jackson - decimal precision

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.DeserializationFeature;

public class JsonDecimalDemo {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        JsonNode node = mapper.readTree("{\"price\": 0.1}");

        System.out.println(node.get("price").doubleValue());      // 0.1
        System.out.println(node.get("price").doubleValue() + 0.2); // 0.30000000000000004

        // Using BigDecimal for precision - need to enable the feature
        mapper.configure(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS, true);
        JsonNode preciseNode = mapper.readTree("{\"price\": 0.1}");
        System.out.println(preciseNode.get("price").decimalValue()); // 0.1 (as BigDecimal)
        System.out.println(preciseNode.get("price").decimalValue().add(
                new java.math.BigDecimal("0.2"))); // 0.3 (exact)
    }
}
```

```
// C# with System.Text.Json
using System;
using System.Text.Json;

class JsonPrecisionDemo 
{
    static void Main() 
    {
        var json = """{"price": 0.1}""";
        var doc = JsonDocument.Parse(json);
        double price = doc.RootElement.GetProperty("price").GetDouble();
        Console.WriteLine(price);           // 0.1
        Console.WriteLine(price + 0.2);     // 0.30000000000000004

        // Using decimal for precision would require custom converter
        doc.Dispose();
    }
}
```

I think it’s important to underly it again: **it’s critical for financial applications**. It can cause serious issues where exact decimal precision is required. Always use dedicated decimal types (Python’s `Decimal`, Java’s `BigDecimal`, JavaScript’s decimal libraries) for monetary calculations, never rely on JSON’s default number parsing for currency values.

## String Encoding Chaos: The Babel Fish Encoding Protocol

### Unicode Normalization: Ghost in the Shell Character Set

JSON strings can contain Unicode, but different languages handle normalization differently:

```
{
  "name": "José"
}
```

The character “é” can be represented as:

- A single codepoint: U+00E9 (é)
- Composed form: U+0065 U+0301 (e + ́)

```
# Python
import json
import unicodedata

# These are different byte sequences but visually identical
name1 = "José"                    # Single codepoint é (U+00E9)
name2 = "Jose\u0301"             # e (U+0065) + combining acute accent (U+0301)

print(name1 == name2)            # False
print(len(name1), len(name2))    # 4 5

json_str1 = json.dumps({"name": name1})
json_str2 = json.dumps({"name": name2})
print(json_str1 == json_str2)   # False!

# But after normalization:
normalized1 = unicodedata.normalize('NFC', name1)
normalized2 = unicodedata.normalize('NFC', name2)
print(normalized1 == normalized2) # True
```

```
// JavaScript
const name1 = "José";           // Single codepoint
const name2 = "Jose\u0301";     // Composed form

console.log(name1 === name2);   // false
console.log(name1.length, name2.length); // 4 5

console.log(JSON.stringify({name: name1}) === JSON.stringify({name: name2})); // false

// Normalization required for comparison
console.log(name1.normalize('NFC') === name2.normalize('NFC')); // true
```

```
// Java

import java.text.Normalizer;

public class UnicodeDemo {
    public static void main(String[] args) {
        String name1 = "José";                    // Single codepoint
        String name2 = "Jose\u0301";             // Composed form

        System.out.println(name1.equals(name2)); // false
        System.out.println(name1.length() + " " + name2.length()); // 4 5

        // Normalization required
        String norm1 = Normalizer.normalize(name1, Normalizer.Form.NFC);
        String norm2 = Normalizer.normalize(name2, Normalizer.Form.NFC);
        System.out.println(norm1.equals(norm2)); // true
    }
}
```

## Object Key Ordering: The Minority Report Hash Collision

JSON specification states that object key order is not significant, but real applications often depend on it. This becomes critical when using JSON for cryptographic operations like HMAC calculations, digital signatures, or content hashing where byte-for-byte consistency is required:

```
{
  "z": 1,
  "a": 2,
  "m": 3
}
```

```
// JavaScript (ES2015+)
const obj = JSON.parse('{"z": 1, "a": 2, "m": 3}');
console.log(Object.keys(obj)); // ['z', 'a', 'm'] - insertion order preserved

// Round trip maintains order
console.log(JSON.stringify(obj)); // {"z":1,"a":2,"m":3}
```

```
# Python 3.7+
import json

data = json.loads('{"z": 1, "a": 2, "m": 3}')
print(list(data.keys()))  # ['z', 'a', 'm'] - insertion order preserved

# Round trip maintains order
print(json.dumps(data))   # {"z": 1, "a": 2, "m": 3}
```

```
// Go maps are explicitly randomized for iteration
package main
import (
    "encoding/json"
    "fmt"
)

func main() {
    var data map[string]int
    json.Unmarshal([]byte(`{"z": 1, "a": 2, "m": 3}`), &data)
    
    // Iteration order is random by design
    fmt.Print("Keys: ")
    for k := range data {
        fmt.Print(k + " ")
    }
    fmt.Println()
    
    // But Marshal sorts keys alphabetically
    result, _ := json.Marshal(data)
    fmt.Println(string(result)) // {"a":2,"m":3,"z":1}
}
```

```
// Java with Jackson and LinkedHashMap preserves order

import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.LinkedHashMap;
import java.util.HashMap;
import java.util.Map;

public class JsonOrderDemo {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        // Using LinkedHashMap preserves insertion order
        LinkedHashMap<String, Integer> data = mapper.readValue(
                "{\"z\": 1, \"a\": 2, \"m\": 3}",
                LinkedHashMap.class
        );

        System.out.println(data.keySet()); // [z, a, m]

        // But regular HashMap doesn't guarantee order
        Map<String, Integer> hashData = mapper.readValue(
                "{\"z\": 1, \"a\": 2, \"m\": 3}",
                HashMap.class
        );
        System.out.println(hashData.keySet()); // Order may vary
    }
}
```

```
# Ruby (1.9+) preserves insertion order
require 'json'

data = JSON.parse('{"z": 1, "a": 2, "m": 3}')
puts data.keys.inspect  # ["z", "a", "m"]

# Round trip preserves order
puts JSON.generate(data)  # {"z":1,"a":2,"m":3}
```

### The Cryptographic Problem: Neuromancer’s Data Haven Leak

This ordering inconsistency breaks cryptographic operations that depend on exact byte representation:

```
// JavaScript - HMAC calculation
const crypto = require('crypto');
const data = {amount: 100, currency: "USD"};

// Different services might serialize differently:
const jsJson = JSON.stringify(data);     // {"amount":100,"currency":"USD"}
const goJson = '{"amount":100,"currency":"USD"}';  // Go sorts keys alphabetically

// Different HMAC signatures!
const jsHmac = crypto.createHmac('sha256', 'secret').update(jsJson).digest('hex');
const goHmac = crypto.createHmac('sha256', 'secret').update(goJson).digest('hex');

console.log(jsHmac === goHmac);  // false - authentication fails!
```

**Solution**: Always use a canonical JSON serialization for cryptographic operations, such as sorting keys alphabetically before signing.

## Null vs. Undefined vs. Missing: Schrödinger’s JSON Property

Different languages handle absence of values differently:

```
{
  "explicit_null": null,
  "empty_string": "",
  "zero_value": 0
}
```

```
// JavaScript
const obj = JSON.parse('{"explicit_null": null, "empty_string": "", "zero_value": 0}');

console.log(obj.explicit_null === null);        // true
console.log(obj.missing_key === undefined);     // true
console.log(obj.hasOwnProperty('explicit_null')); // true
console.log(obj.hasOwnProperty('missing_key'));   // false

// JSON.stringify omits undefined values
const withUndefined = {a: 1, b: undefined, c: null};
console.log(JSON.stringify(withUndefined)); // {"a":1,"c":null}
```

```
# Python
import json

obj = json.loads('{"explicit_null": null, "empty_string": "", "zero_value": 0}')

print(obj['explicit_null'] is None)    # True
print(obj.get('missing_key') is None)  # True - but different semantics!
print(obj.get('missing_key', 'default'))  # 'default'
print('explicit_null' in obj)          # True
print('missing_key' in obj)            # False

# Python can't represent undefined in JSON
data_with_none = {'a': 1, 'b': None, 'c': 0}
print(json.dumps(data_with_none))      # {"a": 1, "b": null, "c": 0}
```

```
// Go with interface{} can't distinguish null from missing
package main
import (
    "encoding/json"
    "fmt"
)

func main() {
    var data map[string]interface{}
    json.Unmarshal([]byte(`{"explicit_null": null, "empty_string": ""}`), &data)
    
    fmt.Println(data["explicit_null"] == nil)  // true
    fmt.Println(data["missing_key"] == nil)    // true - can't distinguish!
    
    // Use struct with pointers for proper null handling
    type Data struct {
        ExplicitNull *string `json:"explicit_null"`
        EmptyString  *string `json:"empty_string"`
        MissingKey   *string `json:"missing_key,omitempty"`
    }
    
    var typed Data
    json.Unmarshal([]byte(`{"explicit_null": null, "empty_string": ""}`), &typed)
    
    fmt.Println(typed.ExplicitNull == nil)   // true
    fmt.Println(*typed.EmptyString == "")    // true
    fmt.Println(typed.MissingKey == nil)     // true
}
```

```
// Java with Jackson

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.JsonNode;

public class JsonNullDemo {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        JsonNode node = mapper.readTree("{\"explicit_null\": null, \"empty_string\": \"\"}");

        System.out.println(node.get("explicit_null").isNull()); // true
        System.out.println(node.get("missing_key") == null);    // true - can't distinguish
        System.out.println(node.has("explicit_null"));         // true
        System.out.println(node.has("missing_key"));           // false
    }
}
```

## Date and Time Fun: The Groundhog Day Timezone Loop

JSON has no native date type, leading to endless format variations:

```
{
  "iso_string": "2023-01-15T10:30:00.000Z",
  "unix_timestamp": 1673780200,
  "unix_milliseconds": 1673780200000,
  "date_only": "2023-01-15",
  "custom_format": "15/01/2023 10:30:00"
}
```

```
// JavaScript
const dates = JSON.parse(`{
  "iso_string": "2023-01-15T10:30:00.000Z",
  "unix_timestamp": 1673780200,
  "unix_milliseconds": 1673780200000
}`);

console.log(new Date(dates.iso_string));         // Sun Jan 15 2023 10:30:00 GMT+0000
console.log(new Date(dates.unix_milliseconds));  // Sun Jan 15 2023 10:56:40 GMT+0000
console.log(new Date(dates.unix_timestamp));     // Thu Jan 19 1970 08:56:20 GMT+0000 (WRONG!)

// JavaScript Date constructor expects milliseconds, not seconds
console.log(new Date(dates.unix_timestamp * 1000)); // Sun Jan 15 2023 10:56:40
```

```
# Python
import json
from datetime import datetime, timezone

dates = json.loads("""{
  "iso_string": "2023-01-15T10:30:00.000Z",
  "unix_timestamp": 1673780200,
  "unix_milliseconds": 1673780200000
}""")

# ISO string parsing (need to handle Z)
iso_date = datetime.fromisoformat(dates['iso_string'].replace('Z', '+00:00'))
print(iso_date)  # 2023-01-15 10:30:00+00:00

# Unix timestamp (seconds)
unix_date = datetime.fromtimestamp(dates['unix_timestamp'], tz=timezone.utc)
print(unix_date)  # 2023-01-15 10:56:40+00:00

# Unix milliseconds
ms_date = datetime.fromtimestamp(dates['unix_milliseconds'] / 1000, tz=timezone.utc)
print(ms_date)   # 2023-01-15 10:56:40+00:00
```

```
// Java

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.JsonNode;

import java.time.Instant;
import java.time.ZonedDateTime;
import java.time.format.DateTimeFormatter;

public class JsonDateDemo {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        String jsonString = "{" +
                "\"iso_string\": \"2023-01-15T10:30:00.000Z\"," +
                "\"unix_timestamp\": 1673780200," +
                "\"unix_milliseconds\": 1673780200000" +
                "}";
        JsonNode dates = mapper.readTree(jsonString);

        // ISO string parsing
        String isoString = dates.get("iso_string").asText();
        ZonedDateTime isoDate = ZonedDateTime.parse(isoString, DateTimeFormatter.ISO_DATE_TIME);
        System.out.println(isoDate); // 2023-01-15T10:30Z

        // Unix timestamp (seconds)
        long unixSeconds = dates.get("unix_timestamp").asLong();
        Instant unixDate = Instant.ofEpochSecond(unixSeconds);
        System.out.println(unixDate); // 2023-01-15T10:56:40Z

        // Unix milliseconds
        long unixMillis = dates.get("unix_milliseconds").asLong();
        Instant msDate = Instant.ofEpochMilli(unixMillis);
        System.out.println(msDate);   // 2023-01-15T10:56:40Z
    }
}
```

```
// C#
using System;
using System.Text.Json;

class JsonDateDemo 
{
    static void Main() 
    {
        var json = """
        {
          "iso_string": "2023-01-15T10:30:00.000Z",
          "unix_timestamp": 1673780200,
          "unix_milliseconds": 1673780200000
        }
        """;

        using var doc = JsonDocument.Parse(json);
        var root = doc.RootElement;

        // ISO string parsing
        var isoString = root.GetProperty("iso_string").GetString();
        var isoDate = DateTime.Parse(isoString);
        Console.WriteLine(isoDate); // 1/15/2023 10:30:00 AM

        // Unix timestamp (seconds) - C# uses milliseconds
        var unixSeconds = root.GetProperty("unix_timestamp").GetInt64();
        var unixDate = DateTimeOffset.FromUnixTimeSeconds(unixSeconds);
        Console.WriteLine(unixDate); // 1/15/2023 10:56:40 AM +00:00

        // Unix milliseconds
        var unixMillis = root.GetProperty("unix_milliseconds").GetInt64();
        var msDate = DateTimeOffset.FromUnixTimeMilliseconds(unixMillis);
        Console.WriteLine(msDate);   // 1/15/2023 10:56:40 AM +00:00
    }
}
```

## Error Handling Inconsistencies: I’m Sorry Dave, I Can’t Parse That

Different parsers fail differently on malformed JSON:

```
// JavaScript (V8)
// Duplicate keys - last value wins
console.log(JSON.parse('{"a": 1, "a": 2}'));  // {a: 2}

// Trailing commas - SyntaxError
try {
    JSON.parse('{"a": 1,}');
} catch (e) {
    console.log("Trailing comma rejected");  // This runs
}

// Leading zeros - SyntaxError
try {
    JSON.parse('{"num": 007}');
} catch (e) {
    console.log("Leading zeros rejected");   // This runs
}

// Single quotes - SyntaxError  
try {
    JSON.parse("{'a': 1}");
} catch (e) {
    console.log("Single quotes rejected");   // This runs
}
```

```
# Python json module
import json

# Duplicate keys - last value wins
data = json.loads('{"a": 1, "a": 2}')
print(data)  # {'a': 2}

# Trailing commas - JSONDecodeError
try:
    json.loads('{"a": 1,}')
except json.JSONDecodeError as e:
    print(f"Trailing comma rejected: {e}")

# Leading zeros - JSONDecodeError
try:
    json.loads('{"num": 007}')
except json.JSONDecodeError as e:
    print(f"Leading zeros rejected: {e}")

# Single quotes - JSONDecodeError
try:
    json.loads("{'a': 1}")
except json.JSONDecodeError as e:
    print(f"Single quotes rejected: {e}")
```

```
// Go json package
package main
import (
    "encoding/json"
    "fmt"
)

func main() {
    var data map[string]int
    
    // Duplicate keys - last value wins (no error)
    err := json.Unmarshal([]byte(`{"a": 1, "a": 2}`), &data)
    if err != nil {
        fmt.Println("Duplicate key error:", err)
    } else {
        fmt.Printf("Duplicate keys allowed, value: %d\n", data["a"]) // 2
    }
    
    // Trailing commas - error
    err = json.Unmarshal([]byte(`{"a": 1,}`), &data)
    if err != nil {
        fmt.Println("Trailing comma error:", err)
    }
    
    // Leading zeros - error
    err = json.Unmarshal([]byte(`{"num": 007}`), &data)
    if err != nil {
        fmt.Println("Leading zeros error:", err)
    }
    
    // Single quotes - error
    err = json.Unmarshal([]byte(`{'a': 1}`), &data)
    if err != nil {
        fmt.Println("Single quotes error:", err)
    }
}
```

```
// Java with Jackson

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.databind.JsonNode;

public class JsonErrorDemo {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        // Duplicate keys - last wins by default
        try {
            JsonNode node = mapper.readTree("{\"a\": 1, \"a\": 2}");
            System.out.println("Duplicate keys allowed: " + node.get("a").asInt()); // 2
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }

        // Strict duplicate key detection
        ObjectMapper strictMapper = new ObjectMapper();
        strictMapper.configure(JsonParser.Feature.STRICT_DUPLICATE_DETECTION, true);
        try {
            strictMapper.readTree("{\"a\": 1, \"a\": 2}");
        } catch (Exception e) {
            System.out.println("Strict mode rejects duplicates: " + e.getMessage());
        }

        // Trailing commas - can be enabled
        ObjectMapper lenientMapper = new ObjectMapper();
        lenientMapper.configure(JsonParser.Feature.ALLOW_TRAILING_COMMA, true);
        try {
            JsonNode node = lenientMapper.readTree("{\"a\": 1,}");
            System.out.println("Trailing comma allowed: " + node.get("a").asInt()); // 1
        } catch (Exception e) {
            System.out.println("Trailing comma error: " + e.getMessage());
        }
    }
}
```

## Real-World War Stories: When Code Becomes Skynet

### The Twitter ID Problem: The Matrix Integer Overflow

Twitter’s tweet IDs exceed JavaScript’s safe integer range. Their API returns:

```
{
  "id": 1234567890123456789,
  "id_str": "1234567890123456789"
}
```

```
// JavaScript clients lose precision
const tweet = {
  "id": 1234567890123456789,
  "id_str": "1234567890123456789"
};

console.log(tweet.id);     // 1234567890123456768 (wrong!)
console.log(tweet.id_str); // "1234567890123456789" (correct)

// Must use id_str in JavaScript
const tweetFromAPI = JSON.parse(`{
  "id": 1234567890123456789,
  "id_str": "1234567890123456789"
}`);

console.log(tweetFromAPI.id === 1234567890123456789);  // false!
console.log(tweetFromAPI.id_str === "1234567890123456789");  // true
```

```
# Python handles large integers correctly
import json

tweet_json = '{"id": 1234567890123456789, "id_str": "1234567890123456789"}'
tweet = json.loads(tweet_json)

print(tweet['id'] == 1234567890123456789)      # True
print(tweet['id_str'] == "1234567890123456789") # True

# Python clients can use either field safely
```

### The PostgreSQL JSON Type: Foundation’s Psychohistory Database

PostgreSQL’s JSON type preserves exact text representation, while JSONB normalizes it:

```
-- Create test table
CREATE TABLE json_test (
    id SERIAL PRIMARY KEY,
    data_json JSON,
    data_jsonb JSONB
);

-- Insert identical data
INSERT INTO json_test (data_json, data_jsonb) 
VALUES ('{"b": 1, "a": 2}', '{"b": 1, "a": 2}');

-- JSON preserves original formatting
SELECT data_json FROM json_test;
-- Returns: {"b": 1, "a": 2}

-- JSONB normalizes and sorts keys  
SELECT data_jsonb FROM json_test;
-- Returns: {"a": 2, "b": 1}

-- This affects application behavior
INSERT INTO json_test (data_json, data_jsonb) 
VALUES ('{"price": 19.99}', '{"price": 19.99}');

-- Extracting values may behave differently
SELECT data_json->>'price' as json_price, 
       data_jsonb->>'price' as jsonb_price 
FROM json_test WHERE id = 2;
-- Both return: "19.99"

-- But binary operations only work on JSONB
SELECT * FROM json_test WHERE data_jsonb @> '{"a": 2}';  -- Works
-- SELECT * FROM json_test WHERE data_json @> '{"a": 2}';  -- Error!
```

### The MongoDB Extended JSON Problem: The Hitchhiker’s Guide to NoSQL

MongoDB uses Extended JSON which includes additional types:

```
// Standard JSON
{
  "date": "2023-01-15T10:30:00.000Z",
  "id": "507f1f77bcf86cd799439011"
}

// MongoDB Extended JSON
{
  "date": {"$date": {"$numberLong": "1673780200000"}},
  "id": {"$oid": "507f1f77bcf86cd799439011"}
}
```

```
// JavaScript with MongoDB driver
const { MongoClient, ObjectId } = require('mongodb');

// Data going into MongoDB
const document = {
  _id: new ObjectId("507f1f77bcf86cd799439011"),
  created_at: new Date("2023-01-15T10:30:00.000Z"),
  count: 42
};

// When exported as Extended JSON (using MongoDB's EJSON)
const EJSON = require('bson').EJSON;
const extendedJson = EJSON.stringify(document, null, 2);
console.log(extendedJson);
/*
{
  "_id": {"$oid": "507f1f77bcf86cd799439011"},
  "created_at": {"$date": "2023-01-15T10:30:00.000Z"},
  "count": 42
}
*/

// Other languages can't parse this directly
```

```
# Python needs special handling
import json
from bson import ObjectId
from datetime import datetime

# Standard JSON from MongoDB export
extended_json = '''{
  "_id": {"$oid": "507f1f77bcf86cd799439011"},
  "created_at": {"$date": "2023-01-15T10:30:00.000Z"}
}'''

# Custom parser needed
def parse_extended_json(obj):
    if isinstance(obj, dict):
        if '$oid' in obj:
            return ObjectId(obj['$oid'])
        elif '$date' in obj:
            return datetime.fromisoformat(obj['$date'].replace('Z', '+00:00'))
        else:
            return {k: parse_extended_json(v) for k, v in obj.items()}
    elif isinstance(obj, list):
        return [parse_extended_json(item) for item in obj]
    return obj

data = json.loads(extended_json, object_hook=parse_extended_json)
```

## Mitigation Strategies: The Dune Survival Manual for JSON

### Use Schema Validation: The First Law of JSON Robotics

Define and validate schemas across all services:

```
// JavaScript validation example
const Ajv = require('ajv');
const addFormats = require('ajv-formats');

const ajv = new Ajv();
addFormats(ajv);

const schema = {
  type: 'object',
  properties: {
    id: { type: 'string', pattern: '^[0-9]+$' },
    price: { type: 'number', multipleOf: 0.01 }
  },
  required: ['id', 'price']
};

const validate = ajv.compile(schema);

const data = { id: "9007199254740992", price: 19.99 };
const valid = validate(data);

if (!valid) {
  console.log('Validation errors:', validate.errors);
}
```

```
# Python validation with jsonschema
import json
import jsonschema

schema = {
    "type": "object",
    "properties": {
        "id": {"type": "string", "pattern": "^[0-9]+$"},
        "price": {"type": "number", "multipleOf": 0.01}
    },
    "required": ["id", "price"]
}

data = {"id": "9007199254740992", "price": 19.99}

try:
    jsonschema.validate(instance=data, schema=schema)
    print("Data is valid")
except jsonschema.ValidationError as e:
    print(f"Validation error: {e.message}")
```

### Normalize Data Types: Ender’s Data Normalization Game

Establish conventions for your organization:

```
// JavaScript - safe serialization helpers
function safeJSONStringify(obj) {
  return JSON.stringify(obj, (key, value) => {
    // Convert large numbers to strings
    if (typeof value === 'number' && Math.abs(value) > Number.MAX_SAFE_INTEGER) {
      return value.toString();
    }
    
    // Convert BigInt to string
    if (typeof value === 'bigint') {
      return value.toString();
    }
    
    // Normalize dates to ISO 8601
    if (value instanceof Date) {
      return value.toISOString();
    }
    
    return value;
  });
}

const data = {
  id: BigInt("9007199254740992"),
  created_at: new Date(),
  price: 19.99
};

console.log(safeJSONStringify(data));
// {"id":"9007199254740992","created_at":"2023-01-15T10:30:00.000Z","price":19.99}
```

### Library Selection Matters: Choose Your JSON Parser, Neo

Choose JSON libraries carefully based on your needs:

```
// JavaScript - handling large integers safely
// Using json-bigint library
const JSONbig = require('json-bigint');

// Configure JSONbig to handle large numbers
const JSONbigConfig = JSONbig({
  alwaysParseAsBig: false,
  useNativeBigInt: true,
  storeAsString: false
});

const data = '{"smallNumber": 123, "largeNumber": 9007199254740993}';

// Standard JSON loses precision on this larger number
const standard = JSON.parse(data);
console.log(standard.largeNumber === 9007199254740993); // false (precision lost)
console.log(standard.largeNumber); // 9007199254740992

// JSONbig preserves precision
const safe = JSONbigConfig.parse(data);
console.log(safe.largeNumber === 9007199254740993n); // true (BigInt)
console.log(safe.largeNumber); // 9007199254740993n
```

```
// Java - Jackson configuration for safety

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.core.JsonParser;

import java.math.BigDecimal;
import java.math.BigInteger;

public class JsonSafeParsingDemo {
    public static void main(String[] args) throws Exception {
        ObjectMapper safeMapper = new ObjectMapper();

        // Configure for precision and safety
        safeMapper.configure(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS, true);
        safeMapper.configure(DeserializationFeature.USE_BIG_INTEGER_FOR_INTS, true);
        safeMapper.configure(JsonParser.Feature.STRICT_DUPLICATE_DETECTION, true);

        String json = """{"price": 19.99, "id": 9007199254740992}""";
        JsonNode node = safeMapper.readTree(json);

        // Now we get BigDecimal and BigInteger types
        System.out.println("Price type: " + node.get("price").numberValue().getClass().getSimpleName());  // BigDecimal
        System.out.println("ID type: " + node.get("id").numberValue().getClass().getSimpleName());       // BigInteger

        // Values are preserved with full precision
        System.out.println("Price value: " + node.get("price").decimalValue()); // 19.99
        System.out.println("ID value: " + node.get("id").bigIntegerValue());    // 9007199254740992
    }
}
```

### Test Cross-Language Compatibility: The Martian Compatibility Protocol

Create comprehensive test suites that verify data round-trips correctly across all services in your architecture. This is critical for detecting subtle incompatibilities before they cause production issues.

**Essential Test Categories:**

**Numeric Precision Tests:**

- Test integers at the boundaries of JavaScript’s safe integer range (2^53-1)
- Verify large database IDs don’t lose precision when passed through JavaScript services
- Test decimal values that are commonly problematic (0.1, 0.2, financial amounts)
- Validate that monetary calculations remain exact across all services

**Unicode and String Handling:**

- Test strings with different Unicode normalization forms (NFC vs NFD)
- Verify proper handling of emoji, accented characters, and non-Latin scripts
- Test strings containing control characters, null bytes, and escape sequences
- Validate that search and comparison operations work consistently

**Data Structure Integrity:**

- Test object key ordering preservation where required (especially for cryptographic operations)
- Verify null vs undefined vs missing field handling across languages
- Test empty arrays vs null arrays vs missing arrays
- Validate nested object structures maintain their shape

**Date and Time Consistency:**

- Test ISO 8601 strings with and without timezone information
- Verify Unix timestamp handling (seconds vs milliseconds)
- Test edge cases like leap seconds, daylight saving transitions
- Validate that date arithmetic produces consistent results

**Error Handling Uniformity:**

- Test malformed JSON handling (trailing commas, duplicate keys, invalid escapes)
- Verify that validation errors are consistent across services
- Test boundary conditions (very large payloads, deeply nested objects)
- Validate that error responses maintain the same format

**Cryptographic Consistency:**

- Test that HMAC signatures match when the same data is serialized by different services
- Verify that content hashes are identical for semantically equivalent data
- Test digital signature verification across language boundaries
- Validate that canonical JSON serialization works consistently

**Performance and Memory Behavior:**

- Test large payload handling to identify memory usage differences
- Verify that streaming vs tree parsing produces identical results
- Test concurrent parsing behavior under load
- Validate that garbage collection patterns don’t affect data integrity

**Real-World Scenario Testing:**

- Test actual API payloads from your production systems
- Verify third-party integration data (payment processors, external APIs)
- Test data migration scenarios between different storage systems
- Validate that cached vs fresh data produces identical results

**Automated Compatibility Matrix:**

Set up automated tests that run the same test cases against services written in different languages, creating a compatibility matrix that shows which combinations work reliably. This should be part of your CI/CD pipeline and run before any deployment that changes JSON handling logic.

## Conclusion: May the Parse Be With You

JSON’s elegant simplicity is both its **greatest strength and also its weakness**. What looks like a straightforward data format transforms into something a bit more complicated when you start moving data between systems that were built by different teams, using different languages, with different assumptions about how the world works. **The issues are real**: large integers silently lose precision, Unicode strings get mangled in creative ways, object key ordering disappears when you need it most, and date parsing becomes a choose-your-own-adventure game. But here’s the thing: these problems won’t hit you every day.

When these issues do surface, they create some of the most soul-crushing debugging experiences in software development. You’ll spend **days tracking down** why user “José” can log in but “José” (with a different Unicode normalization) cannot. You’ll pull your hair out wondering why your HMAC validation works in development but fails in production, only to discover it’s because your API Gateway reorders the JSON keys.

The intermittent nature of these bugs makes them particularly insidious. They **hide in edge cases**, waiting for that one specific user ID, that one particular Unicode sequence, or that one unfortunate key ordering to surface in production and ruin your weekend.

So what’s the solution? Use Protocol Buffers! Just kidding, don’t get me started on that one. **Establish clear conventions**. Validate schemas rigorously. Test cross-language compatibility early and often. Choose your **libraries based on their track record**, not just their performance benchmarks. And when something “impossible” happens in production, remember that JSON parsers are written by humans with different constraints and caffeine levels. JSON isn’t JSON across all languages, but with **proper engineering discipline** and a **healthy dose of paranoia**, you can build systems that **work reliably** despite these inconsistencies. The key is acknowledging that the problem exists in the first place, rather than pretending JSON is the simple, universal format it appears to be.

Remember: in a world of imperfect standards and human-written and soon AI-written parsers, a **little skepticism goes a long way**. Trust, but verify. And always test your edge cases before they test you.