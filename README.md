[![Build Status](https://github.com/sk3wlabs/go-serial/workflows/test/badge.svg)](https://github.com/sk3wlabs/go-serial/actions?workflow=test)

# github.com/sk3wlabs/go-serial

First off, kudos to the original package maintainers for, in my opinion, the best, cross-platform serial library for Go. Cheers.

This repo was forked from <github.com/bugst/go-serial> on 2023-04-26 to address a complication resulting from how the `nativeGetPortsList()` function (in UNIX) determines whether a `ttyS...` serial port is actually a device, or just a placeholder. It follows the pattern used in `setserial`:

- In `setserial`, the file descriptor is opened with `O_RDWR|O_NONBLOCK`, and then an ioctl is performed to query the device's current state. If it errors, it's not a [usable] serial device.
- In this library, the same thing is done, but the `fd` is opened with `O_RDWR|O_NOCTTY|O_NDELAY`. The documentation out there re: `O_NDELAY` and `O_NONBLOCK` can be contradictory, but generally either `O_NONBLOCK` is preferred for a more consistent experience across file descriptor types, or that there is no logical difference between the two.

**On a server with an active serial console, when we're looking for a different serial device, the instant the port enumeration touches the active serial console, the active serial console locks, and a reboot is required to regain use serial console.**  

In an attempt to write a fix that would be no-touch in terms of overall implementation, and low-touch in terms of implementation on unix (from the Go perspective) devices, I simply added a `UnixFdNonBlock` bool to the `Mode` struct that would `|= O_NONBLOCK` to the `unix.Open()` mode. However, the unix library reports that both `O_NDELAY` and `O_NONBLOCK` evaluate to the same `0x4` value.

Unfortunately, I'll need to write a heavier-handed fix, after which I'll speak with the original package maintainers on how best to contribute back a fix that doesn't make a breaking change.

## Changelog

- Functional Things:
  - Added a boolean field to the `Mode` struct called `UnixFdNonBlock` to control whether the filemode used with `unix.Open()` should be or-equalled with `O_NONBLOCK`.
- The Usual Fork Changes:
  - All (or most) url package references have been switched from the original repo to this repo.
  - Go version from `1.17` to `1.19`
    - `ioutil.ReadDir()` -> `os.ReadDir()` 

---
# Original Readme:

## go.bug.st/serial

A cross-platform serial library for go-lang.

### Documentation and examples

See the godoc here: https://godoc.org/go.bug.st/serial

### go.mod transition

This library now support `go.mod` with the import `go.bug.st/serial`.

If you came from the pre-`go.mod` era please update your import paths from `go.bug.st/serial.v1` to `go.bug.st/serial` to receive new updates. Anyway, the latest `v1` release should still be avaiable using the old import.

### Credits

:sparkles: Thanks to all awesome [contributors]! :sparkles:

### License

The software is release under a [BSD 3-clause license]

[contributors]: https://github.com/bugst/go-serial/graphs/contributors
[BSD 3-clause license]: https://github.com/bugst/go-serial/blob/master/LICENSE

