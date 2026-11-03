# Transmission of IPv6 Packets over Controller Area Network

This is the working area for the individual Internet-Draft, "Transmission of IPv6
Packets over Controller Area Network" (6LoCAN).

* [Editor's Copy](https://fnumber.github.io/6lo-can/draft-pjs-6lo-can.html)
  ([plain text](https://fnumber.github.io/6lo-can/draft-pjs-6lo-can.txt))
* [Datatracker Page](https://datatracker.ietf.org/doc/draft-pjs-6lo-can)
* [Compare Editor's Copy to the posted draft](https://author-tools.ietf.org/api/iddiff?doc_1=draft-pjs-6lo-can&url_2=https://fnumber.github.io/6lo-can/draft-pjs-6lo-can.txt)
* [Open issues](https://github.com/fnumber/6lo-can/issues)

## About

Controller Area Network (CAN) is a fieldbus designed for automotive applications
and since adopted for industrial and building automation. This document describes
how to transfer IPv6 packets over CAN, using ISO-TP (ISO 15765-2) for
segmentation, a dedicated 14-bit addressing scheme for CAN nodes mapped into the
29-bit CAN frame identifier, and header compression.

The document builds on
[draft-wachter-6lo-can](https://datatracker.ietf.org/doc/draft-wachter-6lo-can/),
an earlier Internet-Draft by Alexander Wachter.

## Versions

Submitted versions are marked with annotated tags whose names are the full draft
name including the revision number.

| Version | Status | Notes |
|---|---|---|
| [`draft-pjs-6lo-can-00`](https://fnumber.github.io/6lo-can/draft-pjs-6lo-can-00/) | Posted 2 November 2026 | Submitted before the IETF 127 cut-off |
| `draft-pjs-6lo-can-01` | In progress on `main` | SCHC replaces IPHC; CAN FD required; remote frames removed from the LLDAD |

## Contributing

See the
[guidelines for contributions](https://github.com/fnumber/6lo-can/blob/main/CONTRIBUTING.md).

Contributions can be made by creating pull requests or by raising issues. Review
of the addressing scheme (Section 5.3) and of the link-layer duplicate address
detection (Section 5.3.4) is particularly welcome, as those are the parts that are
hardest to change later.

## Command Line Usage

Formatted text and HTML versions of the draft can be built using `make`.

```sh
$ make
```

Command line usage requires that you have the necessary software installed. See
[the instructions](https://github.com/martinthomson/i-d-template/blob/main/doc/SETUP.md).
