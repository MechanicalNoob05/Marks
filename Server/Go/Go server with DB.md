```
.
├── cmd
│   └── main.go
├── go.mod
└── internal

3 directories, 2 files
```

```
├── api 
│   └── app.go 
├── cmd 
│   └── main.go 
├── go.mod 
└── internal
```

```go
go get -u github.com/rs/zerolog/log 
go get github.com/labstack/echo/v4
```

```go
package main 
import ( "os" "time" "github.com/rs/zerolog" ) 
func main() { 
	zerolog.TimeFieldFormat = zerolog.TimeFormatUnix 
	logger :=zerolog.New(
		zerolog.ConsoleWriter{
			Out: os.Stderr, TimeFormat:time.RFC3339
			}
		). 
	Level(zerolog.TraceLevel). 
	With(). 
	Timestamp(). 
	Caller(). 
	Int("pid", os.Getpid()).
	Logger() 
}
```

```go
package api 
import "github.com/rs/zerolog" 
type App struct { logger zerolog.Logger } 
func NewApp(logger zerolog.Logger) *App { 
	return &App{ logger: logger, } 
}
```

```
. 
├── api 
│   └── app.go 
├── cmd 
│   └── main.go 
├── config 
│   └── config.go 
├── go.mod 
├── go.sum 
└── internal
```

```go
go get github.com/spf13/viper
```

```
. 
├── api 
│   └── app.go 
├── cmd 
│   └── main.go 
├── config 
│   └── config.go 
├── go.mod 
├── go.sum 
└── internal
└── docker-compose.yml
```

```docker
version: "3.9" 

services: 
	db: 
		image: postgres 
		restart: always 
		environment: 
			POSTGRES_USER: user 
			POSTGRES_PASSWORD: password 
			POSTGRES_DB: database 
		ports: 
			- 5432:5432 
	adminer: 
		image: adminer 
		restart: always 
		ports: 
			- 8080:8080
```

you will need sudo permission to execute this
```docker
docker compose up
```

```go
package config 
import ( "github.com/rs/zerolog" "github.com/spf13/viper" ) 
type Config struct { 
	Port int `mapstructure:"PORT"` 
	DatabaseDSN string `mapstructure:"DATABASE_DSN"` 
} 
func LoadConfig(logger zerolog.Logger) (Config, error) {
	viper.AddConfigPath(".") 
	viper.SetConfigName(".env") 
	viper.SetConfigType("env") 
	// Viper reads all the variables from env file and log error if any found if 
	err := viper.ReadInConfig(); err != nil { return Config{}, err } 
	// Unmarshal the config into a struct var cfg Config if 
	err := viper.Unmarshal(&cfg); err != nil { 
		return Config{}, err 
	} 
	if cfg.DatabaseDSN == "" { 
		panic("no database dsn") 
	} 
	if cfg.Port == 0 { 
		cfg.Port = 3000 
		logger.Info().Str("config", "Port not set, setting to :3000").Msg("") 
	} 
	logger.Info().Msg("Config loaded") 
	return cfg, nil 
}
```
Update app.go
```go
type App struct { 
	logger zerolog.Logger 
	config config.Config 
} 
func NewApp(logger zerolog.Logger, cfg config.Config) *App { 
	return &App{ 
		logger: logger, 
		config: cfg, 
	} 
}
```
.env file
```.env
PORT=3000 
DATABASE_DSN=postgres://user:password@localhost:5432/database
```

```go
go get github.com/labstack/echo/v4/middleware
```

Append the following to  app.go
```go
func (app *App) Start() { 
	e := echo.New() 
	// do this so u dont get spammed to fuck 
	e.Logger.SetLevel(log.INFO) 
	// format port string cause we are cool n shit 
	portStr := fmt.Sprintf(":%d", app.config.Port) 
	// we didnt setup request_id yet but its there for now
	e.Use(middleware.RequestLoggerWithConfig(middleware.RequestLoggerConfig{
		LogURI: true, 
		LogStatus: true, 
		LogRequestID: true, 
		LogValuesFunc: 
			func(c echo.Context, v middleware.RequestLoggerValues) error {
				app.logger.Info(). 
				Str("URI", v.URI). 
				Int("status", v.Status). 
				Str("request_id", v.RequestID). 
				Msg("request") return nil }, 
			}
		)
	) 
	// basic api endpoint, echo.Context is where u get most of the shit and do most of the shit shit and shit stuff 
	e.GET("/", func(c echo.Context) error { 
		return c.String(http.StatusOK, "Hello, World!") 
		}
	) 
	e.Logger.Fatal(e.Start(portStr)
	) 
}
```

```go
func main() { 
	zerolog.TimeFieldFormat = zerolog.TimeFormatUnix 
	logger := zerolog.New(
		zerolog.ConsoleWriter{Out: os.Stderr, TimeFormat: time.RFC3339}
		). 
		Level(zerolog.TraceLevel). 
		With(). 
		Timestamp(). 
		Caller(). 
		Int("pid", 
		os.Getpid()). 
		Logger() cfg, 
		err := config.LoadConfig(logger) 
		if err != nil { 
			panic(err) 
		} 
		app := api.NewApp(logger, cfg) 
		app.Start() 
}
```
This should start up the server
```go
go run ./cmd
```
Sending get request to localhost for testing 
```curl
curl --GET http://localhost:3000
```
Add two new files in api dir routes.go,handler.go
```
. 
├── api    
│   └── app.go
│   └── routes.go
│   └── handler.go
│   
├── cmd 
│   └── main.go 
├── config 
│   └── config.go 
├── go.mod 
├── go.sum 
└── internal
└── docker-compose.yml
```
Put this in routes.go to create routes
```go
package api 
import "github.com/labstack/echo/v4" 
func (app *App) routes(e *echo.Echo) { 
	app.logger.Trace().Msg("Registering routes") 
	apiGroup := e.Group("/api/v1") apiGroup.GET("/health-check", app.healthCheckHandler) 
}
```
Put this in handler.go
```go
func (app *App) healthCheckHandler(c echo.Context) error {

    return c.JSON(http.StatusOK, "FUCK")
}
```
Put this in app.go
```go
func (app *App) Start() {

    e := echo.New()
    // do this so u dont get spammed to fuck
    e.Logger.SetLevel(log.INFO)

    // format port string cause we are cool n shit
    portStr := fmt.Sprintf(":%d", app.config.Port)

    // we didnt setup request_id yet but its there for now
    e.Use(middleware.RequestLoggerWithConfig(middleware.RequestLoggerConfig{
        LogURI:       true,
        LogStatus:    true,
        LogRequestID: true,
        LogValuesFunc: func(c echo.Context, v middleware.RequestLoggerValues) error {
            app.logger.Info().
                Str("URI", v.URI).
                Int("status", v.Status).
                Str("request_id", v.RequestID).
                Msg("request")

            return nil
        },
    }))

    // removed the e.GET thing and added this, pass in echo
    app.routes(e)

    e.Logger.Fatal(e.Start(portStr))

}
```
Testing the route
```curl
http://localhost:3000/api/v1/health-check
```
> Expected fuck in return

Append to handler .go 
```go
type TestJsonReq struct {
    Name    string `json:"name"`
    Message string `json:"message"`
}

func (app *App) testJsonHandler(c echo.Context) error {

    var req TestJsonReq

    if err := c.Bind(&req); err != nil {
        return err
    }

    return c.JSON(200, req)
}
```
Updates in routes.go
```go
func (app *App) routes(e *echo.Echo) {
    app.logger.Trace().Msg("Registering routes")

    apiGroup := e.Group("/api/v1")
    apiGroup.GET("/health-check", app.healthCheckHandler)
    apiGroup.POST("/test-json", app.testJsonHandler)
}
```
curl request to test new route with post request
```curl
curl -X POST -H "Content-Type: application/json"     -d '{"name": "linuxize", "message": "linuxize@example.com"}'     http://localhost:3000/api/v1/test-json
```
Installing goose
```go
go install github.com/pressly/goose/v3/cmd/goose@latest
```
exporting Go path to .bashrc to run goose
```bash
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.bashrc && source ~/.bashrc
```

```
.
├── api
│   ├── app.go
│   ├── handlers.go
│   └── routes.go
├── cmd
│   └── main.go
├── config
│   └── config.go
├── docker-compose.yml
├── go.mod
├── go.sum
├── internal
└── migrations
```

Creating migration script
```go
goose create create_users sql
```

```
.
├── api
│   ├── app.go
│   ├── handlers.go
│   └── routes.go
├── cmd
│   └── main.go
├── config
│   └── config.go
├── docker-compose.yml
├── go.mod
├── go.sum
├── internal
└── migrations
    └── 20230617161546_create_users.sql
```

```go
go get github.com/jackc/pgx/v5/pgxpool
```
Writing migration file created in migration folder after running
`goose create create_users sql`

```sql
-- +goose Up
-- +goose StatementBegin
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
-- +goose StatementEnd

-- +goose Down
-- +goose StatementBegin
DROP TABLE users;
-- +goose StatementEnd
```

Make db file in database directory in internal
```
.
├── api
│   ├── app.go
│   ├── handlers.go
│   └── routes.go
├── cmd
│   └── main.go
├── config
│   └── config.go
├── docker-compose.yml
├── go.mod
├── go.sum
├── internal
│   └── database
│	    └── db.go
└── migrations
    └── 20230617161546_create_users.sql
    
```

```go
package database

import (
    "context"

    "github.com/jackc/pgx/v4/log/zerologadapter"
    "github.com/jackc/pgx/v4/pgxpool"
    "github.com/rs/zerolog"
)

func Connect(dsn string, logger zerolog.Logger) (*pgxpool.Pool, error) {

    config, err := pgxpool.ParseConfig(dsn)

    if err != nil {
        return nil, err
    }

    adapter := zerologadapter.NewLogger(logger)

    config.ConnConfig.Logger = adapter

    return pgxpool.ConnectConfig(context.Background(), config)
}
```
Create two additional folder and file set call model/model.go and store/store.go
```
.
├── api
│   ├── app.go
│   ├── handlers.go
│   └── routes.go
├── cmd
│   └── main.go
├── config
│   └── config.go
├── docker-compose.yml
├── go.mod
├── go.sum
├── internal
│   └── database
│	│   └── db.go
│   └── model
│	│   └── model.go
│   └── store
│	    └── stores.go
└── migrations
    └── 20230617161546_create_users.sql
```

```go
package store

import "github.com/jackc/pgx/v5/pgxpool"

type Stores struct {
    Users *UserStore
}

type UserStore struct {
    DB *pgxpool.Pool
}

func NewStores(db *pgxpool.Pool) *Stores {

    return &Stores{
        Users: NewUserStore(db),
    }
}
func NewUserStore(db *pgxpool.Pool) *UserStore {
    return &UserStore{
        DB: db,
    }
}
```
Replace connect function in db.go
```go
func Connect(dsn string, logger zerolog.Logger) (*pgxpool.Pool, error) {

    config, err := pgxpool.ParseConfig(dsn)

    if err != nil {
        return nil, err
    }

    return pgxpool.NewWithConfig(context.Background(), config)
}
```
models.go
```go
package model

import "time"

type User struct {
    ID        int
    Username  string
    Email     string
    CreatedAt time.Time
    UpdatedAt time.Time
}
```
Update App struct in app.go
```go
type App struct {
    logger zerolog.Logger
    config config.Config
    stores *store.Stores
}
```
Update NewApp function as well
```go
func NewApp(logger zerolog.Logger, cfg config.Config, db *pgxpool.Pool) *App {
    return &App{
        logger: logger,
        config: cfg,
        stores: store.NewStores(db),
    }
}
```

update the main function
```go
func main() {

    zerolog.TimeFieldFormat = zerolog.TimeFormatUnix
    logger := zerolog.New(zerolog.ConsoleWriter{Out: os.Stderr, TimeFormat: time.RFC3339}).
        Level(zerolog.TraceLevel).
        With().
        Timestamp().
        Caller().
        Int("pid", os.Getpid()).
        Logger()

    cfg, err := config.LoadConfig(logger)
    if err != nil {
        panic(err)
    }

    db, err := database.Connect(cfg.DatabaseDSN, logger)
    if err != nil {
        panic(err)
    }

    app := api.NewApp(logger, cfg, db)

    app.Start()

}
```
Creating new user in data base add the function in store.go
```go
func (us *UserStore) Insert(user *model.User) (int, error) {
    query := `INSERT INTO users (username, email, created_at, updated_at) VALUES ($1, $2, $3, $4) RETURNING id`

    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    var id int
    err := us.DB.QueryRow(ctx, query, user.Username, user.Email, user.CreatedAt, user.UpdatedAt).Scan(&id)
    if err != nil {
        return 0, err
    }

    return id, nil
}
```
Get user using id
```go
func (us *UserStore) GetByID(id string) (*model.User, error) {
    query := `SELECT id, username, email, created_at, updated_at FROM users WHERE id = $1`

    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    row := us.DB.QueryRow(ctx, query, id)

    var user model.User

    err := row.Scan(&user.ID, &user.Username, &user.Email, &user.CreatedAt, &user.UpdatedAt)
    if err != nil {
        return nil, err
    }

    return &user, nil
}
```
Update and delete
```go
func (us *UserStore) Update(user *model.User) error {
    query := `UPDATE users SET username = $1, email = $2, updated_at = NOW() WHERE id = $3`

    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    _, err := us.DB.Exec(ctx, query, user.Username, user.Email, user.ID)
    if err != nil {
        return err
    }

    return nil
}


func (us *UserStore) Delete(id int) error {
    query := `DELETE FROM users WHERE id = $1`

    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    _, err := us.DB.Exec(ctx, query, id)
    if err != nil {
        return err
    }

    return nil
}
```


Updates in handler.go
```go
type CreateUserReq struct {
    Username string `json:"username"`
    Email    string `json:"email"`
}

func (app *App) createUserHandler(c echo.Context) error {

    var req CreateUserReq

    if err := c.Bind(&req); err != nil {
        return err
    }

    var user model.User

    // make sure these are set
    user.CreatedAt = time.Now()
    user.UpdatedAt = time.Now()

    user.Email = req.Email
    user.Username = req.Username

    id, err := app.stores.Users.Insert(&user)
    if err != nil {
        return err
    }

    return c.JSON(201, echo.Map{
        "id": id,
    })
}

type GetUserResp struct {
    ID        int       `json:"id"`
    Username  string    `json:"username"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_a"`
}

func (app *App) getUserHandler(c echo.Context) error {

    id := c.Param("id")

    user, err := app.stores.Users.GetByID(id)
    if err != nil {
        return err
    }

    var resp GetUserResp

    resp.ID = user.ID
    resp.Username = user.Username
    resp.Email = user.Email
    resp.CreatedAt = user.CreatedAt
    resp.UpdatedAt = user.UpdatedAt

    return c.JSON(200, resp)

}
```

Creating Routes in routes.go
```go
apiGroup.POST("/users", app.createUserHandler)
apiGroup.GET("/users/:id", app.getUserHandler)
```

Update function handler in handler.go
```go
type UpdateUserReq struct {
    Username *string `json:"username"`
    Email    *string `json:"email"`
}

func (app *App) updateUserHandler(c echo.Context) error {
    var req UpdateUserReq

    if err := c.Bind(&req); err != nil {
        return err
    }

    id := c.Param("id")

    user, err := app.stores.Users.GetByID(id)
    if err != nil {
        return err
    }

    if req.Username != nil {
        user.Username = *req.Username
    }

    if req.Email != nil {
        user.Email = *req.Email
    }

    user.UpdatedAt = time.Now()

    err = app.stores.Users.Update(user)
    if err != nil {
        return err
    }

    return c.JSON(200, user)
}
```

route for handling update
```go
apiGroup.PATCH("/users/:id", app.updateUserHandler)
```