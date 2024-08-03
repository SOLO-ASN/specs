
| request path       | Request body                                                 |
| ------------------ | ------------------------------------------------------------ |
| /api/explore/query | {<br/>    "first": 4,                // Number of data items to load for the first time<br/>    "after": 0,                // How much data has been loaded starting from 20<br/>    "credSources": ["all"],<br/>    "rewardTypes": ["all"],<br/>    "chains": ["all"],<br/>    "statuses": ["all"],<br/>    "listType": "participantsCount",<br/>    "searchString": "G"<br/>  } |
| /api/spaces/query  | {<br/>    "first": 2,<br/>    "after": 0,<br/>    "filters": [" "],<br/>    "username": "mark12355",<br/>    "searchString": "G",<br/>    "spaceListType": "created_at",<br/>    "verifiedOnly": false<br/>  } |

### `/api/explore/query`
**Function Description**  
Returns the Campaign data table that meets the request conditions.

**Request Body**
```go
type ExploreQueryRequest struct {
    First        int      `json:"first"`       // Start returning from this index
    After        int      `json:"after"`       // Number of records to return
    CredSources  []string `json:"credSources"`  // Task types
    RewardTypes  []string `json:"rewardTypes"`  // Reward types
    Chains       []string `json:"chains"`       // Corresponding blockchain platforms
    Statuses     []string `json:"statuses"`     // Campaign statuses
    ListType     string   `json:"listType"`     // Sort by this type
    SearchString string   `json:"searchString"` // Name contains this string
}
```

**Response Format**
```go
type ExploreQueryResponse struct {
    PageInfo struct {
        EndCursor   int  `json:"endCursor"`     // Position in the data table that has been read
        HasNextPage bool `json:"hasNextPage"`   // Indicates if there is remaining data that hasn't been returned
    } `json:"pageInfo"`
    Explore []ExploreData `json:"explore"` // as defined below            
}

type ExploreData struct {
    Campaign model.Campaign `json:"campaign"` // Campaign data table
    Space    model.Space    `json:"space"`    // Space data table
}
```

---

### `/api/space/query`
**Function Description**  
Returns the space data table that meets the conditions.

**Request Body**
```go
type SpaceQueryRequest struct {
    Id       string `json:"id"`       // ID of the space data table
    Username string `json:"username"` // Username of the logged-in Solomission user
}
```

**Response Format**  
Returns the entire space data table directly.

---

### `/api/spaces/query`
**Function Description**  
Returns all space data tables that meet the conditions.

**Request Body**
```go
type SpacesQueryRequest struct {
    First         int    `json:"first"`            // Start returning from this index
    After         int    `json:"after"`            // Number of records to return
    Filter        string `json:"filter"`           // Filtering condition
    Username      string `json:"username"`         // Username of the logged-in Solomission user
    SearchString  string `json:"searchString"`     // Space name contains this string
    SpaceListType string `json:"spaceListType"`    // Sorting type
    VerifiedOnly  bool   `json:"verifiedOnly"`     // Whether to return only platform-verified spaces
}
```

**Response Format**
```go
type SpacesQueryResponse struct {
    PageInfo struct {
        EndCursor   int  `json:"endCursor"`    // Position in the data table that has been read
        HasNextPage bool `json:"hasNextPage"`  // Indicates if there is remaining data that hasn't been returned
    } `json:"pageInfo"`
    Spaces []model.Space `json:"spaces"` // Multiple space data tables
}
```

---

### `/api/spaces/create`
**Function Description**  
Creates a new space.

**Request Body**
```go
type SpaceCreateRequest struct {
    Username  string         `json:"username"`  // Username of the user creating the space
    Name      string         `json:"name"`      // Space name
    Thumbnail string         `json:"thumbnail"` // URL where the space icon is stored
    Links     datatypes.JSON `json:"links"`     // Link to the official project webpage
    Alias     string         `json:"alias"`     // Space alias
    Categories datatypes.JSON `json:"categories"` // Space categories
    Info      string         `json:"info"`      // Information about the space
}
```

**Response Format**  
`"SUCCESSED"` OR `"FAILED"`

---

### `/api/space/follow`
**Function Description**  
User follows the target space, inserting a new row into the database's spaceFollower table.

**Request Body**
```go
type FollowRequest struct {
    SpaceId  string `json:"spaceid"`  // Space ID
    Username string `json:"username"`  // Username
}
```

**Response Format**  
`"SUCCESSED"` OR `"FAILED"`

---

### `/api/space/unfollow`
**Function Description**  
User unfollows the target space, modifying the status in the database's spaceFollower table.

**Request Body**
```go
type FollowRequest struct {
    SpaceId  string `json:"spaceid"`  // Space ID
    Username string `json:"username"`  // Username
}
```

**Response Format**  
`"SUCCESSED"` OR `"FAILED"`
