# Agent guidance

Instructions for AI coding assistants working in this repository. `CLAUDE.md` adds Claude-specific rules on top of this file.

## Project

Shawn Garello's personal portfolio, developed in a public repository. Shawn is the author of the portfolio: its concept, design, code and content. Assistants help with the work but are not authors (see [Authorship and commits](#authorship-and-commits)).

The audience is software engineering recruiters, other developers, and people who care about design. A visitor should be able to tell who Shawn is and reach Shawn's work quickly. The basketball story is the memorable part, but it must never get in the way of reading.

## The concept

A minimal, single-column site with one basketball story running from top to bottom:

1. **Ticket.** A horizontal digital game ticket loads. Its details (number, event, name, role, date, tip-off time, seat) decode from scrambled characters, the shot in its play diagram draws in, a scan line passes over the barcode, `✓ ADMITTED` decodes on the stub, and the stub tears off. It is brief and skippable (Skip button and Esc), and reduced-motion visitors skip it.
2. **Court.** A small animated half court at the top of the page, four on four with the hoop on the left. Shawn is the only player with the ball; the other seven run. Later, clicking a player may trigger a screen, a cut or a dribble move.
3. **The shot.** Scrolling winds Shawn up and releases the shot. The ball flies out of the court.
4. **The rail.** The ball lands on a vertical node-and-line rail beside the sections and becomes an orange dot that travels down it, lighting each section.
5. **The finish.** A side-view hoop hangs directly under the rail's last node. When the dot reaches that node, it grows back into a ball and falls straight through the hoop with real gravity, bounces, and rolls to a rest on the footer floor.

Design direction: quiet, small and handmade, with one memorable moment rather than effects everywhere. The layout uses a narrow centered column (about 380px of text, 15px type) with the court above it at about 522x348. Final art is planned as pixel art.

**References.** shwn.design and elaineyu.design inspire the feeling: calm pages, handmade details, small surprises. They are not templates. Do not reproduce their layouts, illustrations, components or copy. The owner of shwn.design is also named Shawn, so close resemblance would read as copying.

## Current stage

**Greybox.** `prototype/greybox.html` blocks out the whole sequence with plain shapes so composition, sizes and scroll timing can be settled before final art, content or a framework.

Not decided yet: the framework (Astro is the leading candidate), the pixel-art pipeline, the license, the phone layout for the horizontal ticket, and the final content. Don't start on any of these without being asked.

Content (a short bio and two to four projects) should be written alongside the motion work. Polishing the animation must not block real content.

## Repository layout

| Path | Purpose |
| --- | --- |
| `prototype/greybox.html` | Single-file, dependency-free greybox. Open it directly in a browser. |
| `README.md` | Public overview of the concept and status. |
| `AGENTS.md` | This file. |
| `CLAUDE.md` | Claude-specific rules. |
| `private/` | Ignored. Local-only material such as a resume or drafts. |

## How the prototype works

- **Ticket.** The ticket's wording lives in the `TICKET` object. The date and time are filled in live. `runTicket()` drives the load, decode, scan and tear timeline, and `finishTicket()` ends it. `fitTicket()` scales the ticket to fit the viewport. Each decoding field (`data-scr`) is split into fixed-width character boxes (`lockWidths()`) so scrambling never shifts the layout. `DECODE` sets when each field resolves, and serif fields scramble through similar-width letters.
- **Court.** A 2D canvas with a simple 3/4 projection, `P(x, z, h)`, in feet. `x` runs from 0 (baseline, hoop side) to 47 (half-court line), `z` from 0 (far sideline) to 50 (near sideline), and `h` is height. The scene is drawn to a low-resolution offscreen canvas and scaled up to get the pixel size.
- **Ball.** From the release to the last node, `ballAt(scroll)` is a pure function of scroll position. After that, the ball runs on time-based physics. The phases are:
  - `hand`: drawn by the court.
  - `flight`: a screen-space curve (`flightPoint`) from the release point that swings wide through the left margin and comes in to the rail from outside. The ball must never pass over text. `measure()` samples the flight and the readout warns if it does, and phones use a wider swing that briefly takes the ball off-screen.
  - `rail`: the dot sits at the anchor line, clamped to the rail.
  - `free`: once scrolling reaches `L.railEnd`, `updateFreeBall()` hands the ball to physics in page space (`stepBall()`): gravity, the rim ends and backboard as colliders, a damping net, a floor bounce, rolling friction and screen-edge walls. Scrolling back above the last node returns it to the rail. Arriving at the bottom without scrolling through, such as after a reload, places the ball already at rest.

  Keep the scroll-driven part exactly reversible.
- **Footer easter eggs.** While the ball is on the floor, an invisible `#ballHit` button follows it.
  - Tapping or clicking it (or pressing Enter or Space when it's focused) bounces the ball.
  - Pressing, dragging back and letting go shoots it slingshot-style (`SHOT_POWER`, `SHOT_MAX`), with a pull line and the predicted arc drawn while aiming.
  - A shot that drops through the rim before touching the floor scores (`onScore()`): the net swishes, `+2` and confetti appear, and a counter shows on the floor.
  - Three makes reveal `#egg`, a placeholder line for a surprise that hasn't been decided yet.

  Keep these hidden extras: they should never block content or be required to use the site.
- **Layout.** `measure()` caches page positions. `railDotDocY()` eases the dot onto the last node when the page is too short for that node to reach the dot line. `fitYard()` sizes the space under the last section so the footer floor sits a fixed distance below the rim.
- **Tune panel.** Every adjustable value lives in `DEFS` with a label, range and default. Settings persist in `localStorage` under `court-greybox:v2`. Bump the key, and migrate what still applies, whenever the meaning of a value changes.

## Checking changes

Verify in a real browser before reporting work as done:

- Check desktop (1440x900) and phone (390x844).
- Confirm there are no console errors.
- Play the ticket through, and also skip it.
- Scroll forward through every phase (dribble, wind-up, flight, rail, drop through the hoop, ball at rest), then back up to the top.
- On the floor, tap the ball, drag back and shoot it, and use Enter on the focused ball.
- Check that reduced motion skips the ticket.

Say plainly what was and wasn't checked.

## Accessibility

- The ticket must be skippable and short. Reduced motion skips it.
- Canvases are decorative (`aria-hidden`). All real content must be readable as ordinary HTML without the animation.
- Later work should add keyboard access for any interactive players and a plain view for visitors who only want the content.

## Privacy

Nothing from `private/` is committed, published, uploaded or included in a build. Do not publish personal details (resume contents, contact details, employment history) unless Shawn has approved the exact wording.

## Authorship and commits

- Shawn Garello is the sole author of this portfolio. Commits use Shawn's configured Git identity.
- Assistants must not appear as commit author, committer or co-author. Do not add `Co-authored-by` trailers, AI attribution or "generated with" footers to commits, pull requests, code comments, documentation or site content.
- Never override the Git identity or set author or committer environment variables.
- Commit in small, meaningful steps. Each message should describe what changed and why, and match what the commit actually contains.
- Don't push, rewrite published history, create releases or change repository settings unless asked.
