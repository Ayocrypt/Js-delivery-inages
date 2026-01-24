# MindBody API Class - PHP Documentation

## Overview

The `HW_Mindbody_API` class provides a comprehensive OOP interface for the MindBody API v6 in WordPress. It includes a unified function that combines multiple API endpoints into a single call, returning all booking data needed for appointments: availability, pricing, locations, staff information, and available booking times.

## Key Features

- ✅ **Unified Booking Data** - Get all booking information in one call with `get_unified_bookable_slots()`
- ✅ **Smart Product Matching** - Automatically matches products to sessions by staff and treatment name
- ✅ **Active Session Times** - Returns available booking time slots within each availability window
- ✅ **Efficient API Calls** - Optimized to make minimal API requests (2-3 calls total)
- ✅ **Booked Appointment Handling** - BookableItems API automatically excludes booked times
- ✅ **Token Management** - Automatic authorization token generation
- ✅ **WordPress Integration** - Uses WordPress options and error handling

## Table of Contents

1. [Installation & Setup](#installation--setup)
2. [Quick Start](#quick-start)
3. [Core Methods](#core-methods)
4. [Unified Function](#unified-function)
5. [How It Works](#how-it-works)
6. [Active Session Times Explained](#active-session-times-explained)
7. [Product Matching Logic](#product-matching-logic)
8. [Response Structure](#response-structure)
9. [Examples](#examples)
10. [Error Handling](#error-handling)

---

## Installation & Setup

### 1. Configure WordPress Options

The class reads credentials from WordPress options. Set these in your WordPress admin or `wp-config.php`:

```php
update_option( 'mindbody_api_key', 'your-api-key' );
update_option( 'mindbody_site_id', 'your-site-id' );
update_option( 'mindbody_username', 'your-username' );  // For token generation
update_option( 'mindbody_password', 'your-password' );  // For token generation
```

### 2. Include the Class

The class is already included in the theme. Use the helper function:

```php
$api = hw_mindbody_api();
```

---

## Quick Start

### Basic Usage

```php
// Get API instance
$api = hw_mindbody_api();

// Check if configured
if ( ! $api->is_configured() ) {
    echo 'API not configured';
    return;
}

// Get unified bookable slots
$result = $api->get_unified_bookable_slots( array(
    'ProgramId'   => 2,  // Massage & Bodywork
    'StartDate'   => '2026-01-29T00:00:00Z',
    'EndDate'     => '2026-01-30T23:59:59Z',
    'LocationIds' => array( 1 ),
) );

if ( is_wp_error( $result ) ) {
    echo 'Error: ' . $result->get_error_message();
} else {
    $slots = $result['slots'];
    echo 'Found ' . count( $slots ) . ' available slots';
}
```

### Simple Example: Get Available Slots

```php
$api = hw_mindbody_api();

$slots = $api->get_unified_bookable_slots( array(
    'SessionTypeIds' => array( 23, 24 ),  // Deep tissue massage, Acupuncture
    'StartDate'      => gmdate( 'Y-m-d\TH:i:s\Z' ),
    'EndDate'        => gmdate( 'Y-m-d\TH:i:s\Z', strtotime( '+7 days' ) ),
) );

if ( ! is_wp_error( $slots ) ) {
    foreach ( $slots['slots'] as $slot ) {
        echo $slot['staffName'] . ' - ' . $slot['mainName'] . "\n";
        echo 'Time: ' . $slot['startDateTime'] . ' to ' . $slot['endDateTime'] . "\n";
        echo 'Products: ' . count( $slot['products'] ) . "\n";
        echo 'Active Times: ' . implode( ', ', $slot['activeTimes'] ) . "\n\n";
    }
}
```

---

## Core Methods

### Authentication

#### `get_token( $username = null, $password = null )`

Get authorization token from MindBody API.

```php
$token = $api->get_token();
if ( is_wp_error( $token ) ) {
    // Handle error
}
```

### Basic Endpoints

#### `get_programs( $args = array(), $auth_token = null )`

Get programs available at the site.

```php
$programs = $api->get_programs( array(
    'ScheduleType' => 'Appointment',
    'OnlineOnly'   => true,
    'Limit'         => 100,
) );
```

#### `get_session_types( $args = array(), $auth_token = null )`

Get session types (appointment types).

```php
$session_types = $api->get_session_types( array(
    'ProgramIDs' => array( 2, 8 ),
    'OnlineOnly'  => true,
) );
```

#### `get_services( $args = array(), $auth_token = null )`

Get services/products with pricing.

```php
$services = $api->get_services( array(
    'SessionTypeIds' => array( 23, 24 ),
    'SellOnline'     => true,
) );
```

#### `get_bookable_items( $args = array(), $auth_token = null )`

Get available appointment slots.

```php
$bookable = $api->get_bookable_items( array(
    'SessionTypeIds' => array( 23 ),
    'StartDate'       => '2026-01-29T00:00:00Z',
    'EndDate'         => '2026-01-30T23:59:59Z',
) );
```

#### `get_active_session_times( $args = array(), $auth_token = null )`

Get business hours and booking increments.

```php
$active_times = $api->get_active_session_times( array(
    'SessionTypeIds' => array( 23, 24 ),
    'ScheduleType'   => 'Appointment',
) );
```

#### `get_staff( $args = array() )`

Get staff members.

```php
$staff = $api->get_staff( array(
    'Limit' => 100,
) );
```

#### `get_locations()`

Get locations.

```php
$locations = $api->get_locations();
```

---

## Unified Function

### `get_unified_bookable_slots( $args = array(), $auth_token = null )`

**The main function** - Combines all booking data into one call.

#### Parameters

```php
array(
    'SessionTypeIds' => array( 23, 24 ),  // Optional if ProgramId provided
    'ProgramId'       => 2,                // Optional - gets session types automatically
    'StartDate'      => '2026-01-29T00:00:00Z',
    'EndDate'        => '2026-01-30T23:59:59Z',
    'LocationIds'    => array( 1 ),       // Optional
    'StaffIds'       => array( 68273 ),   // Optional
    'Limit'           => 100,
    'Offset'          => 0,
)
```

#### Returns

```php
array(
    'slots'      => array( ... ),  // List of unified slot dictionaries
    'total'      => 15,            // Total number of slots
    'pagination' => array( ... ),  // Pagination info from API
    'metadata'   => array( ... ),  // Metadata about the query
)
```

#### Example Slot Structure

```php
array(
    'sessionTypeId'       => 23,
    'staffId'            => 68273,
    'everyMins'          => 30,
    'startDateTime'      => '2026-01-29T17:00:00',
    'endDateTime'        => '2026-01-29T17:30:00',
    'bookableEndDateTime' => '2026-01-29T17:25:00',
    'activeTimes'        => array( '17:00:00', '17:30:00', '18:00:00' ),
    'locationId'         => 1,
    'locationName'       => 'HOME',
    'locationAddress'    => 'Primrose Hill Courtyard 7 Erskine Road, London',
    'staffName'          => 'Katia Narain Phillips',
    'staffBio'           => '<div>Katia is a bestselling co-author...</div>',
    'mainName'           => 'Deep tissue massage - Katia Phillips - 60mins',
    'durations'          => '60',
    'bookableOnline'    => true,
    'displayName'       => 'deep tissue massage',
    'description'       => '',
    'products'          => array(
        array(
            'id'          => '100094',
            'productId'   => '100094',
            'name'        => 'Deep tissue Massage - Katia Phillips - 90min',
            'price'       => 150.0,
            'onlinePrice' => 150.0,
            'count'       => 1,
            'sellOnline'  => true,
            'duration'    => 60,
        ),
    ),
    'id'                => '23-68273-1-20260129-1700',
)
```

---

## How It Works

The `get_unified_bookable_slots()` function combines data from multiple MindBody API endpoints:

### Step-by-Step Process

1. **Get Bookable Items** (`/appointment/bookableitems`)
   - Returns available appointment slots
   - Already excludes booked appointments
   - Includes: staff, location, session type, time windows

2. **Get Active Session Times** (`/appointment/activesessiontimes`)
   - Returns business hours and booking increments (e.g., every 30 minutes)
   - **ONE call** for all session types

3. **Get Products/Services** (`/sale/services`)
   - Returns pricing information
   - **ONE call** filtered by session type IDs
   - Matched to slots by staff name and treatment name

4. **Build Unified Slots**
   - Combines all data into a single structure
   - Filters active times by each slot's time window
   - Matches products to exact staff and session type

---

## Active Session Times Explained

### What Are Active Session Times?

Active session times are the **possible booking increments** that a business allows. For example:
- Every 30 minutes: `["09:00:00", "09:30:00", "10:00:00", "10:30:00", ...]`
- Every hour: `["09:00:00", "10:00:00", "11:00:00", ...]`

**Important**: These are NOT availability - they're just the time structure. Actual availability comes from BookableItems.

### How We Use Active Session Times

1. **Fetch Once**: Get all active times for all session types in ONE API call
2. **Filter Per Slot**: For each available slot, filter active times to only those within the slot's window
3. **Result**: Each slot gets only the booking times that fall within its availability window

### Example

```
Slot: 4pm-9pm available
Active Times: ["06:00:00", "07:00:00", ..., "21:00:00"]

Filtered for this slot: ["16:00:00", "17:00:00", "18:00:00", "19:00:00", "20:00:00", "21:00:00"]
```

---

## Product Matching Logic

Products are matched to slots using **exact matching**:

1. **Service name must contain the main treatment name**
   - Session: "Deep tissue massage - Katia Phillips - 60mins"
   - Main treatment: "Deep tissue massage"
   - Service must contain "deep tissue massage"

2. **Service name must contain the staff name**
   - Staff: "Katia Narain Phillips"
   - Service must contain "Katia" or "Phillips" or full name

3. **Both conditions must be true** for a match

This prevents incorrect matches like "Acupuncture" matching "Deep tissue massage".

---

## Response Structure

### Top-Level Response

```php
array(
    'slots'      => array( ... ),  // List of slots
    'total'      => 15,            // Total count
    'pagination' => array(         // Pagination info
        'RequestedLimit'  => 100,
        'RequestedOffset' => 0,
        'PageSize'        => 15,
        'TotalResults'    => 15,
    ),
    'metadata'   => array(         // Query metadata
        'programId'      => 2,
        'sessionTypeIds' => array( 23, 24 ),
        'locationIds'    => array( 1 ),
        'staffIds'       => array( 68273 ),
        'dateRange'      => array(
            'startDate' => '2026-01-29T00:00:00Z',
            'endDate'   => '2026-01-30T23:59:59Z',
        ),
    ),
)
```

---

## Examples

### Example 1: Get Slots by Program

```php
$api = hw_mindbody_api();

$result = $api->get_unified_bookable_slots( array(
    'ProgramId'   => 2,  // Massage & Bodywork
    'StartDate'   => gmdate( 'Y-m-d\TH:i:s\Z' ),
    'EndDate'     => gmdate( 'Y-m-d\TH:i:s\Z', strtotime( '+7 days' ) ),
) );

if ( ! is_wp_error( $result ) ) {
    foreach ( $result['slots'] as $slot ) {
        echo sprintf(
            "%s - %s\nTime: %s to %s\nPrice: £%.2f\n\n",
            $slot['staffName'],
            $slot['mainName'],
            $slot['startDateTime'],
            $slot['endDateTime'],
            $slot['products'][0]['onlinePrice'] ?? 0
        );
    }
}
```

### Example 2: Get Slots for Specific Staff

```php
$api = hw_mindbody_api();

$result = $api->get_unified_bookable_slots( array(
    'SessionTypeIds' => array( 23 ),
    'StaffIds'       => array( 68273 ),
    'StartDate'      => '2026-01-29T00:00:00Z',
    'EndDate'        => '2026-01-30T23:59:59Z',
) );
```

### Example 3: Get Active Times Only

```php
$api = hw_mindbody_api();
$token = $api->get_token();

$active_times = $api->get_active_session_times( array(
    'SessionTypeIds' => array( 23 ),
    'ScheduleType'   => 'Appointment',
), $token );

if ( ! is_wp_error( $active_times ) ) {
    $times = $active_times['ActiveSessionTimes'];
    echo 'Available booking times: ' . implode( ', ', $times );
}
```

---

## Error Handling

All methods return either:
- **Success**: Array with data
- **Error**: `WP_Error` object

### Check for Errors

```php
$result = $api->get_unified_bookable_slots( $args );

if ( is_wp_error( $result ) ) {
    $error_code = $result->get_error_code();
    $error_message = $result->get_error_message();
    
    // Handle error
    error_log( "MindBody API Error: {$error_code} - {$error_message}" );
    return;
}

// Use result
$slots = $result['slots'];
```

### Common Errors

- `missing_credentials` - API not configured
- `token_error` - Failed to get authorization token
- `api_error` - API returned error response
- `missing_params` - Required parameters missing

---

## Performance

The unified function is optimized for efficiency:

- **3 API calls total** (regardless of number of slots)
- **Single call** for all active session times
- **Single call** for all products/services
- **Cached results** used during slot building

This is much more efficient than making individual API calls for each slot.

---

## Notes

- The `BookableItems` API already excludes booked appointments, so all returned slots are available
- Active session times are filtered per slot to show only times within the availability window
- Product matching is strict to prevent incorrect matches
- All methods support optional `$auth_token` parameter (auto-generated if not provided)
- Query parameters use `request.` prefix (e.g., `request.SessionTypeIds=23`)

---

## Support

For issues or questions, check:
- MindBody API Documentation: https://developers.mindbodyonline.com/
- Python Reference: `pythonPostman/README.md`
- Test Script: `test-mindbody-api.php`
