# API Design Part 2

## Introduction

While researching the topic of HATEOS I found that there are conflicting views about how to include hypermedia in JSON. A common appraoch I saw was to split a JSON response into two sections. First include all of the necessary data associated with the requested entity which is not hypermedia, and then include a list, often named "links", which includes both the url of related entities and also to API endpoints that allow the user to manipulate the data.

On the other hand, Martin Nally recommends a different approach. He recommends that a json response represents an entity in the applications data model and that every field of that json response should be an attribute or relationship of that entity. According to this rule their is no attribute named "links". Instead, hypermedia is integrated with the rest of the fields as direct values of an entity's attributes.

I was conflicted about which approach to take in my design. On the one hand, the first approach seems to be more explicit and better suited to interact with programmatic consumers of the API. However the second approach seems simpler and I believe fits more closely with RESTful principles.

I've decided to use a combination of the two approaches. All of the objects attributes and relationships are either simple data types or urls that point to other entities. In addition, each response has a list of Actions. The `Action` data type holds relevant information for accessing an API endpoint for manipulating the data object. Including these links helps to decouple the users of the API from the API itself because most of the methods of manipulating data are dynaimcially provided to users. Further, these actions can be filtered based on the permissions held by the requesting user. However, using a JSON API for POST and PUT requests still requires users to understand how to format the request body, and It is unclear to me how this coupling can be removed in a JSON API.

## API Design

Below are the object structures that will be returned from each API. I've included example constructors that show what an instance of these objects might look like.

```go
type Action struct {
    rel    string
    method string
    href   string
}
```

Tha `Action` object is used in all three APIs and includes the hypermedia for performing a create, update or delete action on application data.

### Poll API

`pollOption`:

```go
type pollOption struct{
    PollOptionID    uint
    PollOptionText  string
    Self            string //link to poll option
    Actions         []Action
}

```

Example `pollOption` constructor:

```go
func NewPollOption() *PollOption {
    return &PollOption{
            PollOptionID:     1,
            PollOptionText:  "Cat",
            Self:            "/polls/poll-option/1",
            Actions: []Action{
                {rel: "create", method: "POST", href: "/polls/poll-option/"},
                {rel: "update", method: "PUT", href: "/polls/poll-option/1"},
                {rel: "delete", method: "DELETE", href: "/polls/poll-option/1"},
            }
        }
}

```

`Poll`:

```go
type Poll struct {
    PollID        uint
    PollTitle     string
    PollQuestion  string
    self          string//link to poll
    PollOptions   []pollOption
    Actions       []Action
}
```

Example `Poll` constructor:

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
            },
            Actions: []Action{
                {rel: "create", method: "POST", href: "/polls"},
                {rel: "update", method: "PUT", href: "/polls/1"},
                {rel: "delete", method: "DELETE", href: "/polls/1"},
            }
    }
}

```

### Voter API

`voterPoll`:

```go
type voterPoll struct {
	PollID   uint
	VoteDate time.Time
    Self     string      //link to voterpoll
    Actions  []Actions
}

```

Example `VoterPoll` constructor:

```go
func NewVoterPoll() *VoterPoll {
    return &VoterPoll{
        PollID: 1,
        VoteDate: 2024-05-29 17:46:47.069217048,
        Self: "/voters/1/polls/1"
        Actions: []Action{
            {rel: "create", method: "POST", href: "/voters/1/polls"},
            {rel: "update", method: "PUT", href: "/voters/1/polls/1"},
            {rel: "delete", method: "DELETE", href: "/voters/1/polls/1"},
        }
    }
}
```

`Voter`:

```go

type Voter struct {
	VoterID     uint
	FirstName   string
	LastName    string
    Self        string   //link to voter
	VoteHistory []string //links to voterpolls
    Actions     []Actions
}

```

Example `Voter` constructor:

```go
func NewVoter() *Voter {
    return &Voter{
        VoterID: 1,
        FirstName: "Michael",
        LastName: "Jones",
        Self: "/voters/1",
        VoteHistory: []VoterPoll{
            {PollID: 1, VoteDate: 2024-05-29 17:46:47.069217048, Self: "/voters/1/polls/1"},
            {PollID: 3, VoteDate: 2024-05-29 17:46:47.069217048, Self: "/voters/1/polls/3"},
            {PollID: 12, VoteDate: 2024-05-29 17:46:47.069217048, Self: "/voters/1/polls/12"},
        },
        Actions: []Action{
            {rel: "create", method: "POST", href: "/voters"},
            {rel: "update", method: "PUT", href: "/voters/1"},
            {rel: "delete", method: "DELETE", href: "/voters/1"},
        }
    }
}
```

### Votes API

`Vote`:

```go
type Vote struct{
    VoteID     uint
    Self       string //link to vote
    Voter      string //link to voter
    Poll       string //link to poll
    VoteValue  string //link to vote value
    Actions    []Actions

}
```

Example `Vote` constructor:

```go
func NewVote() *Vote {
    return &Vote{
        VoteID: 1,
        Self: "/votes/1",
        Voter: "/voters/1",
        Poll: "/polls/1",
        VoteValue: "/polls/poll-option/1",
        Actions: []Action{
            {rel: "create", method: "POST", href: "/votes"},
            {rel: "update", method: "PUT", href: "/votes/1"},
            {rel: "delete", method: "DELETE", href: "/votes/1"},
        }
    }
}
```

## API Description

By including hypermedia in the application design, these APIs become more flexible and accessible to various types of users. Firstly, a user can easily follow links to discover more information about a resource. For example, if a user is retrieving a voter resource, they can follow the hypermedia in its voter history to get more information about the relevant polls. They do not need to understand the structure of the polls api and the necessary endpoints to get this information. Further all actions availble to each user are clearly provided in the actions array. These are filtered to only show actions that a user has access to based on their permissions. For example, the delete action for polls would only be shown to the user who created the poll or users who have administrative access. This allows users to decide how they wish to use the API based on the actions and resources discoverable to them.

While this design may not be ideal for users who hope to build the most fast and efficient bridge between two applications, it provides a better user experience to users who are interactivly engaging with the API and protects users from changes to the API structure.

The main functionality of the API does not change with the use of HATEOAS, but now each service will need to know how to correctly build urls for the resources it depends on and provide this hypermedia to the user.

## References:

"Terrifically Simple JSON"
Martin Nally https://github.com/mpnally/Terrifically-Simple-JSON

Spring REST Chapter 10 HATEOAS
[1] B. Varanasi and S. Belida, Spring REST. Place of publication not identified: Apress, 2015.
