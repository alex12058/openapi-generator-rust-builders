# OpenAPI Generator - Rust Builder Pattern Fork

**A fork of [OpenAPI Generator](https://github.com/OpenAPITools/openapi-generator) with enhanced Rust client generation featuring automatic builder pattern implementation.**

This fork modifies the Rust generator (reqwest library) to automatically generate builder patterns for all API operations with parameters, significantly improving ergonomics when working with APIs that have many optional parameters.

## 🎯 Key Difference from Base Generator

### Standard Generator
```rust
// Functions with many individual parameters - difficult to use
pub async fn get_time_series(
    configuration: &Configuration,
    symbol: &str,
    interval: &str,
    exchange: Option<&str>,
    mic_code: Option<&str>,
    country: Option<&str>,
    r_type: Option<&str>,
    outputsize: Option<i32>,
    format: Option<&str>,
    delimiter: Option<&str>,
    prepost: Option<bool>,
    dp: Option<i32>,
    order: Option<&str>,
    timezone: Option<&str>,
    date: Option<&str>,
    start_date: Option<&str>,
    end_date: Option<&str>,
    previous_close: Option<bool>,
    // ... 21+ parameters total
) -> Result<GetTimeSeries200Response, Error<GetTimeSeriesError>>
```

### This Fork
```rust
// Clean builder pattern with Params struct
pub async fn get_time_series(
    configuration: &Configuration,
    params: GetTimeSeriesParams,
) -> Result<GetTimeSeries200Response, Error<GetTimeSeriesError>>

// Usage with builder pattern
let params = GetTimeSeriesParams::builder()
    .symbol("AAPL")
    .interval("1day")
    .outputsize(10)
    .build();

let response = time_series_api::get_time_series(&config, params).await?;
```

## 🚀 What This Fork Adds

1. **Automatic Params Struct Generation**: Every API operation with parameters gets a corresponding `Params` struct with all fields as `Option<T>` and `#[derive(Clone, Debug, Default)]`

2. **Builder Pattern Implementation**: Each `Params` struct automatically gets:
   - A `ParamsBuilder` struct
   - A `builder()` method that returns `ParamsBuilder`
   - Chainable setter methods using `impl Into<String>` for ergonomic string conversion
   - A `build()` method that constructs the final `Params`

3. **Simplified Function Signatures**: All API functions accept `(config, params)` instead of dozens of individual parameters

4. **Universal Application**: The builder pattern is applied to **all** operations with parameters, not requiring special vendor extensions

## 📦 Installation & Build

### Prerequisites
- Java 21 or later (required for Maven build)
- Maven (included via `./mvnw` wrapper)

### Build Instructions

1. Clone this repository:
```bash
git clone https://github.com/alex12058/openapi-generator-rust-builders.git
cd openapi-generator-rust-builders
```

2. Build the generator:
```bash
./mvnw clean package -DskipTests
```

This produces the JAR file at:
```
modules/openapi-generator-cli/target/openapi-generator-cli.jar
```

## 🔧 Usage

### Generate Rust Client with Builder Pattern

```bash
java -jar modules/openapi-generator-cli/target/openapi-generator-cli.jar generate \
  -i /path/to/your/openapi.json \
  -g rust \
  -o /path/to/output \
  --additional-properties=packageName=my_client,packageVersion=0.1.0,library=reqwest
```

### Parameters
- `-i`: Path to your OpenAPI specification file (JSON or YAML)
- `-g rust`: Specifies Rust generator
- `-o`: Output directory for generated code
- `--additional-properties`:
  - `packageName`: Name of the generated Rust crate
  - `packageVersion`: Version number
  - `library=reqwest`: Use reqwest HTTP client (required for builder pattern)

## 📝 Example Generated Code

### Generated Params Struct
```rust
#[derive(Clone, Debug, Default)]
pub struct GetTimeSeriesParams {
    pub symbol: Option<String>,
    pub interval: Option<String>,
    pub exchange: Option<String>,
    pub outputsize: Option<i32>,
    pub format: Option<String>,
    // ... other fields
}
```

### Generated Builder
```rust
pub struct GetTimeSeriesParamsBuilder {
    params: GetTimeSeriesParams,
}

impl GetTimeSeriesParamsBuilder {
    pub fn symbol(mut self, symbol: impl Into<String>) -> Self {
        self.params.symbol = Some(symbol.into());
        self
    }
    
    pub fn interval(mut self, interval: impl Into<String>) -> Self {
        self.params.interval = Some(interval.into());
        self
    }
    
    // ... other setters
    
    pub fn build(self) -> GetTimeSeriesParams {
        self.params
    }
}
```

### Usage Example
```rust
use twelve_data_client::apis::{configuration, time_series_api};
use twelve_data_client::apis::time_series_api::GetTimeSeriesParams;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Configure authentication
    let mut config = configuration::Configuration::new();
    config.api_key = Some(configuration::ApiKey {
        prefix: Some("apikey".to_string()),
        key: std::env::var("API_KEY")?,
    });

    // Build parameters using builder pattern
    let params = GetTimeSeriesParams::builder()
        .symbol("AAPL")
        .interval("1day")
        .outputsize(10)
        .build();

    // Make API call with clean signature
    let response = time_series_api::get_time_series(&config, params).await?;
    
    println!("Response: {:?}", response);
    Ok(())
}
```

## 🔍 Technical Details

### Modified Template
The changes are implemented in:
```
modules/openapi-generator/src/main/resources/rust/reqwest/api.mustache
```

### Key Modifications
1. **Lines 16-118**: Builder pattern code generation wrapped in `{{#hasParams}}` conditional
2. **Lines 178-233**: Function signature variations based on parameter presence
3. **Lines 194-198**: Field extraction from params struct inside function body

### Template Logic
- Uses `{{#hasParams}}` Mustache conditional to detect operations with parameters
- Generates `Params` struct with all parameters as `Option<T>` fields
- Generates `ParamsBuilder` with chainable setters
- Modifies function signature to accept `params: Params` instead of individual parameters
- Extracts fields from params struct: `let p_symbol = params.symbol;`

## 🆚 Comparison with Base Generator

| Feature | Base Generator | This Fork |
|---------|---------------|-----------|
| Parameter Handling | Individual parameters | Params struct |
| Function Signature | `fn(config, p1, p2, ..., p21)` | `fn(config, params)` |
| Builder Pattern | Manual implementation required | Automatic |
| Optional Parameters | Function parameters | Struct fields |
| Ergonomics | Poor with 20+ params | Excellent with builder |
| Vendor Extensions | Not applicable | Not required |

## 🤝 Contributing

This fork maintains compatibility with the base OpenAPI Generator project. When contributing:

1. Keep changes isolated to the Rust reqwest template
2. Ensure builder pattern applies universally to all parameterized operations
3. Test with OpenAPI specs containing operations with many optional parameters
4. Follow existing Mustache template conventions

## 📄 License

This project maintains the same Apache 2.0 License as the original OpenAPI Generator project.

## 🔗 Links

- **Original Project**: [OpenAPI Generator](https://github.com/OpenAPITools/openapi-generator)
- **Original Documentation**: [OpenAPI Generator Docs](https://openapi-generator.tech/)
- **Rust Generator Docs**: [Rust Client Generator](https://openapi-generator.tech/docs/generators/rust/)

## ⚠️ Notes

- This fork specifically modifies the **reqwest** library variant of the Rust generator
- Builder pattern is applied **automatically** to all operations with parameters
- No special vendor extensions needed in your OpenAPI spec
- The generated code requires `tokio` runtime for async execution
- Operations without parameters retain their original simple signatures

---

## About OpenAPI Generator

This fork is based on [OpenAPI Generator](https://github.com/OpenAPITools/openapi-generator), an open-source project that generates API clients, server stubs, and documentation from OpenAPI specifications. For more information about the base project, visit:

- **Base Project**: [OpenAPI Generator GitHub](https://github.com/OpenAPITools/openapi-generator)
- **Documentation**: [OpenAPI Generator Docs](https://openapi-generator.tech/)
- **All Supported Languages**: See the [base project README](https://github.com/OpenAPITools/openapi-generator#overview) for the full list of supported languages and frameworks

---

## License

Copyright 2018 OpenAPI-Generator Contributors (https://openapi-generator.tech)  
Copyright 2018 SmartBear Software

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at [apache.org/licenses/LICENSE-2.0](https://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
