# `v2ray-core\common\protocol\user.go`

```go
package protocol

// GetTypedAccount returns the typed account of the user, or an error if the account is missing or of an unknown type
func (u *User) GetTypedAccount() (Account, error) {
    // Check if the user's account is missing, and return an error if it is
    if u.GetAccount() == nil {
        return nil, newError("Account missing").AtWarning()
    }

    // Get the raw account instance and any error that occurs
    rawAccount, err := u.Account.GetInstance()
    if err != nil {
        return nil, err
    }

    // Check if the raw account can be converted to AsAccount type, and return it if possible
    if asAccount, ok := rawAccount.(AsAccount); ok {
        return asAccount.AsAccount()
    }

    // Check if the raw account can be converted to Account type, and return it if possible
    if account, ok := rawAccount.(Account); ok {
        return account, nil
    }

    // Return an error if the account type is unknown
    return nil, newError("Unknown account type: ", u.Account.Type)
}

// ToMemoryUser converts the user to a MemoryUser, and returns it along with any error that occurs
func (u *User) ToMemoryUser() (*MemoryUser, error) {
    // Get the typed account of the user, and return an error if it occurs
    account, err := u.GetTypedAccount()
    if err != nil {
        return nil, err
    }

    // Create a MemoryUser with the user's account, email, and level, and return it along with no error
    return &MemoryUser{
        Account: account,
        Email:   u.Email,
        Level:   u.Level,
    }, nil
}

// MemoryUser is a parsed form of User, to reduce number of parsing of Account proto.
type MemoryUser struct {
    // Account is the parsed account of the protocol.
    Account Account
    Email   string
    Level   uint32
}
```