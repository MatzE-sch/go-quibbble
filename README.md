# Go-quibbble

Go-quibbble contains the backend game service for all Quibbble games. This provides a simple REST API for creating and retriving game data as well as a websocket messaging interface to play these games. Any game that implements the [go-boardgame](https://github.com/quibbble/go-boardgame) template can easily be added to take advantage of this networking layer. 

## Status

Go-quibbble is a functional work in progress with the following futures still to be implemented:
- Secure games: games that require some form of authentication to join/play allowing for anonymous competitive play.

## Add Your Game

1. Create a game that implement the [go-boardgame](https://github.com/quibbble/go-boardgame) template. See [Tic-Tac-Toe](https://github.com/quibbble/go-boardgame/examples/tictactoe) for an example.
2. Add your game's game key to `/configs/quibbble.yaml` under `Network`>`Games`.
3. Import the game code and add the builder to `/internal/server/games.go` in the `games` map.
4. (Optional) If your game has adapters import the adapter code and add it to `internal/server/adapters` in the `adapters` map.
5. Your game has been added to the quibbble game service. Start the service and play your game!

## Build and Run

### Go
```
$ go build -o quibbble cmd/main.go
$ ./quibbble
```

### Docker
```
$ docker build --tag quibbble/quibbble:${TAG} -f build/Dockerfile .
$ docker run -d --name quibbble -p 8080:8080 quibbble/quibbble:${TAG}
```

## REST API

### Create Game

```
curl --request POST 'http://localhost:8080/game/create' \
--header 'Content-Type: application/json' \
--data-raw '{
    "GameKey": "TicTacToe", // the name of the game
    "GameID": "example",    // the unique instance id
    "Teams": 2,             // the number of players
    "TurnLength": "60s",    // max time per turn, null for no timer
    "SingleDevice": false,  // play on one device or multiple
    "MoreOptions": {}       // additional options unique to the game
}'
```

### Load Game

```
curl --request POST 'http://localhost:8080/game/load' \
--header 'Content-Type: application/json' \
--data-raw '{
    "GameKey": "Tic-Tac-Toe",                                                // the name of the game
    "GameID": "example",                                                     // the unique instance id
    "BGN": "[Teams \"red, blue\"][Game \"Tic-Tac-Toe\"]0m&0.0 1m&0.1 0m&1.1" // board game notation
}'
```

### Get BGN

```
curl 'http://localhost:8080/game/bgn?GameKey=Tic-Tac-Toe&GameID=example'
```

### Health Check

```
curl 'http://localhost:8080/health'
```

## Websocket Messaging

### Join Game

#### Request

```
ws://localhost:8080/game/join?GameKey=Tic-Tac-Toe&GameID=example
```

#### You Recieve

```
{
    "Type": "Network",
    "Payload": {
        "GameKey": "Tic-Tac-Toe",
        "GameID": "example",
        "Name": "wry-gem"
    }
}
```

```
{
    "Type": "Game",
    "Payload": {
        "Turn": "red",
        "Teams": ["red", "blue"],
        "Winners": [],
        "MoreData": {
            "Board": [["", "", ""], ["", "", ""], ["", "", ""]]
        },
        "Targets": [
            {
                "Team": "red",
                "ActionType": "MarkLocation",
                "MoreDetails": {
                    "Row": 0,
                    "Column": 0
                }
            },
            ...
        ],
        "Message": "red must mark a location"
    }
}
```

#### All Recieve
```
{
    "Type": "Connected",
    "Payload": {
        "wry-gem": "",
        ...
    }
}
```

### Set Team

#### Send Message
```
{
    "ActionType": "SetTeam",
    "MoreDetails": {
        "Team": "red"
    }
}
```

#### All Recieve

```
{
    "Type": "Connected",
    "Payload": {
        "wry-gem": "red",
        ...
    }
}
```

### Game Action

#### Send Message

```
{
    "ActionType": "MarkLocation",
    "Team": "red",
    "MoreDetails": {
        "Row": 1,
        "Column": 1
    }
}
```

#### All Recieve

```
{
    "Type": "Game",
    "Payload": {
        "Turn": "blue",
        "Teams": ["red", "blue"],
        "Winners": [],
        "MoreData": {
            "Board": [["", "", ""], ["", "O", ""], ["", "", ""]]
        },
        "Targets": [
            {
                "Team": "blue",
                "ActionType": "MarkLocation",
                "MoreDetails": {
                    "Row": 0,
                    "Column": 0
                }
            },
            ...
        ],
        "Actions": [
            {
                "Team": "red",
                "ActionType": "MarkLocation",
                "MoreDetails": {
                    "Column": 1,
                    "Row": 1
                }
            }
        ],
        "Message": "blue must mark a location"
    }
}
```

### Undo Game Action

#### Send Message

```
{
    "ActionType": "Undo"
}
```

#### All Recieve

```
{
    "Type": "Game",
    "Payload": {
        "Turn": "red",
        "Teams": ["red", "blue"],
        "Winners": [],
        "MoreData": {
            "Board": [["", "", ""], ["", "", ""], ["", "", ""]]
        },
        "Targets": [
            {
                "Team": "red",
                "ActionType": "MarkLocation",
                "MoreDetails": {
                    "Row": 0,
                    "Column": 0
                }
            },
            ...
        ],
        "Message": "red must mark a location"
    }
}
```

### Chat 

#### Send Message

```
{
    "ActionType": "Chat",
    "MoreDetails": {
        "Msg": "hello"
    }
}
```

#### All Recieve

```
{
    "Type": "Chat",
    "Payload": {
        "Name": "wry-gem",
        "Msg": "hello"
    }
}
```

### Reset

#### Send Message

```
{
    "ActionType": "Reset"
}
```

#### All Recieve

```
{
    "Type": "Game",
    "Payload": {
        "Turn": "red",
        "Teams": ["red", "blue"],
        "Winners": [],
        "MoreData": {
            "Board": [["", "", ""], ["", "", ""], ["", "", ""]]
        },
        "Targets": [
            {
                "Team": "red",
                "ActionType": "MarkLocation",
                "MoreDetails": {
                    "Row": 0,
                    "Column": 0
                }
            },
            ...
        ],
        "Message": "red must mark a location"
    }
}
```
