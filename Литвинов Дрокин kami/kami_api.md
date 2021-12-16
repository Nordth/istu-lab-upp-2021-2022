# Contents
- [Contents](#contents)
- [Client](#client)
    - [Shared proto definitions](#shared-proto-definitions)
    - [Auth](#auth)
        - [Description](#description)
        - [Request proto](#request-proto)
        - [Response status](#response-status)
        - [Response proto](#response-proto)
    - [Inject](#inject)
        - [Description](#description-1)
        - [Request proto](#request-proto-1)
        - [Response status](#response-status-1)
        - [Response proto](#response-proto-1)
- [Chat](#chat)
    - [Global](#global)
        - [Description](#description-2)
    - [Personal](#personal)
        - [Description](#description-3)
- [Inventory](#inventory)
    - [Shared proto definitions](#shared-proto-definitions-1)
    - [Index](#index)
        - [Description](#description-4)
        - [Response status](#response-status-2)
        - [Response proto](#response-proto-2)
    - [Update](#update)
        - [Description](#description-5)
        - [Request proto](#request-proto-2)
        - [Response status](#response-status-3)
- [Configs](#configs)
    - [Shared proto definitions](#shared-proto-definitions-2)
    - [List](#list)
        - [Description](#description-6)
        - [Response status](#response-status-4)
        - [Response proto](#response-proto-3)
    - [Update](#update-1)
        - [Description](#description-7)
        - [Request proto](#request-proto-3)
        - [Response status](#response-status-5)
- [Payments](#payments)
    - [Generate](#generate)
        - [Description](#description-8)
        - [Request proto](#request-proto-4)
        - [Response](#response)
        - [Response status](#response-status-6)
    - [Notify](#notify)
        - [Description](#description-9)
        - [Request proto](#request-proto-5)
        - [Response status](#response-status-7)

# Client

## Shared proto definitions

```proto
message HWID {
    // MAC addreses
    repeated string mac = 1;
    // Raw SMBIOS
    bytes smb = 2;
    // GPU identifier
    repeated string gpu = 3;
    // CPU identifier
	string cpuid = 4;
    // List of volume serials
	repeated string hdd_serial = 5;
	// RAM size in MB
    uint32 ram = 6;
    // Other system information
	string system = 7;
    // Information about windows version
	string winver = 8;
    // List of installed software
	repeated string installed_software = 9;
}
```

## Auth

### Description

Loader must send hello-message to this endpoint, to authorize and recieve client information.

POST `/client/auth/`

### Request proto

```proto
message ClientHello {
	// Protocol version
	string version = 1;
    // API token	
	string token = 2;
    // Hardware identifier	
    HWID hwid = 3;
}
```

### Response status

`OK (200)` on success

`Bad request (400)` if payload is invalid

`Unauthorized (401)` if token is expired or invalid

### Response proto

```proto
message ServerHello {
    // Protocol version
	string version = 1;
    // Avatar as raw bitmap
	bytes avatar = 2;
    // Unique user identifier
	string uuid = 3;
    // Subscription end unix timestamp
	int64 subscription = 4;
    // Server response unix time
	int64 unix_timestamp = 5;
}
```

## Inject

### Description
Loader must perform request for injection to recieve shell-code from server.

POST
`/client/inject/`

### Request proto

```proto
message ClientInjectionHello {

    // Used for IAT resolving
	message ModuleInfo {
		message FunctionInfo {
			string function_name = 1;
			uint32 function_address = 2;
		}
	
        // Module name
		string module_name = 1;
        // List of functions
		repeated FunctionInfo functions = 2;
	}
	
    // Loaded modules info
	repeated ModuleInfo modules = 1;
    // API token
	string token = 2;
    // Hardware identifier
	HWID hwid = 3;
    // Address of allocated region
    // used for RVA fixes
	uint32 image_base = 4;
}
```

### Response status

`OK (200)` on success

`Bad request (400)` if payload is invalid

`Unauthorized (401)` if token is expired or invalid

### Response proto

```proto
message ServerInjectionHello {
    // Shelcode
	bytes shellcode = 1;
    // Decryption key
    bytes dec_key = 2;
    // Original entry point
    uint32 oep = 3;
}
```

# Chat

## Global

### Description

Used for global chat communication. Client should not worry about message text, this must be handled by server. Client sends and recieves messages as raw byte sequence. Client must be authorized on web-page.

WebSocket
`/chat/global/`

## Personal

### Description

Used for personal chat communication. Client should not worry about message text, this must be handled by server. Client sends and recieves messages as raw byte sequence. Client must be authorized on web-page.

WebSocket
`/chat/personal/{uuid}/`

# Inventory

## Shared proto definitions

```proto
enum ItemType {
    WEAPON = 0;
    KNIFE = 1;
    GLOVE = 2;
    MODEL = 3;
    WEAPON_CRATE = 4;
    STICKER_CAPSULE = 5;
    CRATE_KEY = 6;
    STICKER = 7;
    MUSIC_KIT = 8;
    MEDAL = 9;
    PATCH = 10;
    PAINT_KIT = 11;
    CUSTOM_NAME = 12;
    KILL_EATER = 13;
    SLOT = 14;
}

message Attribute {
    ItemType type = 1;
    uint32 uint_value = 2;
    string string_value = 3;
    bool bool_value = 4;
}

message Item {
    ItemType type = 1;
    repeated Attribute attributes = 2;
    bytes id = 3;
}
```

## Index

### Description

Used to recieve inventory as repeated `Item` list. Client must be authorized on web-page.

GET `/inventory/`

### Response status

`OK (200)` on success

`Unauthorized (401)` if token is expired or invalid

### Response proto

```proto
message InventoryData {
    repeated Item items = 1;
}
```

## Update

### Description

Used to update inventory item attributes. Client must be authorized on web-page.

POST `/inventory/update/`

### Request proto

`Item` from [shared proto definitions](#shared-proto-definitions-1)

### Response status

`OK (200)` on success

`Bad request (400)` if payload is invalid

`Unauthorized (401)` if session is expired or invalid

# Configs

## Shared proto definitions

```proto
message Config {
    message Group {
        message Variable {
            uint32 name = 1;
            repeated Variable colors = 2;
            int32 int_value = 3;
            uint32 uint_value = 4;
            float float_value = 5;
            bool bool_value = 6;
        }
        uint32 name = 1;
        repeated Group subgroups = 2;
        repeated Variable variables = 3;
    }
    repeated Group groups = 1;
    string name = 2;
    string share_code = 3;
    string author = 4;
}
```

## List

### Description

Used to retrieve user config list. Client must be authorized.

GET `/configs/list/`

### Response proto

```proto
message ConfigList {
    message Entry {
        string token = 1;
        string name = 2;
        bool active = 3;
    }
    repeated Entry configs = 1;
}
```

### Response status

`OK (200)` on success

`Unauthorized (401)` if session is expired or invalid

## Update

### Description

Used to update user config information. Client must be authorized.

GET `/configs/update/`

### Request proto

`Config` from [shared proto definitions](#shared-proto-definitions-2)

### Response status

`OK (200)` on success

`Bad request (400)` if payload is invalid

`Unauthorized (401)` if session is expired or invalid

# Payments

## Generate

### Description

Used to generate payment url using data collected from web-page form.

POST `/payments/generate/`

### Request proto

```proto
message PaymentRequest {
    string coupon = 1;
    uint32 plan = 2;
    string nickname = 3;
    bytes referer_uuid = 4;
}
```

### Response

If request succeed server will return payment url

### Response status

`OK (200)` on success

`Bad request (400)` if payload is invalid

`Unauthorized (401)` if session is expired or invalid

## Notify

### Description

Used to notify server about succesful payment.

POST `/payments/notify/`

### Request proto

```proto
message PaymentNotification {
    string nickname = 1;
    string checksum = 2;
    uint64 order_id = 3;
    string email = 4;
    uint32 unix_timestamp = 5;
    float price = 6;
}
```

### Response status

`OK (200)` on success

`Bad request (400)` if payload is invalid

`Unauthorized (401)` if session is expired or invalid