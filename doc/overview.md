Project: /pandora/_project.yaml
Book: /pandora/_book.yaml

# Bluetooth test interfaces

The Pandora Bluetooth test interfaces are Remote Procedure Call (RPC) interfaces
exposed to testing tools to trigger behaviors within a Bluetooth stack under
test.

While all Bluetooth stacks are different in their supported profiles, features,
and corresponding APIs, the goal of Pandora is to provide a set of unified
test interfaces which they could all implement, so we can reuse and scale
testing tools and processes across all of them.

The Pandora Bluetooth test interfaces can be found in the
[`/bt-test-interfaces`](https://pandora.git.corp.google.com/bt-test-interfaces/)
on Pandora GoB. They are also copied to Bluetooth Host stacks implementing them
(such as [Android](
https://cs.android.com/android/platform/superproject/+/master:packages/modules/Bluetooth/android/blueberry/server/proto/pandora/)
and Eiffel).

## Requirements

Since each Bluetooth stack exposes different APIs, the test interfaces must be
generic enough and must not rely on any implementation specific behavior.
However, they must ensure that they provide all the necessary access to the
existing testing tools. For this reason, the test interfaces are co-designed by
multiple teams at Google.

The test interfaces must be implemented using [gRPC](https://grpc.io/) services
and must use [protocol buffers](https://developers.google.com/protocol-buffers)
as Interface Definition Language (IDL). A Bluetooth stack under test exposing
such interfaces must thus implement a gRPC server.

The test interfaces definition must follow the [Pandora style guide](
style-guide.md).

## Abstraction level

**A test interface is defined for each Bluetooth profile (standard or custom)**.
This allows the Bluetooth stack under test to implement only the test interfaces
corresponding to its supported profiles. Additional platform/device-specific
interfaces may also be added if necessary (but should be avoided as much as
possible).

**The same test interface can be implemented at different levels**: for
instance, in Android (see figure below showing an example for [PTS-bot](
/pandora/guides/pts-bot/overview)), the Pandora Bluetooth test interfaces can be
implemented both on top of Topshim (which is a Rust shim layer just on top of
the stack), which is advantageous as tests running at that level can apply to
Chrome OS as well, or on top of the Android Bluetooth SDK (Java) which is
advantageous for Mainline, since the Bluetooth module includes both the stack
and the SDK.

![PTS-bot on Android](
/pandora/reference/doc/images/pts-bot-aosp.svg)

## Optional features

As Bluetooth profiles contain optional features, some methods of the test
interfaces might not be implementable by a specific Bluetooth stack.

Such unimplemented methods must return an [UNIMPLEMENTED](
https://grpc.github.io/grpc/core/md_doc_statuscodes.html) status code as defined
by gRPC.

Discovering which features are supported by a Bluetooth stack is not (yet) part
of the test interfaces as this is already doable via Bluetooth SIG
[Implementation Conformance Statements (ICS)](
https://www.bluetooth.com/specifications/qualification-test-requirements/).