---
title: Network Manager and zbus
published_date: 2025-06-27 16:14:20.717539724 +0000
layout: default.liquid
is_draft: false
---
Traveling heavily recently, I've become fatigued from typing all the WiFi
passphrases. My phone can happily connect via a QR code. Why does my laptop
required so much typing, I asked. So I proposed a simple project.

Researching how I might use my web-cam to connect to WiFi networks, I came
across the [Network Manager and Rust's Zbus article on
rbs.io](https://rbs.io/2024/07/network-manager-and-rusts-zbus/). I use
`NewtworkManager` and I enjoy working with rust. I know next to nothing about
`DBus`, but I thought some knowledge of hacking Linux networks couldn't do me any
harm. Besides I had some free time.

The above article set me on the right path. The `zbus` documentation is still a
little obscure in some salient details, however. Hopefully this post will help
fill those in. The finished code is hosted in the
[scampi](https://github.com/tbro/scampi/tree/main) repo.

## Replacing nested `HashMap`s with structs.

I first needed to generate a `settings.rs` with `zbus-xmlgen` as described in
the above article. It is also straightforward to create a connection by building
the nested structures of `HashMaps` also described there. While that solution
works, it is tedious, unsightly and doesn't make the most of rust's type
system. Finding the [issue for using `DeserializeStruct` with nested
structs](https://github.com/dbus2/zbus/issues/312) closed, I hoped there might
now be a more ergonomic way forward.

With the help of the [zbus book](https://dbus2.github.io/zbus/faq.html) I
managed to piece together a solution that feels a little more rust-like. Of
course there are many pieces to the puzzle. You need `serde` and the `zvariant::Type`
macro. But all that is gleaned fairly easily from the documentation, I think. The more
salient discovery was that `zbus` allows declaring [DBbus
signatures](https://www.alteeve.com/w/List_of_DBus_data_types) on your
types. This is done via a macro, like so:

```rust
#[zvariant(signature = "a{sa{sv}}")]
struct MyType{..}
```

Declaring the correct signature(`a{sa{sv}}`) on the struct passed to
`NetworkManager`s `Settings.add_connection()` allowed it to navigate `Dbus`'s
type system. Its children don't have any children of their own, so
`SerializedDict` and `DeserializeDict` works fine on them.

For a more complete example you can have a look at [the
code](https://github.com/tbro/scampi/blob/9642b1a0ee6175469803bf572e13c90915b7d9e9/src/network_manager/mod.rs#L63-L73).

I believe I could go further and replace some fields defined as `String` with
more appropriate types, but it's one more thing to learn and I think this is good
enough for now.
