# API Design Part 2

Poll API

```go
type pollOption struct{
    PollID          uint
    PollOptionText: string
    Self:           string //link to poll option
}

type Poll struct {
    PollID:       uint
    PollTitle:    string
    PollQuestion: string
    self:         string//link to poll
    PollOptions:  []string //links to poll options
}
```

Voter API

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

Votes API

```go
type Vote struct{
    VoteID     uint
    Self       string //link to vote
    Voter      string //link to voter
    Poll       string //link to poll
    VoteValue  string //link to vote value
}
```
