---
sidebar_position: 3
---


# Telemetry Compliance

## Intuita VSCode’s Extension Telemetry Data

The Intuita VSCode’s Extension (v0.7.0 and later) tracks the events happening within the extension that have the following properties (this is described in the extension’s [source code](https://github.com/intuita-inc/intuita-vscode-extension/blob/main/src/telemetry/types.ts)):

- event type (kind):
    - extension is activated
    - extension is deactivated
    - a codemod set execution began, halted or ended,
    - jobs created
    - jobs accepted or rejected
- session ID:
    - created once per extension session by the user’s extension
    - you cannot identify a particular user by this information
- happened at:
    - when a particular event happened
- codemod set name:
    - passed if applicable
- codemod name:
    - passed if applicable
- execution ID:
    - created once per execution of a codemod set by the user’s extension
    - passed if applicable
- file count:
    - the number of files associated with a particular event
    - passed if applicable
- job count:
    - the number of jobs associated with a particular event
    - passed if applicable

Users can disable telemetry by going into Settings and searching the for `telemetryEnabled` setting under Intuita VSCode Extensions's Settings. Intuita also respects if the `telemetry.telemetryLevel` setting is set to off.


## Telemetry Backend

The extension sends all the event data to the Telemetry Backend over HTTPS. The data are stored securely in a Telemetry Database. The VPN settings disallow a direct access to the said database.
