# UEI EV-Charging OpenAPI Specification - Validation Rules Added

This document outlines all the validation rules that have been added to the `simple-energy_EV_1.1.0_openapi_3.1.yaml` file to ensure proper data validation for the EV charging use case.

## Overview

The UEI EV-charging OpenAPI specification includes comprehensive validation rules for various API endpoints, ensuring that:
- Required fields are present
- Enum values are valid
- Data structures conform to expected schemas
- Business logic constraints are enforced

## API Endpoints with Validation Rules

### 1. Search API (`/search`)

#### Intent Fulfillment Stops Validation
- **Rule**: Must contain at least one stop with location information
- **Path**: `message.intent.fulfillment.stops`
- **Validation**: `contains` at least one object with `location` property

#### Connector Type Tags Validation
- **Rule**: If a tag has `code: "connector-type"`, its value must be one of the predefined enum values
- **Path**: `message.intent.fulfillment.tags`
- **Enum Values**:
  - `CHADEMO` - CHAdeMO connector
  - `DOMESTIC_A` through `DOMESTIC_O` - Domestic outlet types (A-O)
  - `IEC_60309_2_single_16` - IEC 60309-2 single phase connector, 16A
  - `IEC_60309_2_three_16` - IEC 60309-2 three phase connector, 16A
  - `IEC_60309_2_three_32` - IEC 60309-2 three phase connector, 32A
  - `IEC_60309_2_three_64` - IEC 60309-2 three phase connector, 64A
  - `IEC_62196_T1` - IEC 62196 Type 1 connector
  - `IEC_62196_T1_COMBO` - IEC 62196 Type 1 connector with Combo 2
  - `IEC_62196_T2` - IEC 62196 Type 2 connector
  - `IEC_62196_T2_COMBO` - IEC 62196 Type 2 connector with Combo 2
  - `IEC_62196_T3A` - IEC 62196 Type 3A connector
  - `IEC_62196_T3C` - IEC 62196 Type 3C connector
  - `TESLA_R` - Tesla connector
  - `TESLA_S` - Tesla connector

### 2. On-Search API (`/on_search`)

#### Connector Specifications Tags Validation
- **Rule**: Must contain at least one tag with `code: "connector-specifications"`
- **Path**: `message.catalog.providers[].items[].tags`
- **Validation**: `contains` at least one tag with the specified code

#### Connector Specification List Validation
- **Rule**: When `code: "connector-specifications"`, the `list` can only contain items with these specific codes
- **Allowed Codes in List** (only these codes are permitted):
  - `connector-id` - Unique identifier for the connector
  - `power-type` - Type of power (AC/DC)
  - `connector-type` - Specific connector type
  - `connector-format` - Format (SOCKET/CABLE)
  - `charging-speed` - Speed classification
  - `power-rating` - Power rating value
  - `status` - Availability status

#### Connector Type Value Validation
- **Rule**: When `code: "connector-type"`, value must be from the predefined enum
- **Complete Enum Values**:
  - `CHADEMO` - CHAdeMO connector
  - `DOMESTIC_A` through `DOMESTIC_O` - Domestic outlet types (A-O)
  - `IEC_60309_2_single_16` - IEC 60309-2 single phase connector, 16A
  - `IEC_60309_2_three_16` - IEC 60309-2 three phase connector, 16A
  - `IEC_60309_2_three_32` - IEC 60309-2 three phase connector, 32A
  - `IEC_60309_2_three_64` - IEC 60309-2 three phase connector, 64A
  - `IEC_62196_T1` - IEC 62196 Type 1 connector
  - `IEC_62196_T1_COMBO` - IEC 62196 Type 1 connector with Combo 2
  - `IEC_62196_T2` - IEC 62196 Type 2 connector
  - `IEC_62196_T2_COMBO` - IEC 62196 Type 2 connector with Combo 2
  - `IEC_62196_T3A` - IEC 62196 Type 3A connector
  - `IEC_62196_T3C` - IEC 62196 Type 3C connector
  - `TESLA_R` - Tesla connector
  - `TESLA_S` - Tesla connector
  - `CCS2` - Combined Charging System Type 2

### 3. Init API (`/init`)

#### Charging Options Tags Validation
- **Rule**: Must contain at least one tag with `code: "charging-options"`
- **Path**: `message.order.items[].tags`

#### Charging Options List Validation
- **Rule**: When `code: "charging-options"`, the `list` must contain:
  - `charging-by` - Charging method (AMOUNT/TIME/UNITS/SOC)
  - `charging-limit` - Charging limit information

#### Charging By Enum Values
- **Valid Values**:
  - `AMOUNT` - Charge by amount
  - `TIME` - Charge by time
  - `UNITS` - Charge by units
  - `SOC` - Charge by State of Charge

### 4. On-Init API (`/on_init`)

#### Connector Specifications Validation
- **Rule**: Similar to on_search but for order items
- **Path**: `message.order.items[].tags`

#### Power Type Validation
- **Rule**: When `code: "power-type"`, value must be from:
  - `AC_1_PHASE` - Single phase AC
  - `AC_2_PHASE` - Two phase AC
  - `AC_2_PHASE_SPLIT` - Split two phase AC
  - `AC_3_PHASE` - Three phase AC
  - `DC` - Direct current

#### Connector Format Validation
- **Rule**: When `code: "connector-format"`, value must be:
  - `SOCKET` - Socket type connector
  - `CABLE` - Cable type connector

#### Status Validation
- **Rule**: When `code: "status"`, value must be:
  - `Available` - Connector is available
  - `Unavailable` - Connector is unavailable
  - `Reserved` - Connector is reserved

### 5. On-Confirm API (`/on_confirm`)

#### Order Status Validation
- **Rule**: Order status must be one of:
  - `COMPLETE` / `COMPLETED`
  - `ACTIVE`
  - `CANCELLED`
  - `SOFT_CANCEL`

#### Fulfillment State Validation
- **Rule**: Fulfillment state must be one of:
  - `ACTIVE`
  - `COMPLETED`
  - `INVALID`
  - `PENDING`
  - `RESERVATION`

### 6. On-Cancel API (`/on_cancel`)

#### Order Status Validation
- **Rule**: Order status must be:
  - `CANCELLED`
  - `SOFT_CANCEL`

#### Payment Validation
- **Rule**: Payment details must include:
  - `type` - Payment type
  - `status` - Payment status
  - `collected_by` - Who collects payment (BAP/BPP)
  - `params` - Payment parameters

#### Bank Details Validation
- **Rule**: Bank payment parameters must include:
  - `bank_code`
  - `bank_account_number`
  - `virtual_payment_address`

#### Transaction ID Validation
- **Rule**: For `PRE-ORDER` payments, `transaction_id` is required

## Common Validation Patterns

### 1. Contains Validation
- Used to ensure arrays contain at least one element matching specific criteria
- Example: `contains` at least one tag with `code: "connector-specifications"`

### 2. If-Then Validation
- Used for conditional validation based on field values
- Example: If `code: "connector-type"`, then `value` must be from enum

### 3. Required Fields
- Ensures mandatory fields are present in the request/response
- Applied at various levels (context, message, nested objects)

### 4. Enum Validation
- Restricts field values to predefined lists
- Ensures data consistency across implementations

## Business Logic Constraints

### 1. Location Requirements
- Country code must be `IND` (India)
- City code must be provided
- GPS coordinates must follow specific format, 6 digit accuracy


### 3. Fulfillment Requirements
- Must have at least one fulfillment
- Fulfillment must have stops with location and time information
- Customer contact information is required for fulfillment

## Implementation Notes

- All validation rules are implemented using OpenAPI 3.1.0 schema validation
- Rules use JSON Schema constructs like `allOf`, `contains`, `if-then`, and `enum`
- Validation is applied at both request and response levels
- Rules ensure backward compatibility while enforcing new constraints
- Enum values are standardized and documented for consistent implementation

## Compliance Requirements

To be compliant with this specification, implementations must:
1. Include all required fields as specified in validation rules
2. Use only the allowed enum values for constrained fields
3. Follow the conditional validation logic for dependent fields
4. Ensure data structures match the expected schemas
5. Validate both incoming requests and outgoing responses

## Future Enhancements

The validation rules can be extended to include:
- Additional connector types as they become standardized
- More granular power rating validations
- Enhanced location validation rules
- Additional payment method validations
- Real-time availability status updates 