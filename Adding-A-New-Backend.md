# Create new backend
To add a new backend, create a new folder in `backend/` with the name of the backend.  The package name should be a new package name related to the new storage type.

If you embed the `backend.BackendUnsupported` type with the new storage struct, then any interface functions that are not implemented will automatically return the s3err.ErrNotImplemented error.  For example:

```go
package mystorage

type MyStorage struct {
	backend.BackendUnsupported
}
```

Implement the Stringer interface for the the new type:

```go
func (m *MyStorage) String() string { return "My Storage" }
```

It is also recommended to implement the Shutdown method that will be called upon gateway termination, but this is optional and the backend.BackendUnsupported provides a no-op shutdown:

```go
func (m *MyStorage) Shutdown() {
	// do important cleanup things here
}
```

**Implement the backend.Backend methods based on desired API functionality.**<br>
Implementing the backend.Backend methods will translate how you want your storage service to handle the corresponding frontend requests.

# Add backend to versitygw app
Create a new file: `cmd/versitygw/mystorage.go` with the appropriate name for the new backend type.

Create a new function that takes no args and returns a *cli.Command, and fill out appropriate fields for the new backend:
```go
func mystorageCommand() *cli.Command {
	return &cli.Command{
		Name:  "mystorage",
		Usage: "usage for mystorage",
		Description: "description for mystorage",
		Action: runMyStorage,
	}
}
```

And create the action function to run with the signature `func (ctx *cli.Context) error`:
```go
func runMyStorage(ctx *cli.Context) error {
	// do any setup or option parsing here

	be, err := mystorage.New()
	if err != nil {
		return fmt.Errorf("init mystorage: %w", err)
	}

	return runGateway(ctx.Context, be)
}
```

Add the storage subcommand to the main app in main() in `cmd/versitygw/main.go`:
```go
	app.Commands = []*cli.Command{
		posixCommand(),
		scoutfsCommand(),
		s3Command(),
		azureCommand(),
		mystorageCommand(),
		adminCommand(),
		testCommand(),
	}
```

Include documentation for the wiki if this will be added with a pull request.