
| request path         | Request body                                                 |
| -------------------- | ------------------------------------------------------------ |
| /api/campaigns/query | {<br/>  "first": 5,<br/>  "after": 0,<br/>  "alias": "Galxe",<br/>  "credSources": ["all"],<br/>  "rewardTypes": ["all"],<br/>  "chains": ["all"],<br/>  "statuses": ["Expired"],<br/>  "listType": "created_at",<br/>  "searchString": "G"<br/>} |

---

### /api/campaigns/query

**Function Description:**  
Returns a data table of campaigns that meet specified criteria.

**Request Body:**

```go
type CampaignsQueryReqest struct {
    First        int      `json:"first"`         // Number of campaigns to return
    After        int      `json:"after"`         // Pagination cursor
    SpaceId      string   `json:"spaceId"`       // ID of the associated space
    CredSources  []string `json:"credSources"`    // Task types
    RewardTypes  []string `json:"rewardTypes"`    // Incentive types
    Chains       []string `json:"chains"`         // Chains for reward distribution
    Statuses     []string `json:"statuses"`       // Status filter
    ListType     string   `json:"listType"`       // Sorting type
    SearchString string   `json:"searchString"`   // String that campaign names must contain
}
```

**Response Format:**

```go
type CampaignsQueryResponse struct {
    PageInfo struct { 
        EndCursor   int  `json:"endCursor"`    
        HasNextPage bool `json:"hasNextPage"`
    } `json:"pageInfo"`
    Campaigns []model.Campaign          // Multiple campaign data entries
}
```

---

### /api/campaign/query

**Function Description:**  
Returns a single campaign that meets specified criteria.

**Request Body:**

```go
type CampaignQueryReqest struct {
    Id string `json:"id" binding:""` // ID of the target campaign
}
```

**Response Format:**  
Campaign data table.

---

### /api/campaign/create

**Function Description:**  
Creates a new campaign.

**Request Body:**

```go
type CampaignCreateReqest struct {
    Name                string         `json:"name"`                   // Campaign name
    RewardTypes         string         `json:"rewardTypes"`           // Incentive types
    Description         string         `json:"description"`           // Detailed description
    StartTime           int            `json:"startTime"`             // Start time
    EndTime             int            `json:"endTime"`               // End time
    LoyaltyPoints       int            `json:"loyaltyPoints"`         // Reward points
    TokenRewardContract string         `json:"tokenRewardContract"`   // Contract for token rewards
    TokenReward         datatypes.JSON `json:"tokenReward"`           // Token rewards
    SpaceID             string         `json:"space"`                 // ID of the associated space
    CredentialGroups    []CredentialGroups `json:"credentialGroups"`    // Task groups for the campaign
    DiscordRole         datatypes.JSON `json:"discordRole"`           // Discord role
    Thumbnail           string         `json:"thumbnail"`             // URL of the campaign image
    TelegramBotApi      string         `json:"telegramBotApi"`       // Telegram bot API
    TelegramChatId      string         `json:"telegramChatId"`       // ID of the Telegram chat group
}
```

**CredentialGroup Structure:**

```go
type CredentialGroup struct {
    Description      string          `json:"description"`           // Description of the task group
    Rewards          model.Rewards   `json:"rewards"`              // Rewards for the task group
    RewardAttrVals   string          `json:"rewardAttrVals"`       // Reward attribute values
    Creds            []Cred          `json:"creds"`                // Tasks within the task group
}
```

**Cred Structure:**

```go
type Cred struct {
    Description    string `json:"description"`      // Description of the task
    Name           string `json:"name"`             // Name of the task
    CredType       string `json:"credType"`         // Task type
    ReferenceLink  string `json:"referenceLink"`    // Reference link
}
```

**Response Format:**  
"SUCCESSED" OR "FAILED"

---

### /api/campaign/telegramisFollow

**Function Description:**  
Queries if the user has joined the Telegram chat to complete the task.

**Request Body:**

```go
type TelegramIsFollowRequest struct {
    CampaignId   string `json:"campaignid"`     // ID of the current campaign
    CredentialId string `json:"credentialid"`   // ID of the task currently being completed
    Username     string `json:"username"`       // User ID of the current user
}
```

**Response Format:**  
"Follow Success" OR "NO_FOLLOW"

---

### /api/campaign/isComplete

**Function Description:**  
Queries if the user has completed the campaign.

**Request Body:**

```go
type CmapaignIsCompleteRequst struct {
    CampaigId string `json:"Campaigid"`  // ID of the campaign
    Username  string `json:"username"`    // Username of the user
}
```

**Response Format:**  
"COMPLETE" OR "NOT_COMPLETE"

---

### /api/campaign/isCredentialComplete

**Function Description:**  
Queries if the user has completed the specified task.

**Request Body:**

```go
type IsCredentialCompleteRequst struct {
    CredentialId string `json:"credentialid"` // Task ID
    Username     string `json:"username"`     // Username of the user
}
```

**Response Format:**  
"COMPLETE" OR "NOT_COMPLETE"
