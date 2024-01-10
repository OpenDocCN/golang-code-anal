# `trojan-go\common\geodata\decode.go`

```
// Package geodata includes utilities to decode and parse the geoip & geosite dat files.
//
// It relies on the proto structure of GeoIP, GeoIPList, GeoSite and GeoSiteList in
// github.com/v2fly/v2ray-core/v4/app/router/config.proto to comply with following rules:
//
// 1. GeoIPList and GeoSiteList cannot be changed
// 2. The country_code in GeoIP and GeoSite must be
//    a length-delimited `string`(wired type) and has field_number set to 1
//
package geodata

import (
    "errors"
    "io"
    "os"
    "strings"

    "google.golang.org/protobuf/encoding/protowire"
)

var (
    ErrFailedToReadBytes            = errors.New("failed to read bytes")
    ErrFailedToReadExpectedLenBytes = errors.New("failed to read expected length of bytes")
    ErrInvalidGeodataFile           = errors.New("invalid geodata file")
    ErrInvalidGeodataVarintLength   = errors.New("invalid geodata varint length")
    ErrCodeNotFound                 = errors.New("code not found")
)

func EmitBytes(f io.ReadSeeker, code string) ([]byte, error) {
    count := 1
    isInner := false
    tempContainer := make([]byte, 0, 5)

    var result []byte
    var advancedN uint64 = 1
    var geoDataVarintLength, codeVarintLength, varintLenByteLen uint64 = 0, 0, 0

Loop:
    // Implement the logic for EmitBytes function here
    }

    return result, nil
}

func Decode(filename, code string) ([]byte, error) {
    f, err := os.Open(filename)
    if err != nil {
        return nil, err
    }
    defer f.Close()

    geoBytes, err := EmitBytes(f, code)
    if err != nil {
        return nil, err
    }
    return geoBytes, nil
}
```