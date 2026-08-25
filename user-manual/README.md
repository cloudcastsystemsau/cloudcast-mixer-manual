# CloudCast Mixer User Manual

CloudCast Mixer is an **AES67 8-bus broadcast mixer**: a realtime audio engine and its
browser control surface, run as one service on Linux — on hardware, in EC2, or in a
container. The desk has **16 faders**, four programme buses (**PGM 1–4**), four auxiliary
buses (**AUX 1–4**), a **per-fader mix-minus** for every channel, and **cue/PFL** with
monitoring in the operator's own browser.

This manual is for the person driving the desk. It covers every panel of the control
surface, the channel strip, mix-minus and backfeeds, sources and discovery, outputs,
console profiles, the clock, intercom, physical control surfaces and the rack panel.
Deployment — the container image, realtime privileges, PTP and firewalls — lives in the
repository's `deploy/` documentation; the control API is specified in
`docs/control-api.md`.

Everything an operator touches on this desk is also available over REST and WebSocket, so
anything this manual shows can be automated.

---

## Contents

1. [Core concepts](#1-core-concepts)
   - [The desk at a glance](#the-desk-at-a-glance) · [Sources, streams and channels](#sources-streams-and-channels) · [**Buses**](#buses)
2. [Getting started](#2-getting-started)
   - [Signing in](#signing-in) · [The surface](#the-surface) · [The header status row](#the-header-status-row)
3. [The channel strip](#3-the-channel-strip)
   - [Patching a source](#patching-a-source) · [Trim, pan and balance](#trim-pan-and-balance) · [**PGM assign**](#pgm-assign) · [The fader and the meter](#the-fader-and-the-meter) · [ON and CUE](#on-and-cue) · [Aux sends](#aux-sends) · [Strip badges](#strip-badges)
4. [Banks](#4-banks)
5. [The main meter and the monitor](#5-the-main-meter-and-the-monitor)
   - [Main meter](#main-meter) · [Headphones](#headphones) · [**Cue modes**](#cue-modes) · [Bus trims](#bus-trims)
6. [Cue and PHONES](#6-cue-and-phones)
7. [Processing](#7-processing)
   - [EQ](#eq) · [De-ess](#de-ess) · [Dynamics](#dynamics)
8. [Mix-minus and backfeeds](#8-mix-minus-and-backfeeds)
9. [Sources](#9-sources)
   - [Configured sources](#configured-sources) · [Discovered on the network](#discovered-on-the-network) · [Adding a source manually](#adding-a-source-manually)
10. [Outputs](#10-outputs)
11. [Console profiles](#11-console-profiles)
    - [**What a profile contains**](#what-a-profile-contains) · [Saving](#saving) · [**Recalling is safe**](#recalling-is-safe) · [Managing the list](#managing-the-list)
12. [Clock](#12-clock)
13. [Intercom](#13-intercom)
14. [Control surfaces](#14-control-surfaces)
15. [The rack panel](#15-the-rack-panel)
16. [Accounts, roles and security](#16-accounts-roles-and-security)
17. [Themes](#17-themes)

---

## 1. Core concepts

### The desk at a glance

![The desk mid-programme: four live sources, cue engaged on Guest 1, the main meter on PGM 1](img/02-desk-overview.png)

One service runs both halves of the product: the **engine**, a realtime audio process that
mixes AES67 streams sample-exactly, and the **control surface**, the browser page in front
of you. The surface never carries audio to air — closing the browser changes nothing on
the plant. The one exception is deliberate: **PHONES** plays the cue bus *into your
browser*, so you can pre-fade listen from wherever you are sitting ([§6](#6-cue-and-phones)).

### Sources, streams and channels

Three ideas, in one direction:

| Thing | What it is | Where it is managed |
|---|---|---|
| **Stream** | An AES67 endpoint — a multicast address the desk receives or transmits | Sources tab (receive), Outputs tab (transmit) |
| **Source** | A named thing you can put on a fader — "Host Mic", "Playout" — wired to a receive stream, carrying its own processing, backfeed and bus permissions | Sources tab |
| **Channel** | One of the 16 faders, with a source patched to it | Mixer tab |

A fader's label is the name of the source patched to it. Processing and backfeed belong to
the **source**, so they follow it onto whichever fader it is patched to.

### Buses

Eight mix buses, in two families:

- **PGM 1–4** — programme buses. Every channel taps them **post-fader**: the fader and the
  ON switch always govern what reaches programme.
- **AUX 1–4** — auxiliary buses, fed by per-channel **aux sends** with their own level and
  a selectable tap point ([§3](#aux-sends)). Auxes carry mix-minus returns, IFB, FX — anything
  that is not programme.

Each bus has a label, a master gain, and (once bound on the Outputs tab) an AES67 transmit
stream. Beside the eight buses there is one **cue** bus ([§6](#6-cue-and-phones)) and a
**headphones** monitor selection ([§5](#5-the-main-meter-and-the-monitor)).

---

## 2. Getting started

### Signing in

![The sign-in view](img/01-login.png)

Browse to the desk's address and sign in. The lead reads **Sign in to open the desk.** —
enter **Username** and **Password** and press **Sign in**.

A fresh installation has one account, **admin**. Its password is either the value the
deployment set (`CCMIX_BOOTSTRAP_PASSWORD`) or a generated one printed once in the
server's log at first start — it is stored hashed and cannot be recovered, only reset.
Five failed attempts lock the account; sessions last 12 hours.

### The surface

Across the top, eight tabs: **Mixer**, **Mix-Minus**, **Sources**, **Outputs**,
**Intercom**, **Clock**, **Control Surface**, **Profiles**. Everything in this manual
lives under one of them. The **Mixer** tab is the desk itself: the channel strips in two
banks of eight, with the main meter and monitor section on the right.

### The header status row

The right side of the header is always visible, whatever tab you are on:

| Control | What it tells you |
|---|---|
| **PHONES** | Plays the cue bus in this browser ([§6](#6-cue-and-phones)) |
| **CUE** counter and level | How many channels are cued, and the cue master level; **CLEAR** drops every engagement |
| Profile badge | The active profile's name; its dot lights when the desk differs from what was last saved ([§11](#11-console-profiles)) |
| **Connected** | The live WebSocket to the desk. **Connecting…** means the surface is not currently talking to the mixer — what you see may be stale |
| **Dark** / **Light** | The desk's own theme toggle ([§17](#17-themes)) |
| Account badge | Who you are signed in as, a **read-only** chip if your role cannot operate the desk, and **Sign out** |

---

## 3. The channel strip

![One channel strip: source, trim, badges, balance, PGM assigns, fader and meter, ON, CUE, aux sends](img/03-channel-strip.png)

Top to bottom, every strip is the same instrument.

### Patching a source

![The source picker open on a strip](img/04-source-picker.png)

The button under the strip's name shows the patched source (or **NO SOURCE**). Click it to
open the picker and choose from the configured sources ([§9](#9-sources)); the **×** beside it
clears the patch. Under the source name the strip shows the wire format — e.g.
**STEREO · L24** — or **not wired to a stream** for a source with no stream yet.

Two health badges can appear here, and they mean different things:

- **NO SIGNAL** — the desk is not receiving the source's stream at all.
- **STALLED — audio thread late** — packets are arriving but the audio engine is not
  keeping up. On a correctly deployed mixer this indicates the host is starving the
  realtime thread — see the deployment documentation's realtime privileges.

### Trim, pan and balance

The stepper under the source sets **input trim**, −20 to +20 dB — gain before everything
else on the strip. Below the badges, a mono source gets **Pan** and a stereo source gets
**Balance**; the centre button returns it to **C**.

### PGM assign

The four numbered buttons **1 2 3 4** assign the channel to PGM 1–4. Lit red means
assigned. A source's configuration can forbid particular buses ([§9](#configured-sources)); a
forbidden assignment renders disabled with the reason in its tooltip.

### The fader and the meter

The fader sets the channel's level into every post-fader destination. Beside it, the
channel meter draws the source's level — peak with a true-peak tick — and, when the
source's processing is engaged, a narrow **gain-reduction** column alongside
([§7](#7-processing)). Meters update 25 times a second and read down to −50 dBFS.

### ON and CUE

**ON** puts the channel on air — it is the switch between the fader and every destination.
**OFF** is not a mute button on a live path so much as the channel not being open at all:
backfeeds can switch what they return based on it ([§8](#8-mix-minus-and-backfeeds)).

**CUE** engages pre-fade listen on the cue bus — see [§6](#6-cue-and-phones). While the desk
confirms the engagement the button shows a dashed ring; solid means the engine has it.

### Aux sends

![A strip with its aux sends open](img/05-aux-sends.png)

The **AUX** row at the strip's foot expands into one row per aux bus. Each send has an
enable, a level (−60 to +10 dB), and a **tap point**:

| Tap | Face | Behaviour |
|---|---|---|
| Pre | **PRE** | *always on* — feeds the aux regardless of the fader and the ON switch |
| Pre-fader | **PRE-FADER** | *follows ON* — open whenever the channel is on, at send level |
| Post | **POST** | *follows fader* — the send follows the fader and ON, like programme does |

**PRE** is the tap for mix-minus returns — the far end keeps hearing the studio even when
their own fader is closed. The **Aux sends: Shown / Hidden** toggle on the toolbar opens
and closes the section on every strip at once.

### Strip badges

| Badge | Meaning |
|---|---|
| **DSP** | The source's processing is engaged (not bypassed) — [§7](#7-processing) |
| **BF** | This fader carries the source's backfeed return — [§8](#8-mix-minus-and-backfeeds) |
| **BF→3** | The source's backfeed is carried by another fader (here, fader 3) |
| **PROC** | Opens the processing dialog |

---

## 4. Banks

![The bank selector with activity badges on both banks](img/08-bank-badge.png)

The desk's 16 faders present as two banks: **CH 1–8** and **CH 9–16**. A badge on the
unselected bank counts its channels that are ON, so a live microphone can never hide
behind the bank you are not looking at — the badge turns red when any of them is assigned
to programme (on air), and its tooltip counts both states.

---

## 5. The main meter and the monitor

![The rack sidebar: main meter on PGM 1, and the monitor section with headphones selection and cue mode](img/06-monitor-sidebar.png)

### Main meter

The **MAIN METER** draws one bus, stereo, with a numeric peak readout beneath. The
**P1–P4 / A1–A4** selector chooses which bus it follows. A clip latches at the top of the
scale until **Clear clip** is pressed — a clip that happened while you looked away still
shows.

### Headphones

**HEADPHONES** selects what you monitor — any PGM or AUX bus — with its own level stepper.
This is the monitor selection the cue system plays against.

### Cue modes

How cue interacts with your monitoring, selectable as three modes:

| Mode | Face | Behaviour |
|---|---|---|
| Split | **SPLIT** | *Air left, cue right* |
| Mix | **MIX** | *Cue over a dimmed selection* — the dim depth stepper appears below, and dims **the selection under cue. Never cue itself.** |
| Takeover | **TAKEOVER** | *Cue replaces the selection* |

### Bus trims

**BUS TRIMS** gives every one of the eight buses a master gain stepper (−20 to +20 dB)
without leaving the desk.

---

## 6. Cue and PHONES

![The cue controls in the header: PHONES, the engagement counter, cue master and CLEAR](img/07-cue-section.png)

Pressing a strip's **CUE** puts that channel, pre-fade, on the cue bus. The header shows
the count of engaged channels, the **CUE** master level stepper (−60 to +10 dB), and
**CLEAR**, which drops every engagement at once.

**PHONES** plays the cue bus *in this browser* — the one place the surface carries audio.
Browsers hold audio until the page has been interacted with; if you press **PHONES**
before touching anything else the desk shows **The browser is holding audio until you
interact with the page.** with a retry button.

Cue engagement and the cue master are **runtime state, not profile state**: a restart or a
profile recall comes up with nothing in the headphones, deliberately.

---

## 7. Processing

![The processing dialog on the EQ tab](img/09-dsp-eq.png)

Each source carries its own processing chain, opened with the strip's **PROC** button. The
dialog's subtitle states the model: **Belongs to the source — it follows it onto any
fader.** Its header meters show the processed output (**OUT**) and gain reduction
(**GR**) live. Three tabs: **EQ**, **De-ess**, **Dynamics**.

### EQ

Parametric EQ with a draggable response curve. Each band selects its type — **Peak**,
**Low shelf**, **High shelf**, **High-pass**, **Low-pass** — with frequency, gain and Q,
plus an overall trim.

### De-ess

A dedicated de-esser with the usual threshold and frequency-region controls.

### Dynamics

![The processing dialog on the Dynamics tab, gain reduction live](img/10-dsp-dynamics.png)

**Compressor** and **Gate**, each with an enable, threshold, ratio, knee, attack, release
and make-up, with **Auto make-up** and **Auto release** options; the limiter lives at the
end of the same tab. While the compressor works, the strip's meter grows its
gain-reduction column and the strip shows the **DSP** badge.

---

## 8. Mix-minus and backfeeds

![The Mix-Minus tab: one row per fader, live bus and configured pair](img/11-mixminus.png)

Every fader has a mix-minus feed: a return to the far end that is *everything minus
themselves*, derived sample-exactly so a caller never hears their own voice late. The
**Mix-Minus** tab is the desk-wide view — one row per channel showing the source patched,
the bus the return is derived from **now**, the configured **ON x · OFF y** pair,
**MONO**/**STEREO**, and **Backfeed off** or **No source patched to this fader** where
that is the state.

The configuration itself belongs to the **source** — the row's **Edit on …** button jumps
to it on the Sources tab. A backfeed configures:

- the bus the return derives from while the fader is **ON** (typically the aux carrying
  the studio's mix-minus),
- an optional different bus while the fader is **OFF** (typically programme, so a
  contributor waiting to come on hears air),
- mono or stereo, and the transmit stream that carries it back.

Because the mix-minus subtracts with the same per-sample ramps as the bus sum, the
cancellation holds *during* fader moves, not just at rest.

---

## 9. Sources

![The Sources tab: configured sources above, SAP discovery below](img/12-sources.png)

### Configured sources

**Configured sources** is the desk's own list — everything that can be patched to a
fader. Each row shows the wire (address, **STEREO · L24**, …) and a **CH n** chip naming
the fader(s) it is patched to, or **no stream wired** for a placeholder. Expanding a row
opens its four sections: **Name**, **Permitted buses**, **Backfeed**, **Processing**.

![A source expanded: name, permitted buses, backfeed and processing](img/13-source-expanded.png)

**Permitted buses** restricts which buses the source may be assigned to — a talkback
microphone that must never reach programme is enforced here, and the strip's forbidden
PGM buttons render disabled with the reason. A source that is configured but not
currently announcing on the network wears a **NOT ANNOUNCING** badge. **Delete source**
is refused while the source is patched to any fader.

### Discovered on the network

**Discovered on the network** lists what is announcing right now over SAP. The panel's own
hint states the model: discovery is how a source is *found*, never how it is *owned*.
**Promote to source** turns an announcement into a configured source with one click and a
name; announcements that match something already configured say so. **Refresh** re-asks.

### Adding a source manually

![The manual source form](img/14-manual-source.png)

**+ Add source manually** is for endpoints that do not announce — an ISDN codec, a fixed
encoder. Fields: **Name**, **Address**, **Port**, **Channels** (**1 (mono)** / **2
(stereo)**), **Encoding** (**L24**/**L16**), **Packet time (µs)**, **TTL**.

---

## 10. Outputs

![The Outputs tab: output routing per bus, monitor outputs below](img/15-outputs.png)

**Output routing** is where buses become AES67 transmit streams. The panel's hint is the
contract: binding creates the transmit endpoint and points the bus at it in one step;
unbinding stops the bus transmitting immediately, so it always asks first.

![An output's bind form open](img/16-output-bind.png)

Each row carries a compact meter and transmit health — packets, starvation, errors. The
bind form takes **Label**, **Address**, **Port**, **Channels**, **Encoding**, **Packet
time (µs)**, **TTL**. Unbinding warns in place — **Unbinding will stop … transmitting
to …** — and waits for confirmation.

**Monitor outputs** gives the **cue** bus and the **headphones** selection their own
transmit streams, for a physical monitor feed in the studio.

---

## 11. Console profiles

![The Profiles tab: the active profile, the saved list, and per-profile actions](img/19-profiles.png)

### What a profile contains

A profile is the desk's setup for a show: source configuration, labels, trims, PGM
assignments, aux sends and tap points, backfeeds, bus labels and trims. Operators switch
profiles between programmes and expect the desk to come back exactly as they left it.

Two kinds of state are deliberately **not** in a profile:

- **Performance** — fader positions, ON states, cue engagements. These reset on load, for
  the same reason they reset on restart.
- **Machine-local** — stream addresses and bindings, interface names, clock
  configuration, accounts. A profile is portable between mixers; anything that would
  break when carried to another machine is kept out of it.

### Saving

The bar names the **Active profile** and tracks **Unsaved changes** / **Saved** — the
same state as the header badge's dot. **Save to active profile** writes the desk as it is
now. There is no autosave: the desk you shaped during a show does not overwrite the
profile behind you unless you say so.

### Recalling is safe

![The recall confirmation](img/20-profiles-recall-warning.png)

**Recall** activates another profile — after a confirmation (**Recall anyway** /
**Cancel**) if the desk has unsaved changes. A recall **never puts audio to air**: the
desk comes up with every fader closed and every channel off, so recalling mid-programme
cannot surprise you with an open microphone.

### Managing the list

**+ New profile** creates one at factory settings; each row offers **Recall**,
**Duplicate**, **Delete** and inline rename. The active profile and the last remaining
profile cannot be deleted, and the rows say so.

---

## 12. Clock

![The Clock tab](img/17-clock.png)

AES67 lives and dies by its media clock, and the **Clock** tab shows what the desk's
clock is actually doing: the PTP port state, interface, timestamping and delay mechanism,
the grandmaster's identity and qualifications (priorities, clock class, accuracy,
variance, time source), offset from master and mean path delay. When the desk cannot make
a conformant claim — no PTP hardware clock, a free-running grandmaster — this panel says
so rather than implying lock. A trend table records recent clock behaviour.

---

## 13. Intercom

![The Intercom tab](img/21-intercom.png)

When the deployment wires the desk to a RemoteTalk intercom, the **Intercom** tab is a
key panel: a grid of channels with **TALK** and **LISTEN** keys, tallies for who is
talking, page tabs for larger panels, and a **REC** chip on channels being recorded. The
header's intercom indicator lights for an incoming private call from anywhere in the
surface, and clicking it jumps here. Without an intercom configured the tab reports that
plainly.

---

## 14. Control surfaces

![The Control Surface tab](img/18-control-surface.png)

The **Control Surface** tab pairs a physical Stream Deck with the desk over WebHID. The
capability block reports what this browser can do — from **Not in this browser** and
**Not from this address** through **Ready — nothing granted yet** to **Granted** — and
**Connect a control surface** asks the browser to grant a device; **Forget this device**
revokes it. The panel states the mapping in sentences: keys cue channels exactly as the
CUE buttons on the mixer do, and the rotaries follow the desk's selection.

Grants are per-browser and per-address, and WebHID requires a secure context — the
capability block names whichever of these is the obstacle.

---

## 15. The rack panel

![The rack panel in demo mode](img/22-rack-panel.png)

`/panel` is a chrome-free strip built for a rack display or an under-monitor screen:
per-channel level, ON and CUE state, and the four programme meters, with a **LIVE** /
**DOWN** connection pill. URL parameters shape it — `theme`, `width`, `channels`,
`meters`, `controls` — so one bookmark is one panel.

The **DEMO** toggle draws invented levels for demonstrations; its own tooltip is the
contract: **Show invented levels for a demonstration. Nothing on the desk changes.** The
screenshot above is the panel in demo mode.

---

## 16. Accounts, roles and security

Every surface — browser, REST, WebSocket — authenticates. There is no anonymous desk
unless the deployment explicitly enables a read-only or operator panel fallback, and that
fallback is logged loudly at every start.

- **Roles**: an account may read, operate (write), or administer. A signed-in account
  that cannot operate sees the desk with a **read-only** chip and disabled controls.
- **Admin**'s password is set at bootstrap or generated and printed once; change it with
  the API's password endpoint. Accounts are managed over the REST API
  (`/api/auth/users`); there is no user-management panel on the surface.
- Five failed sign-ins lock an account. Sessions expire after 12 hours.
- Every administrative and operational mutation lands in the audit log (`/api/audit`).

---

## 17. Themes

![The desk in the light theme](img/23-desk-light.png)

The **Dark** / **Light** toggle in the header switches the desk's own theme. The desk
owns this deliberately — it does not follow the operating system — so a rack display
pinned to dark stays dark whatever the host machine decides.

---

*Manual generated against CloudCast Mixer as of 2026-08-25 (main @ ea6da2c, CCM-1…291).
Screenshots were captured with `build/manual-screenshots.mjs` against a real
`ccmix-server` on the AES67 transport, fed by the repository's `ccmix-testsrc`
identification tone on loopback multicast — every meter shows genuine signal, and the
levels shown are that test signal, not programme. The one exception is §15's rack panel,
photographed in the product's own DEMO mode, which the panel itself labels. The Clock
panel shows the no-PTP state of the capture host; the Intercom panel is shown without a
RemoteTalk connection.*
