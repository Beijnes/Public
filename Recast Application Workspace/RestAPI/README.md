# Liquit REST API Workflow

## OAuth2 Authentication & Zone License Query Flow

```mermaid
%%{init: {'flowchart': {'curve': 'linear', 'useMaxWidth': false}, 'theme': 'base', 'primaryColor': '#667eea', 'primaryBorderColor': '#5568d3', 'lineColor': '#60a5fa', 'textColor': '#e5e7eb', 'fontSize': '16px', 'themeVariables': {'edgeLabelBackground': '#111827', 'textColor': '#e5e7eb'}}}%%
flowchart TD
    A["🔧 START: Configuration"] --> B["📋 Credentials Setup<br/>Server, ClientID, Username, Password"]
    
    B --> D["🔐 AUTHENTICATION STEP"]
    
    D --> E["POST /api/oauth2/token<br/>grant_type: password<br/>scope: idtoken content"]
    E --> F{✅ Token OK?}
    F -->|YES| G["🎫 Extract Bearer Token"]
    F -->|NO| H["❌ AUTH FAILED"]
    H --> Z["🛑 ERROR EXIT"]
    
    G --> I["🔑 Create Authorization Header<br/>Authorization: Bearer {token}"]
    I --> J["⏱️ Generate Cache Buster<br/>Timestamp = Unix Milliseconds"]
    
    J --> K["📊 QUERY ALL ZONES"]
        
    K --> O["Loop all the zones"]
    
    O --> P["🆔 Extract: Zone ID & Name"]
    
    P --> R["📃 Get License Object"]
    
        R --> T["➕ Append to Results"]
    
    T --> U{More Zones?}
    U -->|YES| O
    U -->|NO| W["📤 RETURN allLicenses"]
    
    
    classDef start fill:#667eea,stroke:#5568d3,stroke-width:3px,color:#fff
    classDef auth fill:#f39c12,stroke:#e67e22,stroke-width:3px,color:#fff
    classDef query fill:#16a085,stroke:#138d75,stroke-width:3px,color:#fff
    classDef loop fill:#8e44ad,stroke:#7d3c98,stroke-width:3px,color:#fff
    classDef success fill:#27ae60,stroke:#1e8449,stroke-width:3px,color:#fff
    classDef error fill:#e74c3c,stroke:#c0392b,stroke-width:3px,color:#fff
    classDef process fill:#3498db,stroke:#2980b9,stroke-width:2px,color:#fff
    
    class A start
    class D,E,F,G,I auth
    class K,L,M query
    class N,O,P,Q,R,S,T,U loop
    class V,W,X success
    class H,Z error
    class B,C,J process

    linkStyle default stroke:#60a5fa,stroke-width:2.5px,color:#e5e7eb
```

## Key API Concepts Visualized

### 🔑 Authentication Flow
```
┌─────────────────────────────────────────────┐
│   User Credentials                          │
│   ├─ username: local\admin                  │
│   ├─ password: ****                         │
│   └─ client_id: 74AAE62C-58BE-...          │
└────────────┬────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────┐
│   OAuth2 Token Endpoint                     │
│   POST /api/oauth2/token                    │
│   ├─ grant_type: password                   │
│   └─ scope: idtoken content                 │
└────────────┬────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────┐
│   Access Token Received ✅                   │
│   token_type: Bearer                        │
│   expires_in: 3600                          │
│   access_token: eyJ0eXAi...                │
└────────────┬────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────┐
│   Authorization Header                      │
│   Authorization: Bearer eyJ0eXAi...        │
│   (Used for all API calls)                  │
└─────────────────────────────────────────────┘
```

### 🔍 OData Query Parameters
| Parameter | Purpose | Example |
|-----------|---------|---------|
| **$count=true** | Include total count in response | Enables pagination info |
| **$skip=0** | Pagination: Skip N records | Skip first 0 records |
| **$top=50** | Pagination: Return max N records | Return max 50 per request |
| **$orderby=name** | Sort results | Sort by name ascending |
| **$select=id,name,...** | Select specific fields | Reduces response payload |
| **_=timestamp** | Cache buster | Force fresh data each call |

### 🔄 Loop Through All Zones
```
FOR EACH zone:
  ├─ Extract Zone ID & Name
  ├─ GET /api/v3/system/zones/{zoneId}/?$select=id,license
  ├─ Retrieve License Object
  └─ Append to Results Array
RETURN allLicenses
```

## 📦 Data Structure Flow

### Step 1: Authentication Response
```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "idtoken content"
}
```

### Step 2: Query Zones (Simplified)
```json
{
  "@odata.count": 15,
  "value": [
    {
      "id": "ee840dbe-df14-6ef9-0002-375cae35752b",
      "name": "Production Zone",
      "enabled": true,
      "primary": true,
      "license/state": "Active"
    },
    ...
  ]
}
```

### Step 3: Enriched Results (Loop Through Zones)
```powershell
@(
  @{
    ZoneId   = "ee840dbe-df14-6ef9-0002-375cae35752b"
    ZoneName = "Production Zone"
    License  = @{
      state   = "Active"
      expires = "2025-12-31"
      ...
    }
  },
  @{
    ZoneId   = "12345678-abcd-ef12-3456-789abcdef012"
    ZoneName = "Staging Zone"
    License  = @{
      state   = "Active"
      expires = "2025-06-30"
      ...
    }
  }
)
```

## 💻 PowerShell Usage Examples

### Run Script & Capture Results
```powershell
# Execute script and store results
$licenses = .\restapi-blog.ps1

# View all license data
$licenses | Format-Table -AutoSize

# Export to CSV
$licenses | Select-Object ZoneName, License | Export-Csv zones-licenses.csv -NoTypeInformation

# Filter specific zones
$licenses | Where-Object { $_.ZoneName -like '*Production*' }

# Check license expiration
$licenses | Select-Object ZoneName, @{Name='Expires'; Expression={$_.License.expires}} | Sort-Object Expires
```

### Process Results Directly
```powershell
# Store and process in one line
$licenses = .\restapi-blog.ps1 | ForEach-Object {
    Write-Host "Zone: $($_.ZoneName) | License: $($_.License.state)"
    $_
}
```

### Error Handling
```powershell
try {
    $data = .\restapi-blog.ps1
    Write-Host "✅ Successfully retrieved $($data.Count) zones" -ForegroundColor Green
}
catch {
    Write-Error "❌ Failed to fetch zones: $_"
}
```
