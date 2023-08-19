# API Design Part 2

While researching the topic of HATEOS I found that there are conflicting views about how to include hypermedia in JSON. A common appraoch I saw was to split a JSON response into two sections. First include all of the necessary data associated with the requested entity which is not hypermedia, and then include a list, often named "links", which includes both the url of related entities and also to API endpoints that allow the user to manipulate the data.

On the other hand, Martin Nally recommends a different approach. He recommends that a json response represents an entity in the applications data model and that every field of that json response should be an attribute or relationship of that entity. According to this rule their is no attribute named "links". Instead, hypermedia is integrated with the rest of the fields as direct values of an entity's attributes.

I was conflicted about which approach to take in my design. On the one hand, the first approach seems to be more explicit and better suited to a interact with programmatic consumers of the API. However the second approach seems simpler and I believe fits more closely with RESTful principles.

I've decided to use a combination of the two approaches. All of the objects attributes and relationships are either simple data types or urls that point to other entities. In addition, each response has a list of Actions. The `Action` data type holds relevant information for accessing an API endpoints for manipulating the data object. I think including these links helps to decouple the users of the API from the API itself because most of the methods of manipulating data are dynaimcially provided to users. Further, these actions can be filtered based on the permissions held by the requesting user. However, using a JSON API for POST and PUT requests still requires users to understand how to format the request body, and It is unclearmn to me how this coupling can be removed in a JSON API.

Below are the object structures that will be returned from each API. I've included example constructors that show what an instance of these objects might look like.

```go
type Action struct {
    rel    string
    method string
    href   string
}
```

Tha `Action` object is used in all three APIs and includes the hypermedia for performing a create, update or delete action on application data.

## Poll API

```go
type pollOption struct{
    PollOptionID    uint
    PollOptionText  string
    Self            string //link to poll option
}

```

Example pollOption constructor:

```go
func NewPollOption() *PollOption {
    return &PollOption{
            PollOptionID:     1,
            PollOptionText:  "Cat",
            Self:            "/polls/poll-option/1",
        }
}

```

```go


type Poll struct {
    PollID        uint
    PollTitle     string
    PollQuestion  string
    self          string//link to poll
    PollOptions   []pollOption
}
```

Example Poll constructor:

```go
func NewPollOption() *PollOption {
    return &Poll{
            PollID:        1,
            PollTitle:     "Favorite Animal",
            PollQuestion:  "What is your favorite animal?",
            self:          "/polls/1"
            PollOptions:   []pollOption{
                {PollOptionID: 1, PollOptionText: "Cat", Self: "/polls/poll-option/1",},
                {PollOptionID: 2, PollOptionText: "Dog", Self: "/polls/poll-option/2",},
                {PollOptionID: 3, PollOptionText: "Fish", Self: "/polls/poll-option/3",},
        }
    }
}

```

## Voter API

```go
type voterPoll struct {
	PollID   uint
	VoteDate time.Time
    self     string      //link to voterpoll
}

type Voter struct {
	VoterID     uint
	FirstName   string
	LastName    string
    self        string   //link to voter
	VoteHistory []string //links to voterpolls
}
```

## Votes API

```go
type Vote struct{
    VoteID     uint
    Self       string //link to vote
    Voter      string //link to voter
    Poll       string //link to poll
    VoteValue  string //link to vote value
}
```

References:

Martin Nally "Terrifically Simple JSON" https://github.com/mpnally/Terrifically-Simple-JSON
