# `kubo\core\corehttp\webui.go`

```
// 定义常量 WebUIPath，表示 Web UI 的路径，TODO: 后续需要移动到 IPNS
const WebUIPath = "/ipfs/bafybeidf7cpkwsjkq6xs3r6fbbxghbugilx3jtezbza7gua3k5wjixpmba" // v4.2.0

// 定义变量 WebUIPaths，包含所有过去的 WebUI 路径
var WebUIPaths = []string{
    WebUIPath,  // 将 WebUIPath 添加到 WebUIPaths 列表中
    "/ipfs/bafybeiamycmd52xvg6k3nzr6z3n33de6a2teyhquhj4kspdtnvetnkrfim",  // 添加其他过去的 WebUI 路径
    "/ipfs/bafybeieqdeoqkf7xf4aozd524qncgiloh33qgr25lyzrkusbcre4c3fxay",
    // ... 其他路径
    "/ipfs/bafybeicp23nbcxtt2k2twyfivcbrc6kr3l5lnaiv3ozvwbemtrb7v52r6i",
}
    # 定义了一系列的 IPFS 地址
    "/ipfs/bafybeidatpz2hli6fgu3zul5woi27ujesdf5o5a7bu622qj6ugharciwjq",
    "/ipfs/QmfQkD8pBSBCBxWEwFSu4XaDVSWK6bjnNuaWZjMyQbyDub",
    "/ipfs/QmXc9raDM1M5G5fpBnVyQ71vR4gbnskwnB9iMEzBuLgvoZ",
    "/ipfs/QmenEBWcAk3tN94fSKpKFtUMwty1qNwSYw3DMDFV6cPBXA",
    "/ipfs/QmUnXcWZC5Ve21gUseouJsH5mLAyz5JPp8aHsg8qVUUK8e",
    "/ipfs/QmSDgpiHco5yXdyVTfhKxr3aiJ82ynz8V14QcGKicM3rVh",
    "/ipfs/QmRuvWJz1Fc8B9cTsAYANHTXqGmKR9DVfY5nvMD1uA2WQ8",
    "/ipfs/QmQLXHs7K98JNQdWrBB2cQLJahPhmupbDjRuH1b9ibmwVa",
    "/ipfs/QmXX7YRpU7nNBKfw75VG7Y1c3GwpSAGHRev67XVPgZFv9R",
    "/ipfs/QmXdu7HWdV6CUaUabd9q2ZeA4iHZLVyDRj3Gi4dsJsWjbr",
    "/ipfs/QmaaqrHyAQm7gALkRW8DcfGX3u8q9rWKnxEMmf7m9z515w",
    "/ipfs/QmSHDxWsMPuJQKWmVA1rB5a3NX2Eme5fPqNb63qwaqiqSp",
    "/ipfs/QmctngrQAt9fjpQUZr7Bx3BsXUcif52eZGTizWhvcShsjz",
    "/ipfs/QmS2HL9v5YeKgQkkWMvs1EMnFtUowTEdFfSSeMT4pos1e6",
    "/ipfs/QmR9MzChjp1MdFWik7NjEjqKQMzVmBkdK3dz14A6B5Cupm",
    "/ipfs/QmRyWyKWmphamkMRnJVjUTzSFSAAZowYP4rnbgnfMXC9Mr",
    "/ipfs/QmU3o9bvfenhTKhxUakbYrLDnZU7HezAVxPM6Ehjw9Xjqy",
    "/ipfs/QmPhnvn747LqwPYMJmQVorMaGbMSgA7mRRoyyZYz3DoZRQ",
    "/ipfs/QmQNHd1suZTktPRhP7DD4nKWG46ZRSxkwHocycHVrK3dYW",
    "/ipfs/QmNyMYhwJUS1cVvaWoVBhrW8KPj1qmie7rZcWo8f1Bvkhz",
    "/ipfs/QmVTiRTQ72qiH4usAGT4c6qVxCMv4hFMUH9fvU6mktaXdP",
    "/ipfs/QmYcP4sp1nraBiCYi6i9kqdaKobrK32yyMpTrM5JDA8a2C",
    "/ipfs/QmUtMmxgHnDvQq4bpH6Y9MaLN1hpfjJz5LZcq941BEqEXs",
    "/ipfs/QmPURAjo3oneGH53ovt68UZEBvsc8nNmEhQZEpsVEQUMZE",
    "/ipfs/QmeSXt32frzhvewLKwA1dePTSjkTfGVwTh55ZcsJxrCSnk",
    "/ipfs/QmcjeTciMNgEBe4xXvEaA4TQtwTRkXucx7DmKWViXSmX7m",
    "/ipfs/QmfNbSskgvTXYhuqP8tb9AKbCkyRcCy3WeiXwD9y5LeoqK",
    "/ipfs/QmPkojhjJkJ5LEGBDrAvdftrjAYmi9GU5Cq27mWvZTDieW",
    "/ipfs/Qmexhq2sBHnXQbvyP2GfUdbnY7HCagH2Mw5vUNSBn2nxip",
# 结束一个代码块
}

# 创建一个名为WebUIOption的变量，并将RedirectOption("webui", WebUIPath)的返回值赋给它
var WebUIOption = RedirectOption("webui", WebUIPath)
```