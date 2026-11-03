# Earlier record

This document continues [draft-wachter-6lo-can][wachter], which was presented once
in the 6lo working group and then reviewed on the 6lo mailing list, but never got a
revision in reply before it expired. This folder collects that record so that the
comments are not lost a second time.

Everything here is an IETF Contribution, reproduced under BCP 78. The authoritative
copies are the ones linked below; these are convenience copies.

| File | What it is | Source |
|---|---|---|
| [`ietf106-slides-6locan.pdf`](ietf106-slides-6locan.pdf) | Alexander Wachter's slides, "IPv6 over CAN (6LoCAN)", 6lo session, IETF 106, 20 November 2019 | [datatracker][slides] |
| [`ietf106-6lo-minutes.txt`](ietf106-6lo-minutes.txt) | Minutes of that 6lo session | [datatracker][minutes] |
| [`2020-06-04-review-of-wachter-6lo-can-01.txt`](2020-06-04-review-of-wachter-6lo-can-01.txt) | Carles Gomez's review of draft-wachter-6lo-can-01, sent to the 6lo list on 4 June 2020 | [6lo list archive][review] |

[wachter]: https://datatracker.ietf.org/doc/draft-wachter-6lo-can/
[slides]: https://datatracker.ietf.org/meeting/106/materials/slides-106-6lo-ipv6-over-can-6locan
[minutes]: https://datatracker.ietf.org/doc/minutes-106-6lo/
[review]: https://www.mail-archive.com/6lo@ietf.org/msg01567.html

## What was raised at IETF 106

The agenda item ran out of time and the chairs deferred the questions to the list.
The substantive exchange, from the minutes:

> Carles: regarding 6loCAN, comments? Anybody willing to review?
>
> Pascal: seems SCHC would make sense for the 6loCAN case.
>
> Alexander: Pascal: considering your constraints (payload, etc.), SCHC seems
> appropriate.
>
> Carles: is ISO-TP the native fragmentation mechanism for 6LoCAN?
>
> Alexander: ISO-TP has 1 byte header, flow control. Already there for a long time.
>
> Carles: in the IETF there is the work produced by the LPWAN WG called SCHC, which
> includes fragmentation functionality. This fragmentation offers three different
> approaches, including flow control, and has also been designed for technologies
> with very small L2 MTU.

Both points are answered in -01. Header compression moves from LOWPAN_IPHC to SCHC,
and the choice to keep ISO-TP for fragmentation rather than using the SCHC
fragmentation mechanism is now stated with its reasons rather than left implicit.

The minutes also record that the 2019 draft used a 40-bit node address; the current
addressing scheme uses 14 bits, mapped into the 29-bit CAN frame identifier together
with a 14-bit source address and a multicast bit.

## Disposition of the 2020 review

The review has 35 points. The document has been restructured since, so the section
numbers below refer to the review's numbering of draft-wachter-6lo-can-01, not to the
current draft.

**Addressed in -01**

- Introduction: commas in "two-wire, wired-AND, multi-master bus"; CSMA/CR and CAN FD
  expanded on first use; the arbitration field explained; 6LoCAN expanded on first use
- A protocol stack figure showing how IPv6 is used over CAN
- Whether RFC 8724 was considered for fragmentation — Section 5.9 now gives the reasons
  for using ISO-TP instead
- "classical CAN" stated explicitly where it was meant
- The concept of a node address defined
- A callout for the address range table in the body text
- LLDAD defined once, at first use, and the duplicate definition removed
- NDP expanded
- "100 ms" rather than "100ms"
- RFC 8065 considered — now referenced in the stateless address autoconfiguration and
  security considerations sections
- The frame format section renamed "6LoCAN Frame Format"
- Whether an ISO-TP header is always present — it is, and a small packet now fits in a
  single CAN FD frame, so a compressed packet is not discarded
- The IANA considerations section no longer carries content that is not an IANA action
- Whether CAN offers security services at the link layer — stated explicitly, it does not

**No longer applicable**

- Three wording comments on the in-line IPv6 header fields. That text is gone: the frame
  layout now carries a SCHC Rule ID and compression residue.

**Still open, for a later revision**

- The main CAN characteristics, such as the physical-layer bit rate, are still not given
- The meaning of an ISO-TP abort, and what the sender does, is not specified
- "Network address extension and packet size larger than 4095 octets" still does not say
  whether "packet" means an IPv6 packet
- In the border translator section, "the source address is extended with a 34 bits MAC
  address prefix" still does not say that this is the CAN node address rather than an
  IPv6 address
- Several wording suggestions in the Introduction that need an editorial pass rather than
  a mechanical change
